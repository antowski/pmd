---
title: Controversial
summary: The Controversial ruleset contains rules that, for whatever reason, are considered controversial. They are held here to allow people to include them as they see fit within their custom rulesets.
permalink: pmd_rules_java_controversial.html
folder: pmd/rules/java
sidebaractiveurl: /pmd_rules_java.html
editmepath: ../pmd-java/src/main/resources/rulesets/java/controversial.xml
---
## AssignmentInOperand

**Since:** PMD 1.03

**Priority:** Medium (3)

Avoid assignments in operands; this can make code more complicated and harder to read.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.controversial.AssignmentInOperandRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/controversial/AssignmentInOperandRule.java)

**Example(s):**

```
public void bar() {
    int x = 2;
    if ((x = getX()) == 3) {
      System.out.println("3!");
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|allowIncrementDecrement|false|Allow increment or decrement operators within the conditional expression of an if, for, or while statement|
|allowWhile|false|Allow assignment within the conditional expression of a while statement|
|allowFor|false|Allow assignment within the conditional expression of a for statement|
|allowIf|false|Allow assignment within the conditional expression of an if statement|

## AtLeastOneConstructor

**Since:** PMD 1.04

**Priority:** Medium (3)

Each class should declare at least one constructor.

```
//ClassOrInterfaceDeclaration[
  not(ClassOrInterfaceBody/ClassOrInterfaceBodyDeclaration/ConstructorDeclaration)
  and
  (@Static = 'false')
  and
  (count(./descendant::MethodDeclaration[@Static = 'true']) < 1)
]
  [@Interface='false']
```

**Example(s):**

```
public class Foo {
   // missing constructor
  public void doSomething() { ... }
  public void doOtherThing { ... }
}
```

## AvoidAccessibilityAlteration

**Since:** PMD 4.1

**Priority:** Medium (3)

Methods such as getDeclaredConstructors(), getDeclaredConstructor(Class[]) and setAccessible(),
as the interface PrivilegedAction, allows for the runtime alteration of variable, class, or
method visibility, even if they are private. This violates the principle of encapsulation.

```
//PrimaryExpression[
                        (
                        (PrimarySuffix[
                                ends-with(@Image,'getDeclaredConstructors')
                                        or
                                ends-with(@Image,'getDeclaredConstructor')
                                        or
                                ends-with(@Image,'setAccessible')
                                ])
                        or
                        (PrimaryPrefix/Name[
                                ends-with(@Image,'getDeclaredConstructor')
                                or
                                ends-with(@Image,'getDeclaredConstructors')
                                or
                                starts-with(@Image,'AccessibleObject.setAccessible')
                                ])
                        )
                        and
                        (//ImportDeclaration/Name[
                                contains(@Image,'java.security.PrivilegedAction')])
                ]
```

**Example(s):**

```
import java.lang.reflect.AccessibleObject;
import java.lang.reflect.Method;
import java.security.PrivilegedAction;

public class Violation {
  public void invalidCallsInMethod() throws SecurityException, NoSuchMethodException {
    // Possible call to forbidden getDeclaredConstructors
    Class[] arrayOfClass = new Class[1];
    this.getClass().getDeclaredConstructors();
    this.getClass().getDeclaredConstructor(arrayOfClass);
    Class clazz = this.getClass();
    clazz.getDeclaredConstructor(arrayOfClass);
    clazz.getDeclaredConstructors();
      // Possible call to forbidden setAccessible
    clazz.getMethod("", arrayOfClass).setAccessible(false);
    AccessibleObject.setAccessible(null, false);
    Method.setAccessible(null, false);
    Method[] methodsArray = clazz.getMethods();
    int nbMethod;
    for ( nbMethod = 0; nbMethod < methodsArray.length; nbMethod++ ) {
      methodsArray[nbMethod].setAccessible(false);
    }

      // Possible call to forbidden PrivilegedAction
    PrivilegedAction priv = (PrivilegedAction) new Object(); priv.run();
  }
}
```

## AvoidFinalLocalVariable

**Since:** PMD 4.1

**Priority:** Medium (3)

Avoid using final local variables, turn them into fields.

```
//LocalVariableDeclaration[
  @Final = 'true'
  and not(../../ForStatement)
  and
  (
    (count(VariableDeclarator/VariableInitializer) = 0)
    or
    (VariableDeclarator/VariableInitializer/Expression/PrimaryExpression/PrimaryPrefix/Literal)
  )
]
```

**Example(s):**

```
public class MyClass {
    public void foo() {
        final String finalLocalVariable;
    }
}
```

## AvoidLiteralsInIfCondition

**Since:** PMD 4.2.6

**Priority:** Medium (3)

Avoid using hard-coded literals in conditional statements. By declaring them as static variables
or private members with descriptive names maintainability is enhanced. By default, the literals "-1" and "0" are ignored.
More exceptions can be defined with the property "ignoreMagicNumbers".

```
//IfStatement/Expression/*/PrimaryExpression/PrimaryPrefix/Literal
[not(NullLiteral)]
[not(BooleanLiteral)]
[empty(index-of(tokenize($ignoreMagicNumbers, ','), @Image))]
```

**Example(s):**

```
private static final int MAX_NUMBER_OF_REQUESTS = 10;

