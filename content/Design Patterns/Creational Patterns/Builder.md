#### 1. Overview

- Some objects can be constructed using a simple parametrized constructor, others might need some sort of ceremony to create like with the StringBuilder.
- It can be painful to use a constructor with 10 parameters.
- One way to approach this is to give users a brick by brick way of constructing objects.
- The Builder pattern lets us do this but be careful when creating it as to expose useful construction methods in your Builder API.

##### 2. Builder

- The builder is a piece-wise construction pattern that handles the construction of an object.

- The idea behind the builder is that you can create a logic to gradually construct a complex object. The most important thing is to expose easy and logical methods to add its compose the object.

- At its core the Builder will most likely need to have these functionalitie: an add method (to build the object), a reset method (to clear and reuse the builder) and a build method (to compose the object from its pieces)

```java

class HtmlElement
{
  public String name, text;
  public ArrayList<HtmlElement> elements = new ArrayList<HtmlElement>();
  private final int indentSize = 2;
  private final String newLine = System.lineSeparator();

  public HtmlElement()
  {
  }

  public HtmlElement(String name, String text)
  {
    this.name = name;
    this.text = text;
  }

  private String toStringImpl(int indent)
  {
    StringBuilder sb = new StringBuilder();
    String i = String.join("", Collections.nCopies(indent * indentSize, " "));
    sb.append(String.format("%s<%s>%s", i, name, newLine));
    if (text != null && !text.isEmpty())
    {
      sb.append(String.join("", Collections.nCopies(indentSize*(indent+1), " ")))
        .append(text)
        .append(newLine);
    }

    for (HtmlElement e : elements)
      sb.append(e.toStringImpl(indent + 1));

    sb.append(String.format("%s</%s>%s", i, name, newLine));
    return sb.toString();
  }

  @Override
  public String toString()
  {
    return toStringImpl(0);
  }
}

class HtmlBuilder
{
  private String rootName;
  private HtmlElement root = new HtmlElement();

  public HtmlBuilder(String rootName)
  {
    this.rootName = rootName;
    root.name = rootName;
  }

  // not fluent
  public void addChild(String childName, String childText)
  {
    HtmlElement e = new HtmlElement(childName, childText);
    root.elements.add(e);
  }

  public HtmlBuilder addChildFluent(String childName, String childText)
  {
    HtmlElement e = new HtmlElement(childName, childText);
    root.elements.add(e);
    return this;
  }

  public void clear()
  {
    root = new HtmlElement();
    root.name = rootName;
  }

  // delegating
  @Override
  public String toString()
  {
    return root.toString();
  }
}

class BuilderDemo
{
  public static void main(String[] args)
  {
    // we want to build a simple HTML paragraph
    System.out.println("Testing");
    String hello = "hello";
    StringBuilder sb = new StringBuilder();
    sb.append("<p>")
      .append(hello)
      .append("</p>"); // a builder!
    System.out.println(sb);

    // now we want to build a list with 2 words
    String [] words = {"hello", "world"};
    sb.setLength(0); // clear it
    sb.append("<ul>\n");
    for (String word: words)
    {
      // indentation management, line breaks and other evils
      sb.append(String.format("  <li>%s</li>\n", word));
    }
    sb.append("</ul>");
    System.out.println(sb);

    // ordinary non-fluent builder
    HtmlBuilder builder = new HtmlBuilder("ul");
    builder.addChild("li", "hello");
    builder.addChild("li", "world");
    System.out.println(builder);

    // fluent builder
    builder.clear();
    builder.addChildFluent("li", "hello")
      .addChildFluent("li", "world");
    System.out.println(builder);
  }
}

```

#### 3. Fluent builders

- the idea of a fluent builder is to allow you to chain operations on a builder. Instead of doing it builder.add() builder.add() we could make the add method return the builder which in turn allows to do things like this

```java

builder.add("s").add("ugi").add("_")

```

- One point of attention when working with Fluent builders is to make sure that these don't break in an inheritance chain.

