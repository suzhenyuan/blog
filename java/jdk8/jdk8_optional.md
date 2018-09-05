
# jdk8 Optional 使用总结

## NPE问题

在平常的编码时，经常会遇到NPE的情况，为了避免NPE，不能不采取各种判空操作，尤其在逻辑复杂的时候，判断代码更是一匹布那么长了，比如：

	if(null != u.getUser() && null != u.getUser().getName() && (...)){
		....
	}

现在JDK8中的Optional可以让我们有一个更优雅的方式来处理这个问题。见下面的例子：

	public class User{
		private String Id;
		private String name;
		
		public User next;
		public String getId() {
			return Id;
		}
		public void setId(String id) {
			Id = id;
		}
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
		public User getNext() {
			return next;
		}
		public void setNext(User next) {
			this.next = next;
		}
	}
	
Optional的map方法，在多级取值时，也不需要担心NPE了。
	
	public static void main(String[] args) {
		User user=new User();
		user.setId("122");
		user.setName("frank");
		
		User next = new User();
		next.setId("133");
		next.setName("mike");
		user.setNext(next);
		
		Optional<User> userOpt = Optional.of(user);
		System.out.println(userOpt.map(u->u.getNext()).map(u->u.next).map(u->u.name).orElse("It is a null value"));
		
	}
	
## Optional源代码分析

* Optional初始化


	/**
     * Common instance for {@code empty()}.
     */
    private static final Optional<?> EMPTY = new Optional<>();

	/**
     * If non-null, the value; if null, indicates no value is present
     */
    private final T value;
	
	public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
	
	private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
	
注：Objects.requireNonNull(value) 在value为null时，将会抛出NullPointerException	

* 在取值(get())时必须先判断isPresent(),否则会抛出NoSuchElementException异常


	public boolean isPresent() {
        return value != null;
    }
	
	public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

* 级联操作map(),调用流程如下所示：


	public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
	
	public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
	
	public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
	
* 在value为null的时候，取传入的参数，orElse(other)


	public T orElse(T other) {
        return value != null ? value : other;
    }	
	
	public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

##  总结
* 在使用的时候，禁止直接调用get()
* map()、orElse()、orElseGet()组合使用时才是优雅的方式
