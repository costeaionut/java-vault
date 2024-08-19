
The SOLID design patterns are a group of design principles introduced by Robert Martin that are usually referenced in the specialty literature. They are sort of the *building blocks* one could say.

#### 2.1. Single Responsibility Principle (SRP)
- The SRP dictates that a class should be responsible of a single type of activity or, more academically said, we need a separation of **concerns** (concern being a responsibility in this case).
- The idea behind this is to avoid really cluttered and complex objects that fulfil more responsibilities than necessary. 
- Breaking this pattern leads to a GOD object that does way to many things, is hard to maintain and understand.
##### 2.1.1. Example of SRP 
- We have this simple Journal class that should only be in charge of managing journal entries. A break of the SRP would be to handle the persistence of the object in the journal method.
- To better structure our code, we can create a separate class that handles the persistence. This way the Persistence class is in charge of storing, loading and maintaining the objects long term, can be enhanced to handle future objects and also refactoring of the persistence methods is done in one place.
```java
class Journal
{
  private final List<String> entries = new ArrayList<>();

  private static int count = 0;

  public void addEntry(String text)
  {
    entries.add("" + (++count) + ": " + text);
  }

  public void removeEntry(int index)
  {
    entries.remove(index);
  }

  @Override
  public String toString() {
    return String.join(System.lineSeparator(), entries);
  }

  // here we break SRP
  public void save(String filename) throws Exception
  {
    try (PrintStream out = new PrintStream(filename))
    {
      out.println(toString());
    }
  }

  public void load(String filename) {}
  public void load(URL url) {}
}

// handles the responsibility of persisting objects
class Persistence
{
  public void saveToFile(Journal journal, 
    String filename, boolean overwrite) throws Exception
  {
    if (overwrite || new File(filename).exists())
      try (PrintStream out = new PrintStream(filename)) {
        out.println(journal.toString());
      }
  }

  public void load(Journal journal, String filename) {}
  public void load(Journal journal, URL url) {}
}

class SRPDemo
{
  public static void main(String[] args) throws Exception
  {
    Journal j = new Journal();
    j.addEntry("I cried today");
    j.addEntry("I ate a bug");
    System.out.println(j);

    Persistence p = new Persistence();
    String filename = "c:\\temp\\journal.txt";
    p.saveToFile(j, filename, true);
  }
}
```

#### 2.2. Open Closed Principle (OCP)

- The idea behind the Open Closed Principle (OCP) is that a class or a method is open for extension (as in enhance your already existing functionalities) but closed to modifications. 
- We should allow extensions of a certain functionality without going and modifying already existing and tested code.
- At no point in time you should go back to implemented code and change it. 
- You can play with interfaces and inheritance to extend the capabilities of a functionality but that is a separate piece of logic with it's own set of tests. The functionality already create is not touched and modified.

##### 2.2.1. Example of OCP:
- In this example we will look at a way to create a service that needs to filter our products based on different criteria.
- To be sure that we don't violate the OCP we will use the Specification Design Pattern.
- The specification patterns allows to check a certain property of an object is satisfied. A single Specification generic interface allows to create unitary properties of an object that can be furthermore combined AndSpecification, OrSpecification so you can easily combine multiple smaller pieces.