public void checkRequests() {

    if (i == 10) {                        // magic number, buried in a method
      doSomething();
    }

    if (i == MAX_NUMBER_OF_REQUESTS) {    // preferred approach
      doSomething();
    }

    if (aString.indexOf('.') != -1) {}     // magic number -1, by default ignored
    if (aString.indexOf('.') >= 0) { }     // alternative approach

    if (aDouble > 0.0) {}                  // magic number 0.0
    if (aDouble >= Double.MIN_VALUE) {}    // preferred approach
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|ignoreMagicNumbers|-1,0|Comma-separated list of magic numbers, that should be ignored|

## AvoidPrefixingMethodParameters

**Since:** PMD 5.0

**Priority:** Medium Low (4)

Prefixing parameters by 'in' or 'out' pollutes the name of the parameters and reduces code readability.
To indicate whether or not a parameter will be modify in a method, its better to document method
behavior with Javadoc.

```
//MethodDeclaration/MethodDeclarator/FormalParameters/FormalParameter/VariableDeclaratorId[
        pmd:matches(@Image,'^in[A-Z].*','^out[A-Z].*','^in$','^out$')
]
```

**Example(s):**

```
// Not really clear
public class Foo {
  public void bar(
      int inLeftOperand,
      Result outRightOperand) {
      outRightOperand.setValue(inLeftOperand * outRightOperand.getValue());
  }
}
```

```
// Far more useful
public class Foo {
  /**
   *
   * @param leftOperand, (purpose), not modified by method.
   * @param rightOperand (purpose), will be modified by the method: contains the result.
   */
  public void bar(
        int leftOperand,
        Result rightOperand) {
        rightOperand.setValue(leftOperand * rightOperand.getValue());
  }
}
```

## AvoidUsingNativeCode

**Since:** PMD 4.1

**Priority:** Medium High (2)

Unnecessary reliance on Java Native Interface (JNI) calls directly reduces application portability
and increases the maintenance burden.

```
//Name[starts-with(@Image,'System.loadLibrary')]
```

**Example(s):**

```
public class SomeJNIClass {

     public SomeJNIClass() {
         System.loadLibrary("nativelib");
     }

     static {
         System.loadLibrary("nativelib");
         }

     public void invalidCallsInMethod() throws SecurityException, NoSuchMethodException {
         System.loadLibrary("nativelib");
     }
}
```

## AvoidUsingShortType

**Since:** PMD 4.1

**Priority:** High (1)

Java uses the 'short' type to reduce memory usage, not to optimize calculation. In fact, the JVM does not have any
arithmetic capabilities for the short type: the JVM must convert the short into an int, do the proper calculation
and convert the int back to a short. Thus any storage gains found through use of the 'short' type may be offset by
adverse impacts on performance.

```
//PrimitiveType[@Image = 'short'][name(../..) != 'CastExpression']
```

**Example(s):**

```
public class UsingShort {
   private short doNotUseShort = 0;

   public UsingShort() {
    short shouldNotBeUsed = 1;
    doNotUseShort += shouldNotBeUsed;
  }
}
```

## AvoidUsingVolatile

**Since:** PMD 4.1

**Priority:** Medium High (2)

Use of the keyword 'volatile' is generally used to fine tune a Java application, and therefore, requires
a good expertise of the Java Memory Model. Moreover, its range of action is somewhat misknown. Therefore,
the volatile keyword should not be used for maintenance purpose and portability.

```
//FieldDeclaration[
                                contains(@Volatile,'true')
                        ]
```

**Example(s):**

```
public class ThrDeux {
  private volatile String var1;	// not suggested
  private          String var2;	// preferred
}
```

## CallSuperInConstructor

**Since:** PMD 3.0

**Priority:** Medium (3)

It is a good practice to call super() in a constructor. If super() is not called but
another constructor (such as an overloaded constructor) is called, this rule will not report it.

```
//ClassOrInterfaceDeclaration[ count (ExtendsList/*) > 0 ]
/ClassOrInterfaceBody
 /ClassOrInterfaceBodyDeclaration
 /ConstructorDeclaration[ count (.//ExplicitConstructorInvocation)=0 ]
```

**Example(s):**

```
public class Foo extends Bar{
  public Foo() {
   // call the constructor of Bar
   super();
  }
 public Foo(int code) {
  // do something with code
   this();
   // no problem with this
  }
}
```

## DataflowAnomalyAnalysis

**Since:** PMD 3.9

**Priority:** Low (5)

The dataflow analysis tracks local definitions, undefinitions and references to variables on different paths on the data flow.
From those informations there can be found various problems.

1. UR - Anomaly: There is a reference to a variable that was not defined before. This is a bug and leads to an error.
2. DU - Anomaly: A recently defined variable is undefined. These anomalies may appear in normal source text.
3. DD - Anomaly: A recently defined variable is redefined. This is ominous but don't have to be a bug.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.controversial.DataflowAnomalyAnalysisRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/controversial/DataflowAnomalyAnalysisRule.java)

**Example(s):**

```
public void foo() {
  int buz = 5;
  buz = 6; // redefinition of buz -> dd-anomaly
  foo(buz);
  buz = 2;
} // buz is undefined when leaving scope -> du-anomaly
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|maxViolations|100|Maximum number of anomalies per class|
|maxPaths|1000|Maximum number of checked paths per method. A lower value will increase the performance of the rule but may decrease anomalies found.|

## DefaultPackage

**Since:** PMD 3.4

**Priority:** Medium (3)

Use explicit scoping instead of accidental usage of default package private level.
The rule allows methods and fields annotated with Guava's @VisibleForTesting.

```
//ClassOrInterfaceDeclaration[@Interface='false']
/ClassOrInterfaceBody
/ClassOrInterfaceBodyDeclaration
[not(Annotation//Name[ends-with(@Image, 'VisibleForTesting')])]
[
FieldDeclaration[@PackagePrivate='true']
or MethodDeclaration[@PackagePrivate='true']
]
```

## DoNotCallGarbageCollectionExplicitly

**Since:** PMD 4.2

**Priority:** Medium High (2)

Calls to System.gc(), Runtime.getRuntime().gc(), and System.runFinalization() are not advised. Code should have the
same behavior whether the garbage collection is disabled using the option -Xdisableexplicitgc or not.
Moreover, "modern" jvms do a very good job handling garbage collections. If memory usage issues unrelated to memory
leaks develop within an application, it should be dealt with JVM options rather than within the code itself.

```
//Name[
(starts-with(@Image, 'System.') and
(starts-with(@Image, 'System.gc') or
starts-with(@Image, 'System.runFinalization'))) or
(
starts-with(@Image,'Runtime.getRuntime') and
../../PrimarySuffix[ends-with(@Image,'gc')]
)
]
```

**Example(s):**

```
public class GCCall {
    public GCCall() {
        // Explicit gc call !
        System.gc();
    }

    public void doSomething() {
        // Explicit gc call !
        Runtime.getRuntime().gc();
    }

    public explicitGCcall() {
        // Explicit gc call !
        System.gc();
    }

    public void doSomething() {
        // Explicit gc call !
        Runtime.getRuntime().gc();
    }
}
```

## DontImportSun

**Since:** PMD 1.5

**Priority:** Medium Low (4)

Avoid importing anything from the 'sun.*' packages.  These packages are not portable and are likely to change.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.controversial.DontImportSunRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/controversial/DontImportSunRule.java)

**Example(s):**

```
import sun.misc.foo;
public class Foo {}
```

## NullAssignment

**Since:** PMD 1.02

**Priority:** Medium (3)

Assigning a "null" to a variable (outside of its declaration) is usually bad form.  Sometimes, this type
of assignment is an indication that the programmer doesn't completely understand what is going on in the code.

NOTE: This sort of assignment may used in some cases to dereference objects and encourage garbage collection.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.controversial.NullAssignmentRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/controversial/NullAssignmentRule.java)

**Example(s):**

```
public void bar() {
  Object x = null; // this is OK
  x = new Object();
     // big, complex piece of code here
  x = null; // this is not required
     // big, complex piece of code here
}
```

## OneDeclarationPerLine

**Since:** PMD 5.0

**Priority:** Medium Low (4)

Java allows the use of several variables declaration of the same type on one line. However, it
can lead to quite messy code. This rule looks for several declarations on the same line.

```
//LocalVariableDeclaration
   [count(VariableDeclarator) > 1]
   [$strictMode or count(distinct-values(VariableDeclarator/@BeginLine)) != count(VariableDeclarator)]
```

**Example(s):**

```
String name;            // separate declarations
String lastname;

String name, lastname;  // combined declaration, a violation

String name,
       lastname;        // combined declaration on multiple lines, no violation by default.
                        // Set property strictMode to true to mark this as violation.
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|strictMode|false|If true, mark combined declaration even if the declarations are on separate lines.|

## OnlyOneReturn

**Since:** PMD 1.0

**Priority:** Medium (3)

A method should have only one exit point, and that should be the last statement in the method.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.controversial.OnlyOneReturnRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/controversial/OnlyOneReturnRule.java)

**Example(s):**

```
public class OneReturnOnly1 {
  public void foo(int x) {
    if (x > 0) {
      return "hey";   // first exit
    }
    return "hi";	// second exit
  }
}
```

## SuspiciousOctalEscape

**Since:** PMD 1.5

**Priority:** Medium (3)

A suspicious octal escape sequence was found inside a String literal.
The Java language specification (section 3.10.6) says an octal
escape sequence inside a literal String shall consist of a backslash
followed by:

   OctalDigit | OctalDigit OctalDigit | ZeroToThree OctalDigit OctalDigit

Any octal escape sequence followed by non-octal digits can be confusing,
e.g. "\038" is interpreted as the octal escape sequence "\03" followed by
the literal character "8".

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.controversial.SuspiciousOctalEscapeRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/controversial/SuspiciousOctalEscapeRule.java)

