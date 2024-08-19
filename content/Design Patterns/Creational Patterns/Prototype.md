- [[#1. Overview|1. Overview]]
- [[#2. How to achieve Deep Cloning|2. How to achieve Deep Cloning]]
- [[#3. Examples|3. Examples]]
		- [[#Cloneable|Cloneable]]
		- [[#Copy constructor|Copy constructor]]
		- [[#Serialization|Serialization]]

#### 1. Overview
- The idea behind a **Prototype** is to use a partially or fully initialized object that you clone - usually a deep clone without all the references and make use of it. So I guess the Prototype is just the idea of obtaining an object from another object and ways to achieve it.
- A **shallow copy** is when you try to clone an object and you just take the references of that object and pass them to the clone. The issue with this is that when modifying any of the two objects (original or clone) fields you will affect both.
- A **deep copy** implies that the two objects start with the same values in the fields but have different references so they do not interfere with one another.

#### 2. How to achieve Deep Cloning
- For deep cloning we can override the clone() method that the base class for everything (Object) implements and make sure that we return a new object with the same values. 
	- Make the class implements the Cloneable interface. The Cloneable interface is a moniker that doesn't expose any method it just tells you that you can clone it.
	- Make sure that the cloning of the attributes is done deep as well.
	- This is not a recommended way of doing deep clones as the Cloneable interface doesn't specify which type of clone it makes (shallow or deep) and the default behavior of clone is to do shallow copies.
- Another more ***tasteful*** way of making deep copies is to do a copy constructor. Allow an object to be instantiated via another instance of itself.
	- To achieve this we make a constructor that takes other object as argument.
	- For this to work properly we need to make sure that this copying mechanism goes down the encapsulation chain - all the attributes must have a deep copy way as well.
	- The disadvantage of this method is that we need to create copy constructors for everything which can get tiresome.
- We can use the serialization method to avoid the copy constructor restrictions. Basically this method uses serialization and deserialization to create a copy by value.
	- We can use for example `SerializationUtils.roundtrip()` which does both serialization and deserializations.
	- The disadvantage of this method is that you need to have serializable objects otherwise it's a bust.

#### 3. Examples

###### Cloneable
```java
// Cloneable is a marker interface
class Address implements Cloneable {
  public String streetName;
  public int houseNumber;

  public Address(String streetName, int houseNumber)
  {
    this.streetName = streetName;
    this.houseNumber = houseNumber;
  }

  @Override
  public String toString()
  {
    return "Address{" +
      "streetName='" + streetName + '\'' +
      ", houseNumber=" + houseNumber +
      '}';
  }

  // base class clone() is protected
  @Override
  public Object clone() throws CloneNotSupportedException
  {
    return new Address(streetName, houseNumber);
  }
}

class Person implements Cloneable
{
  public String [] names;
  public Address address;

  public Person(String[] names, Address address)
  {
    this.names = names;
    this.address = address;
  }

  @Override
  public String toString()
  {
    return "Person{" +
      "names=" + Arrays.toString(names) +
      ", address=" + address +
      '}';
  }

  @Override
  public Object clone() throws CloneNotSupportedException
  {
    return new Person(
      // clone() creates a shallow copy!
      /*names */ names.clone(),

      // fixes address but not names
      /*address */ // (Address) address.clone()
      address instanceof Cloneable ? (Address) address.clone() : address
    );
  }
}

class CloneableDemo
{
  public static void main(String[] args)
    throws CloneNotSupportedException
  {
    Person john = new Person(new String[]{"John", "Smith"},
      new Address("London Road", 123));

    // shallow copy, not good:
    //Person jane = john;

    // jane is the girl next door
    Person jane = (Person) john.clone();
    jane.names[0] = "Jane"; // clone is (originally) shallow copy
    jane.address.houseNumber = 124; // oops, also changed john

    System.out.println(john);
    System.out.println(jane);
  }
}

```

###### Copy constructor
```java
class Address
{
  public String streetAddress, city, country;

  public Address(String streetAddress, String city, String country)
  {
    this.streetAddress = streetAddress;
    this.city = city;
    this.country = country;
  }

  public Address(Address other)
  {
    this(other.streetAddress, other.city, other.country);
  }

  @Override
  public String toString()
  {
    return "Address{" +
      "streetAddress='" + streetAddress + '\'' +
      ", city='" + city + '\'' +
      ", country='" + country + '\'' +
      '}';
  }
}

class Employee
{
  public String name;
  public Address address;

  public Employee(String name, Address address)
  {
    this.name = name;
    this.address = address;
  }

  public Employee(Employee other)
  {
    name = other.name;
    address = new Address(other.address);
  }

  @Override
  public String toString()
  {
    return "Employee{" +
      "name='" + name + '\'' +
      ", address=" + address +
      '}';
  }
}

class CopyConstructorDemo
{
  public static void main(String[] args)
  {
    Employee john = new Employee("John",
      new Address("123 London Road", "London", "UK"));

    //Employee chris = john;
    Employee chris = new Employee(john);

    chris.name = "Chris";
    System.out.println(john);
    System.out.println(chris);
  }
}
```

###### Serialization
```java
import org.apache.commons.lang3.SerializationUtils;
import java.io.Serializable;

// some libraries use reflection (no need for Serializable)
class Foo implements Serializable
{
  public int stuff;
  public String whatever;

  public Foo(int stuff, String whatever)
  {
    this.stuff = stuff;
    this.whatever = whatever;
  }

  @Override
  public String toString()
  {
    return "Foo{" +
      "stuff=" + stuff +
      ", whatever='" + whatever + '\'' +
      '}';
  }
}

class CopyThroughSerializationDemo
{
  public static void main(String[] args)
  {
    Foo foo = new Foo(42, "life");
    // use apache commons!
    Foo foo2 = SerializationUtils.roundtrip(foo);

    foo2.whatever = "xyz";

    System.out.println(foo);
    System.out.println(foo2);
  }
}

```