# Scope

## Learning Goals

- Introduce Variable Scope Rules
- Demonstrate Variable Shadowing
- Introduce Accessor and Mutator Methods
- Use IntelliJ to Generate Getter and Setter Methods

## Introduction

We have seen several examples where new curly brace blocks can be defined, and
every time a new set of matching `{` and `}` is created, it defines a new scope.
This means that the variables that are defined inside that scope are only available
for use within that scope.

# Scope Rules

A block is a set of statements enclosed in set braces `{` `}`.
Java has many scope rules, but we will focus on the following three:

- The scope of an instance or static variable is the entire class block
- The scope of a formal parameter of a method is the entire method block
- The scope of a local variable declaration in a block is the rest of the block
  in which the declaration appears, starting at its own initializer and including further
  declarations to the right in the declaration statement


## Scope Walkthrough

Consider the following code.  The print statement lists the variables that are within scope
for the current block.

```java
import java.util.Arrays;

public class ScopeExample { // <- first scope

  private static int classLevelVariable = 1;

  public static void main(String[] args) { // <- second scope

    int methodLevelVariable = 2;

    //50% chance of true
    if (Math.random() < 0.5) { // <- third scope
      int ifBlockVariable = 3;
      System.out.printf("if block scope: %d %d %s %d %n" , ifBlockVariable, methodLevelVariable, Arrays.toString(args), classLevelVariable);
    } // <- end of third scope

    else { // <- fourth scope
      int elseBlockVariable = 4;
      System.out.printf("else block scope: %d %d %s %d %n" , elseBlockVariable, methodLevelVariable, Arrays.toString(args), classLevelVariable);

    } // <- end of fourth scope

    System.out.printf("main block scope:  %d %s %d %n" , methodLevelVariable, Arrays.toString(args), classLevelVariable);

    anotherMethod(5);

  } // <- end of second scope

  public static void anotherMethod(int parameterVariable) {  // <- fifth scope
    System.out.printf("anotherMethod block scope: %d %d %n" , classLevelVariable, parameterVariable);
  }  // <- end of fifth scope

} // <- end of first scope
```

As we discussed before, curly brace "blocks" can be contained inside each other
like Russian dolls, which is the case here:

- The first scope is the top level scope, which means nothing can be defined
  outside it for our program.  
- The second scope is inside the first scope, which means the second scope has
  access to all the variables defined in the first scope
- The third scope is inside the second scope, which means the third scope has
  access to all the variables defined in the second scope, including the ones
  defined in the first scope
- The fourth scope is also inside the second scope, which means the fourth scope
  has access to all the variables defined in the second scope, including the
  ones defined in the first scope
- The fourth scope is a peer of the third scope, which means they cannot see
  each other's variables. A scope can only see its variables and the variables
  of all the enclosing scopes.
- The fifth scope is inside the first scope and a peer to the second scope,
  which means the fifth scope has access to all the variables defined in the first
  scope but cannot see the variables in the second scope.

In our example, the variables `ifBlockVariable` and `elseBlockVariable`
are each declared within their respective part of the `if-else` statement, so they
are not accessible outside either the `if` or the `else`.

If you run the program enough times it should eventually execute both branches
of the `if-else` statement, printing the variables in scope for the current block:

```java
if block scope: 3 2 [] 1 
main block scope:  2 [] 1 
anotherMethod block scope: 1 5 
```


```java
else block scope: 4 2 [] 1 
main block scope:  2 [] 1 
anotherMethod block scope: 1 5 
```

## Shadowing

- A *simple* name is a single identifier, such as `age` or `ssn`.
- A *qualified* name consists of a name, a `.`, and an identifier, such as `this.age` or `employee1.ssn`.

Consider the `Person` class:

```java
public class Person {
  private  int age;
  private String city;
  private String state;

  public void update(String city) {
    int age = 26;                    //local variable age shadows instance variable age
    city =  city.substring(0,5);     //parameter variable city shadows instance variable city
    state = "OH";                    //instance variable state is not shadowed so value is updated
  }

  @Override
  public String toString() {
    return "age=" + age + ", city=" + city  + ", state=" + state  ;
  }

  public static void main(String [] args) {
    Person p = new Person();
    p.update("Cincinnati");
    System.out.println(p);
  }
}
```

- Instance variables declared in class (enclosing scope): `age`, `city`, `state`.
- Local/parameter variables declared in `update` method (nested scope): `age`, `city`.

Shadowing happens when a variable is declared in one scope (the nested scope)
and has the same name as a variable declared in an enclosing scope.
In this case, the declaration in the nested scope
*shadows* the declaration of the enclosing scope.  The variable in the enclosing
scope can't be accessed in the nested scope using its simple name, we would need to use a qualified name.

A variable declared in the `update` method shadows the instance variable with the same name:

- The instance variable named `city` is shadowed by the parameter named `city`.
- The instance variable named `age` is shadowed by the local variable named `age`.
- The instance variable named `state` is not shadowed.

This means instance variables `city` and `age` can't be accessed using their simple
names, they must be accessed as `this.city` and `this.age`.  Any assignment using just
`city` or `age` is to the parameter or local variable, not the instance variable.

Let's step through the `update` method to observe variable shadowing in action.
Set a breakpoint at the first line of code in the `update` method.

![shadow set breakpoint](https://curriculum-content.s3.amazonaws.com/6676/java-methods/shadow_breakpoint.png)

Launch the debugger.  Observe the call stack contains the frame for the `update` method call with the 
parameter variable `city` initialized to the actual argument value "Cincinnati".  Since this is an
instance method, we also see `this` referencing the `Person` instance.  

![shadow breakpoint reached](https://curriculum-content.s3.amazonaws.com/6676/java-methods/shadow_breakpoint_reached.png)

Stepping through the `update` method, we see new values assigned
to the local variable `age` and parameter variable `city`,
rather than the instance variable with the same name:

| Code                           | Visualizer                                                                                                                                                                |
|--------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `int age = 26;`                | ![shadow update step1](https://curriculum-content.s3.amazonaws.com/6676/java-methods/shadow_update_step1.png) <br>Local variable <code>age</code> is assigned value.      |
| `city =  city.substring(0,5);` | ![shadow update step2](https://curriculum-content.s3.amazonaws.com/6676/java-methods/shadow_update_step2.png) <br>Parameter variable <code>city</code> is assigned value. | 
| `state = "OH";`                | ![shadow update step3](https://curriculum-content.s3.amazonaws.com/6676/java-methods/shadow_update_step3.png) <br>Instance variable <code>state</code> is assigned value. |

Returning from the `update` method to the `main` method, we see
the `state` instance variable was updated, but not `age` or `city`.

![shadow return to main](https://curriculum-content.s3.amazonaws.com/6676/java-methods/shadow_return.png)

Shadowing seems like it prevents us from accessing an instance variable.
However, we can use a *qualified* name such as `this.age` to refer to
a shadowed instance variable. While it may seem odd to declare a parameter
with the same name as an instance variable, this is in fact common
practice when defining constructor and mutator methods.  We'll
cover mutator methods in the next lesson.

## Conclusion

Java has many scope rules.  In this lesson, we focused on three:

- The scope of an instance or static variable is the entire class block
- The scope of a formal parameter of a method is the entire method block
- The scope of a local variable declaration in a block is the rest of the block
  in which the declaration appears, starting at its own initializer and including further
  declarations to the right in the declaration statement


## Resources

- [Java Variable Scope](https://www.baeldung.com/java-variable-scope)    
- [Scope of a Declaration](https://docs.oracle.com/javase/specs/jls/se19/html/jls-6.html#jls-6.3)