```java
// In this example we can see the Specification Pattern being used to satisfy the OCP.


enum Color
{
  RED, GREEN, BLUE
}

enum Size
{
  SMALL, MEDIUM, LARGE, YUGE
}

class Product
{
  public String name;
  public Color color;
  public Size size;

  public Product(String name, Color color, Size size) {
    this.name = name;
    this.color = color;
    this.size = size;
  }
}

// we introduce the Specification interface that exposes a method that checks a condition is satisfied
interface Specification<T>
{
  boolean isSatisfied(T item);
}

// We implement ColorSpecification and SizeSpecification, each being a unitary way to check one property
class ColorSpecification implements Specification<Product>
{
  private Color color;

  public ColorSpecification(Color color) {
    this.color = color;
  }

  @Override
  public boolean isSatisfied(Product p) {
    return p.color == color;
  }
}

class SizeSpecification implements Specification<Product>
{
  private Size size;

  public SizeSpecification(Size size) {
    this.size = size;
  }

  @Override
  public boolean isSatisfied(Product p) {
    return p.size == size;
  }
}

// We can easily create combinations of specifications in any way it benefits us
class AndSpecification<T> implements Specification<T>
{
  private Specification<T> first, second;

  public AndSpecification(Specification<T> first, Specification<T> second) {
    this.first = first;
    this.second = second;
  }

  @Override
  public boolean isSatisfied(T item) {
    return first.isSatisfied(item) && second.isSatisfied(item);
  }

}

// Finally we can use filters to make use of the Specifications
interface Filter<T>
{
  Stream<T> filter(List<T> items, Specification<T> spec);
}

class BetterFilter implements Filter<Product>
{
  @Override
  public Stream<Product> filter(List<Product> items, Specification<Product> spec) {
    return items.stream().filter(p -> spec.isSatisfied(p));
  }
}

class OCPDemo
{
  public static void main(String[] args) {
    // vv AFTER
    BetterFilter bf = new BetterFilter();
    System.out.println("Green products (new):");
    bf.filter(products, new ColorSpecification(Color.GREEN))
      .forEach(p -> System.out.println(" - " + p.name + " is green"));

    System.out.println("Large products:");
    bf.filter(products, new SizeSpecification(Size.LARGE))
      .forEach(p -> System.out.println(" - " + p.name + " is large"));

    System.out.println("Large blue items:");
    bf.filter(products,
      new AndSpecification<>(
        new ColorSpecification(Color.BLUE),
        new SizeSpecification(Size.LARGE)
      ))
      .forEach(p -> System.out.println(" - " + p.name + " is large and blue"));
  }
}
```

  

#### 2.3. Liskov Substitution Principle (LSP)

- The Liskov Substitution Principle (LSP) ensures that all child classes can be swapped for their parent class and the logic should be unaffected or that your interface implementations can be swapped with one another without breaking your application.
- The idea is to make sure that when working with such a chain of inheritance all the results of methods inside the classes and outside are predictable and that a class substitution doesn't break the logic.
##### 2.3.1. Example of LSP

```java
class Rectangle
{
  protected int width, height;

  public Rectangle() {
  }

  public Rectangle(int width, int height) {
    this.width = width;
    this.height = height;
  }

  public int getWidth() {
    return width;
  }

  public void setWidth(int width) {
    this.width = width;
  }

  public int getHeight() {
    return height;
  }

  public void setHeight(int height) {
    this.height = height;
  }

  public int getArea() { return width*height; }

  @Override
  public String toString() {
    return "Rectangle{" +
      "width=" + width +
      ", height=" + height +
      '}';
  }

  public boolean isSquare()
  {
    return width == height;
  }
}

class Square extends Rectangle
{
  public Square() {
  }

  public Square(int size) {
    width = height = size;
  }

  @Override
  public void setWidth(int width) {
    super.setWidth(width);
    super.setHeight(width);
  }

  @Override
  public void setHeight(int height) {
    super.setHeight(height);
    super.setWidth(height);
  }
}

// To make sure we don't break LSP we can use a Factory and don't break-up the classes too much
class RectangleFactory
{
  public static Rectangle newSquare(int side)
  {
    return new Rectangle(side, side);
  }

  public static Rectangle newRectangle(int width, int height)
  {
    return new Rectangle(width, height);
  }
}

class LSPDemo
{
  // The method is just a didactic example as it's quite stupid but:
  // - When we have a rectangle the method works as expected
  // - When we have a square we will first save the initial width of the square then because we update height we also update width and so the logic changes.
  static void useIt(Rectangle r)
  {
    int width = r.getWidth();
    r.setHeight(10);
    System.out.println("Expected area of " + (width*10) + ", got " + r.getArea());
  }

  public static void main(String[] args) {
    Rectangle rc = new Rectangle(2, 3);
    useIt(rc);

    Rectangle sq = new Square();
    sq.setHeight(5);
    useIt(sq);
  }
}
```
#### 2.4. Interface Segregation Principle (ISP)

- The Interface Segregation Principle (ISP) says that an interface should contain only the minimum code so that we never need to only implement parts of the interface in a class.   
- Basically this is the SRP of the interface declaration. 
- Don't slap everything in one interface but do break it up in small enough chunks that it makes sense.

