
#### Multiton
- The multiton is a variation of the [[Basic Singleton|singleton pattern]] that has more than one instance but a finite number of instances based on certain characteristics.
- Technically we still want to return the one of a couple of instances but based on some information.
##### Example
- This is a basic multiton implementation but this is **not thread safe**!
```java
enum Subsystem
{
  PRIMARY,
  AUXILIARY,
  FALLBACK
}

class PrinterMultiton
{
  private PrinterMultiton()
  {
  }

  private static HashMap<Subsystem, PrinterMultiton>
    instances = new HashMap<>();

  public static PrinterMultiton get(Subsystem ss)
  {
    if (instances.containsKey(ss)){
	    return instances.get(ss);
    }
    
    PrinterMultiton instance = new PrinterMultiton();
    instances.put(ss, instance);
    return instance;
  }
}

```

- A thread safe implementation of the multiton:
```java
enum Subsystem
{
  PRIMARY,
  AUXILIARY,
  FALLBACK
}

public class Multiton {
	//We make use of the ConcurrentHashMap to ensure synchronization
    private static final ConcurrentMap<Subsystem, Multiton> multitons = new ConcurrentHashMap<>();

    private final Subsystem key;
    private Multiton(Subsystem key) { this.key = key; }

    public static Multiton getInstance(final Subsystem key) {
        return multitons.computeIfAbsent(key, Multiton::new);
    }
}

```