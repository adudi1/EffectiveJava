# Effective Java by Joshua Bloch

* https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/
* https://www.amazon.com/Effective-Java-Joshua-Bloch/dp/0134685997?SubscriptionId=AKIAILSHYYTFIVPWUY6Q&tag=duckduckgo-osx-20&linkCode=xm2&camp=2025&creative=165953&creativeASIN=0134685997
---

# Item 1

* <b>Consider static factory methods instead of constructors</b>
* Simply a static method that returns an instance of a class
* \+ They have names, easier to read and use. (Solves: Class can have only one constructor with a given signature. Its a bad idea to have constructors only differing the order of parmenters)
* \+ They are not required to create a new object everytime they are invoked, unlike constructors. Classes that maintain strict control over what instances exist at any time are said to be instance controlled. (Solves: Guarantees/alows a class to be singleton or noninstantiable or for an immutable class that no two equal instances exist. Further reading: Flyweight pattern)
* \+ They can return an object of any subtype of their return type, unlike constructors. (An API can return objects without making their classes public. Java 8 allows static methods in interfaces. Implementation code for these staic methods may need to be kept in seperate pakage-private class as Java 8 requires all static members to be public.)
* \+ class of the returned object can vary depending on parameters. 
* \+ class of returned object need not exist when the class containing the method is written. Basis for service provider framework. [More](https://stackoverflow.com/questions/11823773/understanding-the-concept-behind-service-provider-framework-like-jdbc-using-the)
* \- classes cannot be subclasses without public or protected constructors. Pro: It encourages programmers to use composition instead of inheritence, required for immutable classes.
* \- hard for programmers to figure out how to instantiate a class they do not stand out like constructors. Common names for factory methods: form, of, valueOf, instance, getInstance, create, newInstance, get<type> ex: getFileStore, new<type>, <type>
* Avoid public constructors without first considering static factories.

----

# Item 2

* <b>Consider a builder when faced with many constructors</b>
* Static factories and constructors do not scale well to large number of optional parameters.
* Telescoping constructor pattern: a constructor with each optional parmenters. (Harder to write client code and hard to read.) 
* JavaBeans pattern: parameterless constructor to create the object and then call setter methods to set each parameter. (construction is split in multiple calls, object can be in inconsistent state partway. Class cannot be made immutable.)
* Builder pattern: client calls a constructor or static factory with all the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable. The builder is typically a static memeber class of the class it builds.
* well suited for class hierarchies. build method in each subclass can ve declared to return a correct subclass. (Generic builder with a recursive type parameter like Builder<T extends Builder<t>> returns self(). Simulated self type).
* A subclass method declared to return a subtyoe of the return type declared in the super-class, is known as covariant return typing.
* Its often better to start with a builder in the first place if there will ever be a need for it.
* more than a handful of parameters espically optional or identical type -> prefer builder.
  
---

# Item 3

* <b>Enfore the singleton property with a private constructor or an enum type</b>
* Singleton is simply a class that is instantiated exactly once, typically a stateless object like function or a unique system component.
* \- Singleton class can make it difficult to test its clients because its not possible to subtitite a mock implementation for a singleton unless it implements an interface that serves as its type.
* Method 1: Keep the constructor private. public static final member to provide access to the sole instance. (pro: clear it is a singleton and simpler) `public static final Elvis INSTANCE = new Elvis();`
  * One caveat is a priviledged client can invoke access to the private constructor reflectively with the aid of AccessibleObject.setAccessible method, to defend you would need to throw an exception in the constructor if attempted to create a second instance.
* Method 2: public member is static factory method. (Pro: felxibility to make it non singleton, method reference can be used as supplier, If these pros are not applicable use method 1. Same caveat as above.)
`private static final Elvis INSTANCE = new Elvis();
  public static Elvis getInstance {return INSTANCE;}`
* To make the singleton class using method1 or 2 serializable, not just sufficient to add implements singleton to declaration, declare all instance fields transient and provide readResolve method. Otherwise, each time a serialized instance is deserialized, a new instance will be created.
* Method 3: declare a single element enum. Best way! (Can't use this approach if the singleton must extend a superclass other than Enum (though an enum can be declared to implement interfaces)).
`public enum Elvis {
INSTANCE;
public void foo(){...}
}`

---

# Item 4
* <b>Enforce noninstantiability with a private constructor</b>
* Enforcing noninstantiability by making the class abstract doesn't work, misleads into thinking the class designed for inheritance.

---

# Item 5
* <b>Prefer Dependency Injection to hardwiring resources</b>
* Static utility classes and singletons are inappropriate for classes whose behaviour is parameterized by an underlying resource.
* equally applicable to constructors, static factories and builders
* inject resource or resource factory

----

# Item 6
* <b>avoid creating unnesseary objects</b>
  * don't: `String s = new String("hello');`
  * do: `String s = "hello";` (rather than creatinf a new object each time it is executed, ensures the object is reused by any other code running in the same virtual machine that happens to contain the same literal)
* If you need to create an expensive object repeatedly, advisable to cache it for reuse.
* String.matches(<regex>) is not suitable for performance critical situations. It internally creates Pattern instance for the regular expression, creating pattern instance is expensive beacuse it requires compiling the regular expression into a finite state machine.
* One way to create unnecessary objects is autoboxing, which allows the programmer to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types. Prefer prmitives to boxed prmitives and watchout for unintentional autoboxing. 
* Avoiding object creation by maintaing your own object pools is a bad idea unless the objects in the pools are extermely heavyweight (eg: database connection)
* creating objects unnecessarily merely affects style and performance, failing to make defensive copies where required can leas to insidious bugs and security holes.
---

# Item 7
* <b>Eliminate obsolete object references</b>
* Obsolete reference is simply that can never be dereferenced again
* One common source for memory leakes is caches. If you are lucky to implement a cache for which an entry is relevant exactly so long as there are <b>references to its key outside of the cache</b>, represent the cache as a WeakHashMap, entries will be removed automatically after they become obsolete.
* usually, lifetime of cache entry is less well defined, have a background thread to cleanse entries or as side effect of adding new entries to the cache. More sophisticated caches may have to use java.lang.ref.
* Another common source for memory leakes is listeners and call backs. If you implement an API where clients register callbacks but don't deregister them explicitly, they will accumulate unless you take some action. To ensure they are garbage collected, only store weak references, by storing them only as keys in a WeakHashMap.
---

# Item 8
* <b>Avoid Finalizers and Cleaners</b>
* No guarantee they will executed promptly. Never do anything time critical in them like close files.
* try-with-resources or try-finally may be used to reclaim other non memory resources.
* uncaught exceptions during finalization are ignored, finalization of object terminates, can lead to corrupt state.
* Performance penality
* Security issues, opens up class to finalizer attacks. The finalizer can store reference to the corrputed object in a static field, preventing it ffrom garbage collected. Arbitrary methods can be invoked on this object that should have never been allowed to exist in the first place. To protect non final classes from finalizer attacks, write a final finalize method that does nothing.
* Just have your class implement AutoClosable if it requires termination of files or threads, and require clites to call close method typically in try-with-resources.
* one legitimate use is to have safety net when the owner neglects to call close. Think long and hard if it is worth the cost.
* another conserning native peers.

---

# Item 9
* <b>Prefer Try-with-resources to Try-Finally</b>
* java has many library that must be manually closed by invoking close.
  * Input stream, Output stream, java.sql.Connection
* try-finally is ugly when used with more than one resource
* try-with-resources - best way to close reources, resource should implement AutoClosable interface.
