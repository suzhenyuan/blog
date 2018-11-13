---
layout: post_layout
comments: true
title: feign源代码解析
sub_title: 
meta-keyword: spring boot,eureka server, service discovery, feign, feign source code
meta-description: Introduce to feign source code.[2.0.1.RELEASE]
categories: spring-cloud
tags: spring-cloud,feign
description: feign 源代码解析，分析了restTemplate、request与response工作原理与流程，最后介绍了cloud项目的异常统一处理方式。
---


## pom配置

本文介绍的feign版本为`2.0.1.RELEASE`， pom设置如下所示：

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
        <version>2.0.1.RELEASE</version>
    </dependency>

同时会依赖引入feign-core-9.5.1.jar，相关的源代码就在这里。

## 源代码分析

* InvocationHandlerFactory
  控制反转调用入口在`feign.InvocationHandlerFactory.class`中

    
    static final class Default implements InvocationHandlerFactory {
    	@Override
    	public InvocationHandler create(Target target, Map<Method, MethodHandler> dispatch) {
    		return new ReflectiveFeign.FeignInvocationHandler(target, dispatch);
    	}
    }
    

* ReflectiveFeign
    
feign调用invoke入口(cglib)，这里equals()、hashCode()和toString()直接调本地方法就行了，不需要往下执行。
    
	
    @Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		if ("equals".equals(method.getName())) {
			try {
				Object otherHandler = args.length > 0 && args[0] != null ? Proxy.getInvocationHandler(args[0]) : null;
				return equals(otherHandler);
			} catch (IllegalArgumentException e) {
				return false;
			}
		} else if ("hashCode".equals(method.getName())) {
			return hashCode();
		} else if ("toString".equals(method.getName())) {
			return toString();
		}
		return dispatch.get(method).invoke(args);
	}

* SynchronousMethodHandler
  
   上文`dispatch.get(method).invoke(args);`调用函数如下：


   
    @Override
	public Object invoke(Object[] argv) throws Throwable {
		RequestTemplate template = buildTemplateFromArgs.create(argv);
		Retryer retryer = this.retryer.clone();
		while (true) {
			try {
				return executeAndDecode(template);
			} catch (RetryableException e) {
				retryer.continueOrPropagate(e);
				if (logLevel != Logger.Level.NONE) {
					logger.logRetry(metadata.configKey(), logLevel);
				}
				continue;
			}
		}
	}
	
	

   这里做了几个事情：
    - RequestTemplate: 组装请求参数
    - executeAndDecode: 发送请求与回应数据解码
    
* RequestTemplate 参数组装过程



最终创建的request，包括如下的内容：method，url，body，charset。


    public Request request() {
        Map<String, Collection<String>> safeCopy = new LinkedHashMap<String, Collection<String>>();
        safeCopy.putAll(headers);
        return Request.create(
            method, url + queryLine(),
            Collections.unmodifiableMap(safeCopy),
            body, charset
        );
      }

* executeAndDecode 发送请求与响应处理


        Object executeAndDecode(RequestTemplate template) throws Throwable {
            //创建request
    		Request request = targetRequest(template);
    
    		if (logLevel != Logger.Level.NONE) {
    			logger.logRequest(metadata.configKey(), logLevel, request);
    		}
    
    		Response response;
    		long start = System.nanoTime();
    		try {
                //通过client发送请求
    			response = client.execute(request, options);
    			// ensure the request is set. TODO: remove in Feign 10
    			response.toBuilder().request(request).build();
    		} catch (IOException e) {
    			if (logLevel != Logger.Level.NONE) {
    				logger.logIOException(metadata.configKey(), logLevel, e, elapsedTime(start));
    			}
    			throw errorExecuting(request, e);
    		}
    		long elapsedTime = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start);
    
    		boolean shouldClose = true;
    		try {
    			if (logLevel != Logger.Level.NONE) {
    				response = logger.logAndRebufferResponse(metadata.configKey(), logLevel, response, elapsedTime);
    				// ensure the request is set. TODO: remove in Feign 10
    				response.toBuilder().request(request).build();
    			}
    			if (Response.class == metadata.returnType()) {
    				if (response.body() == null) {
    					return response;
    				}
    				if (response.body().length() == null || response.body().length() > MAX_RESPONSE_BUFFER_SIZE) {
    					shouldClose = false;
    					return response;
    				}
    				// Ensure the response body is disconnected
    				byte[] bodyData = Util.toByteArray(response.body().asInputStream());
    				return response.toBuilder().body(bodyData).build();
    			}
                //http相应码是200～300区间以及404的，采用正确的decoder
    			if (response.status() >= 200 && response.status() < 300) {
    				if (void.class == metadata.returnType()) {
    					return null;
    				} else {
    					return decode(response);
    				}
    			} else if (decode404 && response.status() == 404 && void.class != metadata.returnType()) {
    				return decode(response);
    			} else {
                    //使用errorDecoder
    				throw errorDecoder.decode(metadata.configKey(), response);
    			}
    		} catch (IOException e) {
    			if (logLevel != Logger.Level.NONE) {
    				logger.logIOException(metadata.configKey(), logLevel, e, elapsedTime);
    			}
    			throw errorReading(request, response, e);
    		} finally {
    			if (shouldClose) {
    				ensureClosed(response.body());
    			}
    		}
    	}    
    

   - 这里通过httpclient发送请求
   - 请求回来的处理，根据response.status()来采取不同的策略，这里需要关注的是errorDecoder。

