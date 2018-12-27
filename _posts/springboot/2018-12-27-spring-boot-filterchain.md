---
layout: post_layout
comments: true
title: Servlet filter的责任链模式
sub_title: 
meta-keyword: Servlet,filter,责任链模式,设计模式
meta-description: 通过对FilterChain源代码的分析，了解其责任链实现模式
categories: spring-boot
tags: Servlet, filter
description:  通过对FilterChain源代码的分析，了解其责任链实现模式
date: 2018-12-27
---

## 什么是责任链模式
责任链模式在面向对象程式设计里是一种软件设计模式，它包含了一些命令对象和一系列的处理对象。每一个处理对象决定它能处理哪些命令对象，它也知道如何将它不能处理的命令对象传递给该链中的下一个处理对象。该模式还描述了往该处理链的末尾添加新的处理对象的方法。

## Servlet中的Filter


    public final class ApplicationFilterChain implements FilterChain {

        private ApplicationFilterConfig[] filters = new ApplicationFilterConfig[0];
        private int pos = 0;
        private int n = 0;
        private Servlet servlet = null;
        private void internalDoFilter(ServletRequest request,
                                      ServletResponse response)
            throws IOException, ServletException {
            if (pos < n) {
                ApplicationFilterConfig filterConfig = filters[pos++];
                Filter filter = filterConfig.getFilter();
                filter.doFilter(request, response, this);
                return;
            }
            servlet.service(request, response);
        }
    }

   Filer的某个具体实现,比如CsrfFilter:

    public final class CsrfFilter extends OncePerRequestFilter {
	
    	@Override
    	protected void doFilterInternal(HttpServletRequest request,
    			HttpServletResponse response, FilterChain filterChain)
    					throws ServletException, IOException {
    		if (!csrfToken.getToken().equals(actualToken)) {
                ... ...
    			return;
    		}
    		filterChain.doFilter(request, response);
    	}
    }

从以上两段代码,可以看得出来,当某个Filter执行之后,调用filterChain.doFilter(request, response),会把该请求传递给下一个filter,如此循环,直到所有的filter都执行完了(即pos==n),最后再调用servlet.service(request, response);
如果某个filter返回了,没有继续调用filterChain.doFilter(request, response),则servlet.service(request, response)将不会得到机会执行,servlet容器将会返回请求给浏览器.请求结束.

示意图如下所示:
![Servlet filter filterchain][servlet_filterchain]


[servlet_filterchain]: /images/java/servlet_filterchain.png "Servlet filter filterchain"