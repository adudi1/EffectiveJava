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

---

# Item 10
* <b>Obey general contract when overridding equals</b>
* Do not override if:
  * Each instance of class is inherently unique. True for classes such as Thread that represent active entities rather than values.
  * If there is no need for the class to provide logicial equality test.
  * Superclass has already overwritten equals and superclass behaviour is appropriate for this class.
  * Class is private or package private and you are sure the euals method will never be invoked.
* when to overwrite equals:
  * class has a notion of logical equaliity that differs from mere object identity and a superclass has not already overridden equals. Typically a value class.
  * A value class that does not require the equals method to be overridden is a class that is instance controled to ensure at most one object exsits with each value. (Enum types)
* Overriding equals must adhere to general contract, implements equivalence relation
  * Reflexive: For any non null reference value x, `x.equals(x)` must return true.
    * If voilated, the contatins method may just as well say the collection doesn't contain the instance you just added.
  * Symmetric: For any non null reference value x and y, `x.equals(y)` must return true only if `y.equals(x)` returns true.
  * Transitive: For any non null reference value x, y and z, if `x.equals(y)` returns true and `y.equals(z)` returns true then `x.equals(z) must return true`.
  * Consistent: For any non-null reference values x and y, multiple invokations of x.equals(y) must consistently return true or consistently false, provided no information used in equals contract changes.
    * mutable objects should remain equal or remain unequal as long as objects do not change. 
    * immutable objects shoule remain equal or remain unequal for all time.
  * For a non null reference x, `x.equals(null)` nust return false.
* If the equals contract is voilated the program may crash or behave eratically.
* Equivalence relation is an operator that partitions a set of elements into subsets whose elements are deemed equal to one another. These subsets are known as equivalence classes. From a user perspective, all equivalence classes must be interchangable.
* <b>There is no way to extend an instantiable class and add a value compoenent while preserving the equals contract.</b>
* Favour composition over inheritance.
* Bummer: Timestamps and Dates must never be intermixed.
* Do not write equals method that depends on unreliable resources. (URL equals depending on ip address). Equals method should only perform deterministic computations on memory resident objects.
* equals method must use instanceof operator to check that its argument is of correct type.
* Receipe for high quality equals method:
  * use == operator to check if argument is a reference to this object
  * use instanceof to check if he argument has the correct type
  * cast the argument to its correct type
  * For each significant field in a class, check if that field of an argument matches corresponding field of this object.
  
 ` @override public boolean equals(Object o) {
    if (this == o)
        return true;
    if (!(o instanceof PhoneNumber))
        return false;
    PhoneNumber pn = (PhoneNumber)o;
    return pn.lineNum == linenNum && pn.prefix == prefix && pn.areacode == areacode; 
 }`
* Always override hashcode whenever you override equals
* Do not substitue Object for another type in equals declaration
* make sure of using override annotation, it will prevent you from making mistakes.
* Autovalue framework automatically generates these methods.

---
# Item 11

* <b>Always override hashcode when you override equals</b>
* If two objects are equal according to the equals(object) method, then calling hashcode on the object must produce same integer result.
* Its not required that hashcode to produce different values when two objects are equal, however that will improve the performance.
* Recipe:
  * Declare the int variable named result, and initialize it to the hashcode c for the significant field of your object.
  * For every remaining significant field f of the object:
    * compute int hashcode c for the field
    * If premitive type, compute Type.hashcode(f), Type is boxed primitive class of f.
    * If f is object, recursively invoke hashcode on the field. More complex is canonical representation. If null, use 0.
    * If the field is an array, treat as if each significant element were a seperate field.
    * combine the hashcode c computed as
    `result = 31 * result +c;`
  * return result
  * (A nice property of 31 is multiplication can be replaced by a shift and substraction for better performance on some architectures: `31*i == (i << 5) - i `)
* 
` @override public int hashcode() { 
int result = Short.hashcode(areacode);
result = 31 * result + Short.hashcode(prefix);
result = 31 * result + Short.hashcode(lineNum);
return result;}`
* Objects has static method hash that takes arbitary number of objects, however it is slow.
`return Objects.hash(areacode, prefix, lineNum);`
* Do not be tempted to exclude significant fields from hashcode computation to improve performance.
* Don't provide a detailed specification for the value returned by hashcode, so clients can't resonably depend on it, this gives a felibility to change it.

---

# Item 12

* <b>Always override toString</b>
* providing a good toString implementation makes your class much more pleasant to use and makes systems using this class easier to debug.
* A disadvantage of specifing format for the toString is, it is stuck forever, you will not have the flexibility to change it later.
* clearly document intentions
* provide access to the information contained in the value returned by toString
* makes no sense to write toString for atic utility classes or enums
* good to write for abstract classes whose subclasses share common tring representation
* override object's toString implementation in every instantiable class, unless a superclass has already done so
---