## 关于errorDecoder 与异常统一处理

   在服务提供者与服务消费者之间，如果需要把提供者发生的异常传递给消费者，提供者可以通过`@ControllerAdvice`捕捉异常，返回500. 如下所示：

   
    @ControllerAdvice
    public class ProviderControllerAdviceHandler {
    
    	private static final Logger log = LoggerFactory.getLogger(ProviderControllerAdviceHandler.class.getSimpleName());
    
    	@ExceptionHandler(Throwable.class)
    	@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    	@ResponseBody
    	public BizException processProviderException(HttpServletRequest request, HttpServletResponse response, Throwable e) throws ServletException, IOException {
    		try{
    			log.error("服务提供者发生异常---------------------------");
    			log.error("getRequestURI: "+request.getRequestURI());
    			log.error("getRemoteAddr: "+request.getRemoteAddr());
    			log.error("getQueryString: "+request.getQueryString());
    			log.error("异常信息："+e.getClass()+","+e.getMessage());
    			log.error(e.getMessage(), e);
    			
    			//如果是代码手动抛出的异常,则类型为bizException
    			if(e instanceof BizException){
    				log.error("异常信息："+((BizException)e).getErrorCode()+","+((BizException)e).getErrorMsg());
    				return (BizException)e;
    			}
    		}catch(Exception ee){
    			ee.printStackTrace();
    		}
    		//其他异常转化为bizException
    		return new BusinessException(new GlobalResponseCode(GlobalResponseCode.SYS_PASS_EXCEPTION.getCode(),e.getMessage()));
    	}
    }

   
`@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)` 是重点

   在消费者端自定义feign的errorDecoder，如下所示：

    
	
    public class FeignErrorDecoder implements ErrorDecoder{
    
    	public Exception decode(String methodKey, Response response) {
    		try {
    			System.out.println(response.headers().toString());
    			String body = IOUtils.toString(response.body().asInputStream());
    			System.out.println(body);
    			JSONObject m = JSONObject.parseObject(body);
    			System.out.println(m.get("code"));
    			System.out.println(m.get("msg"));
    			return new BizException(m.getInteger("code"),m.getString("msg"));
    		} catch (IOException e) {
    			e.printStackTrace();
    		}
    		return new Default().decode(methodKey, response);
    	}
    }

   消费者统一异常处理流程如下：

    
	
    @ControllerAdvice
    public class GlobalExceptionHandler {
    	private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    	@ExceptionHandler(Throwable.class)
    	@ResponseStatus(HttpStatus.OK)
    	@ResponseBody
    	public ResBody processConsumerException(HttpServletRequest request, HttpServletResponse response,Throwable e) throws ServletException, IOException {
    		log.error(e.getMessage(), e);
    		log.error("服务消费者捕捉到了异常---------------------------");
    		log.error("getRequestURI: "+request.getRequestURI());
    		log.error("getRemoteAddr: "+request.getRemoteAddr());
    		log.error("getQueryString: "+request.getQueryString());
    		log.error("异常信息："+e.getClass()+","+e.getMessage());
    		ResBody resBody = null;
    		//如果是代码手动抛出的异常,则类型为bizException
    		if (e instanceof BizException) {
    			log.error("异常信息："+((BizException)e).getErrorCode()+","+((BizException)e).getErrorMsg());
    			resBody= ResBody.buildFailResBody(new GlobalResponseCode(((BizException) e).getErrorCode(), ((BizException) e).getErrorMsg()));
    		}
    		else{
    			//其他异常返回系统程序异常错误
    			resBody = ResBody.buildFailResBody();
    		}
    		return resBody;
    
    	}
    }

  在异常处理中，还需要根据同步请求还是异步请求，采取不同的策略。

## feign配置

    #连接超时时间
    ribbon.ConnectTimeout=30000
    #读取超时时间
    ribbon.ReadTimeout=60000


## 写在最后的话

这里只是大概梳理了一下feign对网络请求的流程，太细节的内容，我也没有太深入，主要还是点到即止，先解决问题再说。

在实现异常统一处理流程的时候，只是看了相关的文档和博客，但是遇到的问题，根本就无从解决，最后还是回归代码，代码里面已经告诉了你一切内容。feign的流程代码也不是很复杂，相信大家都可以看得明白。


## 其他内容 

- spring-cloud-openfien-core-2.0.1.RELEASE.jar
FeignClientsRegistrar.registerFeignClients()


- spring-boot-autoconfigure-2.0.4.RELEASE.jar
IntegrationAutoConfigurationScanRegistrar.registerBeanDefinitions()