**Example(s):**

```
public void foo() {
  // interpreted as octal 12, followed by character '8'
  System.out.println("suspicious: \128");
}
```

## UnnecessaryConstructor

**Since:** PMD 1.0

**Priority:** Medium (3)

This rule detects when a constructor is not necessary; i.e., when there is only one constructor,
its public, has an empty body, and takes no arguments.

```
//ClassOrInterfaceBody[count(ClassOrInterfaceBodyDeclaration/ConstructorDeclaration)=1]
/ClassOrInterfaceBodyDeclaration/ConstructorDeclaration
[@Public='true']
[not(FormalParameters/*)]
[not(BlockStatement)]
[not(NameList)]
[count(ExplicitConstructorInvocation/Arguments/ArgumentList/Expression)=0]
```

**Example(s):**

```
public class Foo {
  public Foo() {}
}
```

## UnnecessaryParentheses

**Since:** PMD 3.1

**Priority:** Medium (3)

Sometimes expressions are wrapped in unnecessary parentheses, making them look like function calls.

```
//Expression
           /PrimaryExpression
            /PrimaryPrefix
             /Expression[count(*)=1]
              /PrimaryExpression
              /PrimaryPrefix
```

**Example(s):**

```
public class Foo {
   boolean bar() {
      return (true);
      }
}
```

