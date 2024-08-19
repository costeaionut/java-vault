#### 1. Overview
- The singleton pattern that allows us to have components that instantiate only once and that instance is then provided everywhere it's needed.
- There are times where it makes sense to have only one instance such as a database repository, a factory object or the call of the constructor is expensive and we want to avoid using it much.

#### 2. Basic Singleton
- For a basic singleton we just need to follow a couple of steps:
	1. We need to have a **private** constructor to restrict the creation of the class.
	2. We need to have an object instance in our class.
	3. We need to have a method to expose our private instance.
- The issue with this implementation is that we can easily break the singleton contract knowingly and by accident:
	- For example we can use reflection to call the constructor and get an instance - this requires the developer to knowingly break the singleton restrictions.
	- However, we can unknowingly break the contract using serialization. When deserializing an object the JVM doesn't care that the constructor is private and it will return an instance.
- We can easily solve this two issues by making the constructor throw an Unsupported exception and by overriding the `public Object readResolve()` method that is checked by the deserialization to return our instance.

##### Example
```java
public class BasicSingleton{
	//1. Private constructor
	private BasicSingleton(){
		...
	}

	// 2. Object instance
	private static final BasicSingleton INSTANCE = new BasicSingleton();

	//... other methods and fields

	//3. A way to expose our instance.
	public static BasicSingleton getInstance() {
		return Instance;
	}
}
```

Fixing the issues with the BasicSingleton:
```java
public class BasicSingleton implements Serializable{

	private BasicSingleton(){
		throw new UnsupportedOperationException("Constructor is private");
	}

	private static final BasicSingleton INSTANCE = new BasicSingleton();
	private int value;

	public static BasicSingleton getInstance() {
		return Instance;
	}

	public int getValue() {
		return value;
	}

	public void setValue(int value) {
		this.value = value;
	}

	// To fix deserialization issue
	// The readResolve method is called when ObjectInputStream has read an        // object from the stream and is preparing to return it to the caller.
	public Object readResolve(){
		return INSTANCE;
	}
}

public static void serializeSingleton(BasicSingleton object) {
	...;
}

public static BasicSingleton deserializeSingleton() {
    ...;
}

public static void main(String[] args) {
	BasicSingleton singleton = BasicSingleton.getInstance();
	singleton.setValue(111);

	serializeSingleton(singleton);

	BasicSingleton singleton2 = deserializeSingleton();
	singleton2.setValue(222);

	System.out.println(singleton == singleton2);
	// false if missing readResolve, true otherwise

	System.out.println(singleton.getValue()) 
	//111 if we are missing the readResolve, 222 otherwise
	 
	System.out.println(singleton2.getValue())//
}
```

If the constructor throws an error and initializing directly inside a static variable is not possible we can use a static block
```java
public class BasicSingleton{
	//1. Private constructor
	private BasicSingleton() throws IOException {
		...
	}

	// 2. Object instance
	private static final BasicSingleton INSTANCE;

	static {
		try {
			INSTANCE = new BasicSingleton();
		} catch (IOException e) {
			System.out.println("Failed to initialize singleton");
		}
	}

	//... other methods and fields

	//3. A way to expose our instance.
	public static BasicSingleton getInstance() {
		return Instance;
	}
}
```
