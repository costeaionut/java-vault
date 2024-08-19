- Design patterns are common architectural approaches and well known *How to*'s.
- Design patterns are usually into three categories. This is called the Gamma categorization from the name `Erich Gamma` one of the original authors of the Gang Of Four book:
	- **Creational Patterns**
	- **Structural Patterns**
	- **Behavioral Patterns**

#### Creational Patterns
- These patterns are useful when we want to have a specific way of creating an object.
- [Builder](Builder) pattern allows us to have a brick by brick kind of construction method.
- [[Factory]] pattern allows us to have dedicated methods to obtain an object whole.
- [[Prototype]] pattern helps us build an object based on another existing object. It mostly looks into ways on how to deep copy an object.
- [[Basic Singleton|Singleton]] pattern helps us define an object that can be instantiated only once in our program. We also explore methods of how to make this pattern [[Singleton Thread Safety|thread safe]].
- [[Multiton]] patter - a variation of singleton that helps us define a finite number of instances based on some characteristics of the system.
#### Structural Patterns
- These patterns are controlling the way we want to structure and design our object.
- Usually heavy focused on the design part.
#### Behavioral Patterns
- This category of patterns usually look at how to solve/fix a certain specific problem.
- They are not really connected in a particular way.
