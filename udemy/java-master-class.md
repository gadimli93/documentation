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
- boolean => 1 byte =>
- byte => 8 bits, has a width of 8 => whole number
- char => single 16 bits => 
- short => 16 bits => whole number
- int => 32 bits => whole number
- long => 64 bits => whole numbers
- float => 32 bits(single-precision) => floating point numbers
- double => 64 bits(double-precision) => floating point numbers

-**package** - is way to orginize the projects. where _javamastering_ is subfolder of _nmg_. most companies uses their domain names reversed.

```java
javamastering.nmg => nmg.mastering
```

- **wrapper class** - java uses this class for all primitive types. in the case of int we use _Integer_, and it gives us ways to perform operations on _int_. such as MIN_VALUE, MAX_VALUE. 

```java
package nmg.javamastering;
public class Main {
    public static void main(String[] args) {

        int myValue = 10_000;

        int myMinIntValue = Integer.MIN_VALUE;
        int myMaxIntValue = Integer.MAX_VALUE;
        System.out.println("Integer Minimum value = " + myMinIntValue);
        System.out.println("Integer Maximum value = " + myMaxIntValue);
    }
}
```

- if we try to store vallue more that a data type can store, **overflow** will occur. reverse also true that called _underflow_.

```java   public static void main(String[] args) {

        int myValue = 10_000;

        int myMinIntValue = Integer.MIN_VALUE;
        int myMaxIntValue = Integer.MAX_VALUE;
        System.out.println("Integer Minimum value = " + myMinIntValue);
        System.out.println("Integer Maximum value = " + myMaxIntValue);

        System.out.println("Busted MAX value = " +(myMaxIntValue + 1) );
        System.out.println("Busted MIN value = " +(myMinIntValue - 1) );

    }
```

-output:
```
Integer Minimum value = -2147483648
Integer Maximum value = 2147483647
Busted MAX value = -2147483648
Busted MIN value = 2147483647
```

## casting

- **casting** - is treat or convert a number from one type to another. put a type in front of number in paranthesis;
```
(byte)(myMinByteValue / 2)
```
- you don't always need to cast the value with integer type of values (such as long and double) but you'll need with/from other.


### floarint numbers

- _precision_ - refers to the format and amount of space occupied by the type.
- _single precision_ - occupies 32 bits
- _double precision_ - occupies 64 bits

 