# Item 13

* <b>Override clone judiciously</b>
* Clonable interface has flaw, it lacks clone method. 
* Clonable interface determines the beahviour of Object's protected clone implementation: If a class implements Cloneable, Object's clone method returns a field-by-field copy of the object, otherwise it returns cloneNotSupported exception. (atypical use of interface, discouraged) 
* In practice, a class implementing Clonable is expected to provide a properly functioning public clone method. 
* If the superclass provides a well behaved clone method. First call super.clone.
* Immutable classes should never provide a clone method
* Like constructor, clone method should never invoke an overridable method on the clone under construction.
* all classes that implement clonable hould override clone with a public method whose return type is class itself. This method should first call super.clone() and then fix any fields that needs fixing. 
* A better approach to object copying is to provide a copy constructor or copy factory.
---

# Item 14
* <b>Consider implementing Comparable</b>
* compareTo is sole method in Comparable interface.
* By implementing comparable the class indicates it contents have natural ordering.
* CompareTo return -1, 0 or +1. Throws classCastException if the specified object type prevents it form being comapred to this object.
  * x.compareTo(y) throws an expection only if y.comapreTo(x) throws exception.
  * `(x.compareTo(y) == 0) == x.equals(y)` is strongly recommened, but not required. Classes that voilate this fact should clearly indicate it (This class has natural ordering that is inconsistent with equals).
* There is no way to extend an instantiable class with a new value component while preserving the comapreTo contract. Write an unrelated class containing an instance of the first class. Then provide the view method to return the contained instance. 
* Avoid use of < amd > operators. Instead use static compare methods in the boxed prmitive classes or the comparator construction methods in the Comparator interface.
  
 ---
 // chapter 4
 
 # Item 15
 * <b> Minimize the accessibility of classes and members </b>
 * A well designed API will hide all its details, cleanly seperating its API from implementation details. (Information hiding or Encapulation)
 * + decouples the components that compromise the system.
 * + components can be developed in parallel (isolation)
 * + Decreases rick in developing large systems, as each individual component may prove successful even if the whole system isn't.
 * Thumb Rule: Make each class and member as inaccessible as possible.
 * If a top level class or an interface can be made package-private then it should be. (public is the other option for top level classes and interfaces (non nested)). So you can make it part of implementation, you can modify, replace or remove in subsequent releases, if public you have to support it forever.
 * If a package-private top level class is only used by only one class, consider making the top-level class a private static nested class of the sole class that uses it.
 * private—The member is accessible only from the top-level class where it is declared
 * package-private—The member is accessible from any class in the package where it is declared. Technically known as default access, this is the access level you get if no access modifier is specified (except for interface members, which are public by default).
 * protected—The member is accessible from subclasses of the class where it is declared and from any class in the package where it is declared.
 * public—The member is accessible from anywhere.
 * both private and package-private members are part of a class’s implementation and do not normally impact its exported API. These fields can, however, “leak” into the exported API if the class implements Serializable.
 * If a method overrides a superclass method, it cannot have a more restrictive access level in the subclass than in the superclass. (the Liskov substitution principle)
 * if a class implements an interface, all of the class methods that are in the interface must be declared public in the class
 * it is acceptable to make a private member of a public class package-private in order to test it, but it is not acceptable to raise the accessibility any higher.
 * classes with public mutable fields are not generally thread-safe
 * Even if a field is final and refers to an immutable object, by making it public you give up the flexibility to switch to a new internal data representation in which the field does not exist.
 *  You can expose constants via public static final fields. is critical that these fields contain either primitive values or references to immutable objects, a field containing a reference to a mutable object has all the disadvantages of a nonfinal field.
 * Note that a nonzero-length array is always mutable, so it is wrong for a class to have a public static final array field, or an accessor that returns such a field. (frequent source of security holes)
 * Beware of the fact that some IDEs generate accessors that return references to private array fields
 Fix: `private static final Thing[] PRIVATE_VALUES = { ... };

public static final List<Thing> VALUES =

   Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));`
 or
  `private static final Thing[] PRIVATE_VALUES = { ... };

public static final Thing[] values() {

    return PRIVATE_VALUES.clone();

}`
* Java 9: A module is a grouping of packages, like a package is a grouping of classes.
* Public and protected members of unexported packages in a module are inaccessible outside the module; within the module, accessibility is unaffected by export declarations
---

# Item 16
* <b>In public classes, use accessor methods not public fields</b>
* degenerate classes: that serve no purpose other than to group instance fields
* 


 
 