- This is best seen in the example below but the idea is that we must make sure that the base builder will return the derived builder that we will use in its defined methods otherwise the derived builder's methods will not be usable.

- This can be achieved using the recursive generic templating in Java as seen below.

```java

// builder inheritance with recursive generics
class Person
{
  public String name;

  public String position;

  @Override
  public String toString()
  {
    return "Person{" +
      "name='" + name + '\'' +
      ", position='" + position + '\'' +
      '}';
  }
}

class PersonBuilder<SELF extends PersonBuilder<SELF>>
{
  protected Person person = new Person();

  // critical to return SELF here
  public SELF withName(String name)
  {
    person.name = name;
    return self();
  }

  protected SELF self()
  {
    // unchecked cast, but actually safe
    // proof: try sticking a non-PersonBuilder
    //        as SELF parameter; it won't work!
    return (SELF) this;
  }

  public Person build()
  {
    return person;
  }
}

class EmployeeBuilder extends PersonBuilder<EmployeeBuilder>
{
  public EmployeeBuilder worksAs(String position)
  {
    person.position = position;
    return self();
  }

  @Override
  protected EmployeeBuilder self()
  {
    return this;
  }
}

class RecursiveGenericsDemo
{
  public static void main(String[] args)
  {
    EmployeeBuilder eb = new EmployeeBuilder()
      .withName("Dmitri")
      .worksAs("Quantitative Analyst");
    System.out.println(eb.build());
  }
}

```

##### 4. Faceted Builder

- In some cases we might need to build objects that are quite complicated and it would make sense to have some separation. Or simply some parts of the base object are reused in other classes.
- In this case we can use a Faceted builder that makes sure that we have a fluent way of building our object.
- In my personal opinion we need to be sure that we make this as seamless as possible otherwise it might do more harm than good.

```java
class Person
{
  // address
  public String streetAddress, postcode, city;

  // employment
  public String companyName, position;
  public int annualIncome;

  @Override
  public String toString()
  {
    return "Person{" +
      "streetAddress='" + streetAddress + '\'' +
      ", postcode='" + postcode + '\'' +
      ", city='" + city + '\'' +
      ", companyName='" + companyName + '\'' +
      ", position='" + position + '\'' +
      ", annualIncome=" + annualIncome +
      '}';
  }
}

// builder facade
class PersonBuilder
{
  // the object we're going to build
  protected Person person = new Person(); // reference!

  public PersonJobBuilder works()
  {
    return new PersonJobBuilder(person);
  }

  public PersonAddressBuilder lives()
  {
    return new PersonAddressBuilder(person);
  }

  public Person build()
  {
    return person;
  }
}

class PersonAddressBuilder extends PersonBuilder
{
  public PersonAddressBuilder(Person person)
  {
    this.person = person;
  }

  public PersonAddressBuilder at(String streetAddress)
  {
    person.streetAddress = streetAddress;
    return this;
  }

  public PersonAddressBuilder withPostcode(String postcode)
  {
    person.postcode = postcode;
    return this;
  }

  public PersonAddressBuilder in(String city)
  {
    person.city = city;
    return this;
  }
}

class PersonJobBuilder extends PersonBuilder
{
  public PersonJobBuilder(Person person)
  {
    this.person = person;
  }

  public PersonJobBuilder at(String companyName)
  {
    person.companyName = companyName;
    return this;
  }

  public PersonJobBuilder asA(String position)
  {
    person.position = position;
    return this;
  }

  public PersonJobBuilder earning(int annualIncome)
  {
    person.annualIncome = annualIncome;
    return this;
  }
}

class BuilderFacetsDemo
{
  public static void main(String[] args)
  {
    PersonBuilder pb = new PersonBuilder();
    Person person = pb
      .lives()
        .at("123 London Road")
        .in("London")
        .withPostcode("SW12BC")
      .works()
        .at("Fabrikam")
        .asA("Engineer")
        .earning(123000)
      .build();
    System.out.println(person);
  }
}

```