# Java A-Z Master class notes

## Chapter 3

- **public** - is access modifier keyword. access modified allows ud to define thr scope or how other parts of code or even some else's code can access this code. 
- **class** - used to define a class with the _name_ following the keyword -> Hello in this case and left& right curky braces to define the class block.

```java
public class Hello {
}
```
- **method** - is a collection of statements that performs an operation.
-**main** - is a special method and entry point of java code, that jave looks for when running the program.

- **static** - is java keyword => 
- **void** - java keyword, means that method will not return anthing
- **parameters** - inside the paranthesis
- **System.out.println("Hello");** - _sout_ to print the string
- ** method's { } ** -> are code block
- **semicolo ;** -  is used to let java know that line is completed
```java
public class Hello {
  public static void main(String[] args) {
  System.out.println("Hello");
}
}
```

### Variables

Variables - are the way to store the information. Variables define by data types.



#### Data Types
 - **int** - integer data type used to define the numbers.
 - **equal sign = ** - used to initialize the variable, its optional.
 - **expressions** -  are used to simplified the formulas and re-used of variables again.

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");

        int myFirstNumber = 5;
        System.out.println(myFirstNumber);

        int mySecondNumber = 12;
        int myThirdNumber = 6;
        System.out.println( mySecondNumber + " and " + myThirdNumber);
        int myTotal = mySecondNumber + myThirdNumber + myFirstNumber;
        System.out.println(myTotal);
        int myLastOne = 1000 - myTotal;
        System.out.println(myLastOne);

    }
}
```

### Primitive Data Types
These are the primitive types in thava decs -> asc order by size.
- boolean
- byte
- char
- short
- int
- long
- float
- double


-**package** - is way to orginize the projects. where _javamastering_ is subfolder of _nmg_. most companies uses their domain names reversed.
```
javamastering.nmg => nmg.mastering
```

 