## UseConcurrentHashMap

**Since:** PMD 4.2.6

**Priority:** Medium (3)

Since Java5 brought a new implementation of the Map designed for multi-threaded access, you can
perform efficient map reads without blocking other threads.

```
//Type[../VariableDeclarator/VariableInitializer//AllocationExpression/ClassOrInterfaceType[@Image != 'ConcurrentHashMap']]
/ReferenceType/ClassOrInterfaceType[@Image = 'Map']
```

**Example(s):**

```
public class ConcurrentApp {
  public void getMyInstance() {
    Map map1 = new HashMap(); 	// fine for single-threaded access
    Map map2 = new ConcurrentHashMap();  // preferred for use with multiple threads

    // the following case will be ignored by this rule
    Map map3 = someModule.methodThatReturnMap(); // might be OK, if the returned map is already thread-safe
  }
}
```

## UseObjectForClearerAPI

**Since:** PMD 4.2.6

**Priority:** Medium (3)

When you write a public method, you should be thinking in terms of an API. If your method is public, it means other class
will use it, therefore, you want (or need) to offer a comprehensive and evolutive API. If you pass a lot of information
as a simple series of Strings, you may think of using an Object to represent all those information. You'll get a simpler
API (such as doWork(Workload workload), rather than a tedious series of Strings) and more importantly, if you need at some
point to pass extra data, you'll be able to do so by simply modifying or extending Workload without any modification to
your API.

```
//MethodDeclaration[@Public]/MethodDeclarator/FormalParameters[
     count(FormalParameter/Type/ReferenceType/ClassOrInterfaceType[@Image = 'String']) > 3
]
```

**Example(s):**

```
public class MyClass {
  public void connect(String username,
    String pssd,
    String databaseName,
    String databaseAdress)
    // Instead of those parameters object
    // would ensure a cleaner API and permit
    // to add extra data transparently (no code change):
    // void connect(UserData data);
    {

  }
}
```

