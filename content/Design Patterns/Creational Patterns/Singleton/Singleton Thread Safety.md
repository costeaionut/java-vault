
#### Lazy Singleton

- There might be cases where we don't want the singleton to initialize at program start but rather initialize it only when we need the instance aka at `getInstance` call.
- To achieve this we have to initialize the object on the get instance call. Also we need to check if the object was already initialized to avoid any duplication.
- This plain method will have some drawbacks, mostly it will not be thread safe as multiple threads could call the `getInstance` for the first time simultaneously resulting in different instances.
- To make it thread safe we can use the double-check locking method and making the singleton INSTANCE volatile (as we want the value to be accessed from the main memory not from core cache).

##### Examples

```java
public class LazySingleton {
	private LazySingleton() {
	}
	
	public static volatile LazySingleton instance;
	
	// This method has synchronization issues
	public LazySingleton getInstance() {
		if (instance == null) {
			instance = new LazySingleton();
		}
		
		return instance;
	}
	
	//Double-checked locking
	// Although this way is thread safe when using with volatile instance 
	// it's a bit complicated
	public LazySingleton getInstanceThreadSafe() {
		if(instance == null) {
			synchronized(LazySingleton.class) {
				if(instance == null) {
					instance = new LazySingleton();
				}
			}
		}
		
		return instance;
	}
}
```


#### Inner Static Singleton
- One way to achieve a thread safe singleton is to make use of a static inner class that will hold our singleton instance.
- In this case Java ensures the laziness of our singleton as the inner static class will be loaded when an object of that class is used or a method called.

##### Example:
```java
class InnerStaticSingleton
{
	private InnerStaticSingleton() {}
	private static class Impl {
		private static final InnerStaticSingleton INSTANCE = new InnerStaticSingleton();
	}
	
	public InnerStaticSingleton getInstace() {
		return Impl.INSTANCE;
	}
}
```

#### Enum based Singleton
- We can create a singleton using an Enum and it won't have the issues previously mentioned about thread safety, reflection constructor call but it will have its unique issues.
- As this singleton uses the properties of an Enum we won't be able to inherit from it or extend it in any way. Also when talking about serialization and persistence, for an Enum all we save is the name - the values of the fields are lost.
- Effectively the fields of the Enum singleton are transient and can't be saved. If we save and the load an instance and we get the correct custom fields values that happens because on load we got the same instance but there might be cases where the app gets killed and we want to read the value at a later date.

##### Example
```java
enum EnumBasedSingleton
{
  INSTANCE; // hooray

  // enum type already has a private ctor by default,
  // but if you need to initialize things...
  EnumBasedSingleton() {
    value = 42;
  }

  // reflection not a problem, guaranteed 1 instance in JVM

  // field values not serialized
  int value;

  public int getValue()
  {
    return value;
  }
  public void setValue(int value)
  {
    this.value = value;
  }
}
```