##### 2.4.1. Example of ISP:
- In this example we can see how declaring a `Machine` interface that has unnecessary methods can blow up in our face when we need only parts of it.
- For example we can have a multi-functional printer that handles printing, copying and scanning and this interface fits but if we only have a Printer or a Fax Machine then we suddenly have useless methods that need to somehow be handled.
- A better approach is to break the interface into bite-sized interfaces that do no more than needed. Instead of one big Machine interface we can have IPrinter, IScanner, IFaxer and combine them as needed in our code.
```java 
package com.activemesa.solid.isp;

class Document
{
}

interface Machine
{
  void print(Document d);
  void fax(Document d) throws Exception;
  void scan(Document d) throws Exception;
}

// ok if you need a multifunction machine
class MultiFunctionPrinter implements Machine
{
  public void print(Document d)
  {
    //
  }

  public void fax(Document d)
  {
    //
  }

  public void scan(Document d)
  {
    //
  }
}

class OldFashionedPrinter implements Machine
{
  public void print(Document d)
  {
    // yep
  }

  public void fax(Document d) throws Exception
  {
    throw new Exception();
  }

  public void scan(Document d) throws Exception
  {
    throw new Exception();
  }
}

interface Printer
{
  void Print(Document d) throws Exception;
}

interface IScanner
{
  void Scan(Document d) throws Exception;
}

class JustAPrinter implements Printer
{
  public void Print(Document d)
  {

  }
}

class Photocopier implements Printer, IScanner
{
  public void Print(Document d) throws Exception
  {
    throw new Exception();
  }

  public void Scan(Document d) throws Exception
  {
    throw new Exception();
  }
}

interface MultiFunctionDevice extends Printer, IScanner //
{

}

class MultiFunctionMachine implements MultiFunctionDevice
{
  // compose this out of several modules
  private Printer printer;
  private IScanner scanner;

  public MultiFunctionMachine(Printer printer, IScanner scanner)
  {
    this.printer = printer;
    this.scanner = scanner;
  }

  public void Print(Document d) throws Exception
  {
    printer.Print(d);
  }

  public void Scan(Document d) throws Exception
  {
    scanner.Scan(d);
  }
}
```

#### 2.5. Dependency Inversion Principle (DIP)
- The DIP says that we must fulfill these two conditions:
	1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
	2. Abstractions should not depend on details. Details should depend on abstractions.
- The principle is somehow simple and complicated at the same time. 😒
- Mostly the idea is that when you have a class (the high level module) that should use another class (the low level module), make sure that the low level module doesn't expose irrelevant methods and attributes but it only exposes (via an interface or an abstraction) the logic that the high level module would need. 
- Make sure also that the high-level module interacts with the low-level module via its abstraction not via the abstractions implementation directly.
##### Example of DIP: 

```java 
package com.activemesa.solid.dip;

import org.javatuples.Triplet;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;

// A. High-level modules should not depend on low-level modules. 
// Both should depend on abstractions.

// B. Abstractions should not depend on details. 
// Details should depend on abstractions.

enum Relationship
{
  PARENT,
  CHILD,
  SIBLING
}

class Person
{
  public String name;
  // dob etc.


  public Person(String name) {
    this.name = name;
  }
}

interface RelationshipBrowser
{
  List<Person> findAllChildrenOf(String name);
}

class Relationships implements RelationshipBrowser
{
  public List<Person> findAllChildrenOf(String name) {

    return relations.stream()
      .filter(x -> Objects.equals(x.getValue0().name, name)
              && x.getValue1() == Relationship.PARENT)
      .map(Triplet::getValue2)
      .collect(Collectors.toList());
  }

  // Triplet class requires javatuples
  private List<Triplet<Person, Relationship, Person>> relations =
    new ArrayList<>();

  public List<Triplet<Person, Relationship, Person>> getRelations() {
    return relations;
  }

  public void addParentAndChild(Person parent, Person child)
  {
    relations.add(new Triplet<>(parent, Relationship.PARENT, child));
    relations.add(new Triplet<>(child, Relationship.CHILD, parent));
  }
}

class Research
{
  public Research(Relationships relationships)
  {
    // high-level: find all of john's children
    List<Triplet<Person, Relationship, Person>> relations = relationships.getRelations();
    relations.stream()
      .filter(x -> x.getValue0().name.equals("John")
              && x.getValue1() == Relationship.PARENT)
      .forEach(ch -> System.out.println("John has a child called " + ch.getValue2().name));
  }

  public Research(RelationshipBrowser browser)
  {
    List<Person> children = browser.findAllChildrenOf("John");
    for (Person child : children)
      System.out.println("John has a child called " + child.name);
  }
}

class DIPDemo
{
  public static void main(String[] args)
  {
    Person parent = new Person("John");
    Person child1 = new Person("Chris");
    Person child2 = new Person("Matt");

    // low-level module
    Relationships relationships = new Relationships();
    relationships.addParentAndChild(parent, child1);
    relationships.addParentAndChild(parent, child2);

    new Research(relationships);
  }
}

```