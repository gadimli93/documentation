# Javascript A to Z 

## Chapter-1

- **Javascript** - is high level object-oriented, multip paradigm programming language
  - pogramming language : intruct computer to _do_ things
  - high-level : not to concern with concepts like memory management and etc.
  - object-oriented : based on objects for storing most kinds of data
  - multi-paradigm : able to use different style programming, such as imperative and declarative

- Google chrome -> console experience

```js
alert(hello wordl)
```

### JS's role in Web Development

- HTML, CSS, JS are core concept of web developmnent, when work in sycn to create interactive& dynamic websites, web applciations
  - HTML(noun) : responsible for content
  - CSS(adjective) : responsible for presentation
  - JS(verb) : responsible for
    -  add dynamic effects to any webpage 
    -  manipulate the content or CSS
    -  load data from remote servers
    -  build entire application in the browser (called web applciation)
- Modern JS libraries and frameworks : helps to write modern large scale web applications a lot easier and faster, and best 100% on JS
  - React
  - Angular
  - Vue   
- JS language and web browser are two separate things
  - JS used on the web browser : front-end applications
  - JS used on web servers(using the technology below) : back-end applications
    - node.js : creates back-end applications that run on web server and interact with DB and etc.   
  - JS can be used for :  native mobile applications
    - technologies used along: react-native, ionic 
  - Js can be used for : native desktop application
    -  technologies used along: electron

- JS Releases : ECMAScript(ES)  
  - ES5
  - ES6 (2015) : biggest update to the language 
  - ES7 (2016) :
  - ES8 (2017) :
  - ES9 (2018) :
  - ES10 (2019) :
  - ES11 (2020) :
  - new updates every upcoming year
    - Modern JS: 2015 - 2020 

### Values and Variables
 - value : smalles unit of information or data
 - every value : either OBJECT or PRIMITIVE (not object)
 
 ```js
 //oject
 let me = {
  name = 'nmg'
 }; 
 
 //primitive
 let myName = 'NMG';
 let age = 25;
 ```
 
 - 7 Primitive Data Types : 
  - Number :  floating point numbers, used for decimals and intigers
  - String : sequence of character, used for text
  - Booleans : logical type, that an only be 'true or false', used for taking decisions
  - undefined :  value taken by variable that is not yest defined ('empty value')
  - Null :  also means 'empty value'
  - Symbol (ES2015) : value that is unique and cannot be changed
  - BigInt(ES2020) : larger intigers than Number type can hold

```js
// number
let age = 25;

//string
let firstName = 'NMG';

//boolean
let fullName = true;

//undefinied
let children;
```
- JS has dynamic typing, meaning no need for manual data type assing.
  - it is the VALUE has the type, NOT the variable 
  - dynamic typing : meas we can change the data type of the variable.

```js
// dynamic typing

let jsIsFun = true;

console.log(jsIsFun);


jsIsFun = 25;
console.log(jsIsFun);
```

- use _let_ keyword to declare the variables
  - Re-assiging value also called **mutated the variable** => this term is commonly used in JS
- use _const_ keyword to declare the variable that cannot be change once declared
  - tech term: inmutable variable
  - by default use _const_ for declatarion to avoid the bugs
- _var_ keyword is old way to declare the variables.
  - _var_ and _let_ look similar but are different under surface
  - _var_ cannot mutate the variable
  - avoid using _var_   

```js
// below code will still work even withoud declaration
// bcz Js will automatically declare the variable in the GlobalObject
// not a good practise
lastName = 'NMG';
console.log(lastName);

```
 - Template Literals : can assemble multiple pieces into one final string

```js
const firstName = 'NMG';
const birthYear = 1993;
const myDetails = `I'm ${firstName}, a ${2022-birthYear} years old.`;
 console.log(myDetails);
 
 // new line => with \n\ -> no space or gap after \n\
 
 console.log(`string with \n\
 multiple \n\
 lines');
 
 //   new line => with ``
 console.log(`string with
 multiple
 lines`);
 

}
```
### Type Conversion and Coercion

- Type conversion : when manually converted from to another type (explisit)
- Type coercion : when JS automatically convert type to another (implisit)
- not every dat type can be converted to another
- type coersion can be open gate for bugs if not be careful


```js
// type conversion
const inputYear = '1991';
    console.log(Number(inputYear), inputYear);
    console.log(Number(inputYear) + 18);   

// NaN - not a Number, which has Number data type and which is invalid
console.log(Number('john'));
   console.log(typeof NaN);
   
// type coersion
    // template literal also change all value and turns them in String
    console.log('I.m ' + 23 + ' years old')
    console.log('I.m ' + '23' + ' years old')
    
// type coersion
    // template literal also change all value and turns them in String
    console.log('I.m ' + 23 + ' years old')
    console.log('I.m ' + '23' + ' years old')

// here JS automatically read them as Number
console.log('23' - '10' - 3);
// -> output: 10
 // Not everything can be turned to String
 console.log('23' + '10' + 3);
 // -> output : 23103
   
```
 ### Boolean coersion/convertion
 
 - in JS there is 5 falsy values, which will be the output when we will try to convert to boolean
  - 0
  - '' (empty string)
  - undefined
  - null
  - NaN
- any string that is not a empty string has a true value
- same apply to empty object `{}`

```js
console.log(Boolean(0));
    console.log(Boolean(undefined));
    console.log(Boolean('john'));
    console.log(Boolean({}));
    console.log(Boolean(''));
// output : 
false
false
true
true
false


const money = 0;
    if (money) {
        console.log("dont spend it all")
    } else {
        console.log("need a job")
    }
// output : false
    

```
- If- Else condition operator, no matter what we put in paranthesis, JS will always consider it as boolean, and 1st condition as true;

```js
let height;
    if (height) {
        console.log("defined")
    } else {
        console.log("undefined")
    }
```
### Equality operator
- == assign
- == (loose) equal  -> perorms coercion
- === (strict) equal -> does not perform coercion
  - try to avoid loose equal operator

```js
const age = 18;
if (age === 18) console.log('you are adult');

if(age == 18) console.log('you're adult');

const favourite = prompt('whats your favorite number ?');
console.log(favourite);
console.log(typeof favourite);

// -> output: string

if (favourite == 73) { // loose 
    console.log('great number!')
}
// -> output :  great number

if (favourite === 73) {
    console.log('great number!')
}

// -> output: ''


if (favourite === 73) {
    console.log('a superb number')
} else if (favourite === 37) {
    console.log('37 also a great number')
} 
else if (favourite === 7) {
   console.log('7 is a amazing all time great')  
} else {
    console.log('number 3 is a good number')
};
    
// !== => not equal operator

```
 ### Basic Boolean logic: AND, OR, NOT Operators
 
 - A(horizontal) AND B(vertical) => true when both are true
 
|-----|-------| -----|
AND   | TRUE  | FALSE 
|-----| ------| ----- 
TRUE  | TRUE  | FALSE |
FALSE | FALSE | FALSE |

- A(horizontal) OR B(vertical) => true when one is true
 
|-----|-------| -----|
OR    | TRUE  | FALSE
|-----| ------| ------
TRUE  | TRUE | TRUE  |
FALSE | TRUE | FALSE |

- A(horizontal) NOT (!) B(vertical) => can be either true/false
  - Not operator has a precedence over AND and OR
 
|-----|-------|
|TRUE | FALSE
|-----| -----|
FALSE | TRUE |


### Switch statement

```js

const day = 'sunday';

switch(day) {
    case 'monday' :
        console.log("plan the week");
        console.log("go to meeting");
        break;
    case 'tuesday' :
        console.log("prepare theory");
        break;
    case 'wednesday' :
        console.log("lunch with friend");
        break;
    default :
        console.log("not a working day but do self study")
}

```

### Statement and expression

- expression : produce a value in JS
  - e.g: 3+4, 1991, true && false %% !false
- statement : sequence of actions, doens't produce a value
  - e.g: if/else statement
  - string inside the if/else statement that written on String itself is an expression
- JS 
  
### Condition (ternary) operator

```js
const age = 23;

age >= 18 ? console.log('allowed to drink') : console.log('only water');

const drink =  age >= 18 ? 'wine' : 'water' ;
console.log(drink);


console.log(`i dont like to drink ${age >= 18 ? 'wine' : 'water' }`);

```

```js
// Coding Challenge #4
let bill_value = 275;
let tip = bill_value >= 50 && bill_value <= 300 ? bill_value * 0.15 : bill_value * 0.2; 

let final_value = bill_value + tip;


console.log(`The bill value was ${bill_value}, tip was ${tip}, and the total value ${final_value}`)
```

## Chapter - 2

### Strict mode
- activate the scrict mode for whole file
- helps to avoid certain bugs
- makes visiable errors, which otherwise JS would silently fail

```js
// at the top of the file, the very 1st line

'use strict';

let hasDriversLicense = false;
let passTest = true;

if(passTest) hasDriverLicense = true;
if(hasDriversLicense) console.log('allowed');

// output wont be printed since strict mode catched the error
script.js:6 Uncaught ReferenceError: hasDriverLicense is not defined

const interface = 'audio';


//outputs
Uncaught SyntaxError: Unexpected strict mode reserved word (at script.js:10:7)

```
  
### Functions
- Function : a piece of code that can be used multiple times
  - functions also receive and return the data
- function are perfect for dry code(non-repeatable code)
- console.log(): also a function, but built-in

```js
// function body
function logger() {
    
}
// sample
function logger() {
    console.log('nmg');
}
// calling, running or invoking the function
logger();
logger();

//output
nmg
nmg

function fruitProcessor(apples, oranges) {
//  console.log(apples, oranges);
    const juice = `Juice with ${apples} apples and ${oranges} oranges.`;
    return juice;
}

// the results of the fruitProcessor function value will be the
// the juice value 
const appleJuice = fruitProcessor(5, 0);
console.log(appleJuice);


const appleOrangesJuice = fruitProcessor(2, 4);
console.log(appleOrangesJuice) 

const testJuice = fruitProcessor(1.1);
console.log(testJuice);
```

### function declaration vs expressions
- expressions produces the _value_
- functios are just value and will be stored in variables
- hoisting :
  - in function declaration the it is still can be access even it is declared after the function is defined later.
  - in function expression, function can not be access before ts initialization.

```js
// declaration
function caclAge1(birthYear) {
    return 2037 - birthYear;
}

const age1 = caclAge1(1991);
console.log(age1)

// expression

const caclAge2 =  function(birthYear) {
    return 2037 - birthYear;
}
// function(birthYear) { return 2037 - birthYear; => is an expression
// and it producess the value
// and we assigned its value to calAge2 variable

const age2 = caclAge2(1991);
console.log(age1, age2)

```

### Arrow Function
- a special form of function expression, but it is a lot simplier on structure
- arrow function does not require a {} braces and it returns the value implicitly( without explicitly declaring the return value)

```js
// special form of expression
const caclAge2 =  function(birthYear) {
    return 2037 - birthYear;
}

// arrow function =>
const calcAge3 = birthYear => 2037 - birthYear;
const age3 = calcAge3(1991);
console.log(age3);
```
- in some complicated expressions, when we have multiple parameters both arrow (=>) and curly braces {} will be used, to wrap up the variables

```js
const caclAge2 =  function(birthYear) {
    return 2037 - birthYear;
}

// arrow function =>
const calcAge3 = birthYear => 2037 - birthYear;
const age3 = calcAge3(1991);
console.log(age3);

// usage of both => and {} together
const yearUntilRetirement = birthYear => {
    const age = 2037 - birthYear;
    const retirement = 65 - age;
    return retirement;
```
```js
const caclAge2 =  function(birthYear) {
    return 2037 - birthYear;
}

// arrow function =>
const calcAge3 = birthYear => 2037 - birthYear;
const age3 = calcAge3(1991);
console.log(age3);

const yearsUntilRetirement = (birthYear, firstName) => {
    const age = 2037 - birthYear;
    const retirement = 65 - age;
    // return retirement;
    return `${firstName} retires in ${retirement} years`;
}

console.log(yearsUntilRetirement(1991, 'Mike'));
console.log(yearsUntilRetirement(1993, 'Mikey'));
```
- arrow function is good, but not convenient for big scale projects, for one simple reason, bcuz they don't use 'this' keyword.

### Function calling other functions

```js
// function calling other function

function cutFruitPiecec(fruit) {
    return fruit * 4;
}

function fruitProcessor(apples, oranges) {
    const applesPieces = cutFruitPiecec(apples);
    const orangesPieces = cutFruitPiecec(oranges);
     
    const juice = `Juice with ${applesPieces} pieces out of ${apples} apples and ${orangesPieces} pieces out of ${oranges} oranges.`;
    return juice;
    }

    console.log(fruitProcessor(2, 3));
```
- REVIEW 
  - work flow :
    - **receive** input data, **transform** data, **output** data
    
  - struncture :
    - after _function_ keyword comes _function name_
    - inside the parenthesis comes _parameters_, a placeholder to receive _input_ value(like local variables of a function)
    - after parenthesis of parameters inside the curly braces{} comes _function body_, a block of code that we want to reuse. _Processes_ the function's input data
    - function body concludes with _return_ statement, that _outputs_ a value from the function and terminate execution.
    - at the outside of the function we _invoke_ the function by storing it in the variable and calling the function using parenthesis();
    - in case of function holds parameters we, call the function with _arguments_, which are the actual values of function parameters, to input data
    - once the all expression is done, basically it will be replace with the return value that we will save in the variable to output it from the function.
  
  -  three different way to write functions :
    - function declaration : function that can be used before it's declared
    - function expressions : essentially a function value stored in a variable
    - arrow function : great quick one-line function without using return. has no 'this' keyword
  
```js
// REVIEW of FUNCTIONS

const caclAge = function (birthYear) {
    return 2037 - birthYear;
}

const yearsUntilRetirement = function ( birthYear, firstName) {
    const age = caclAge(birthYear);
    const retirement = 65 - age;

    // return statement executed immediately therefore console.log that comes right after is ignored 
    // one solution to this is to lineup the console.log, to put it before return statement.
    if (retirement > 0) {
        return retirement;
        console.log(`${firstName} retires in ${retirement} years`);
    } 
    else {
            return -1;
            console.log(`${firstName} has already retired`);
    }
}

console.log(yearsUntilRetirement(1991, 'Mike Jr.'));
console.log(yearsUntilRetirement(1950, 'Mike Sr.'));
```
- coding challenge
```js
// function: coding challenge
//////////////////////////////////////
// Coding Challenge #1

/*
Back to the two gymnastics teams, the Dolphins and the Koalas! There is a new gymnastics discipline, which works differently.
Each team competes 3 times, and then the average of the 3 scores is calculated (so one average score per team).
A team ONLY wins if it has at least DOUBLE the average score of the other team. Otherwise, no team wins!

1. Create an arrow function 'calcAverage' to calculate the average of 3 scores
2. Use the function to calculate the average for both teams
3. Create a function 'checkWinner' that takes the average score of each team as parameters ('avgDolhins' and 'avgKoalas'), and then logs the winner to the console, together with the victory points, according to the rule above. Example: "Koalas win (30 vs. 13)".
4. Use the 'checkWinner' function to determine the winner for both DATA 1 and DATA 2.
5. Ignore draws this time.

TEST DATA 1: Dolphins score 44, 23 and 71. Koalas score 65, 54 and 49
TEST DATA 2: Dolphins score 85, 54 and 41. Koalas score 23, 34 and 27

HINT: To calculate average of 3 values, add them all together and divide by 3
HINT: To check if number A is at least double number B, check for A >= 2 * B. Apply this to the team's average scores 😉

*/

// generic function to calculate any three numbers
const calcAverage = (a,b,c) => (a + b + c) / 3;
console.log(calcAverage(3,4,5));

//test 1
let scoreDolphine = calcAverage(44,23,71);
let scoreKoalas = calcAverage(65,54,49);

console.log("Score is : " + scoreDolphine + ":" + scoreKoalas);

const checkWinner = function(avgDolphins, avgKoalas) {
    if(avgDolphins >= 2 * avgKoalas) {
        console.log(`Dolphins win (${avgDolphins} vs ${avgKoalas})`);
    } else if(avgKoalas >= 2 * avgDolphins) {
        console.log(`Koalas win (${avgKoalas} vs ${avgDolphins})`);
    } else {
        console.log('No team wins!');
    }
}

checkWinner(scoreDolphine,scoreKoalas);
//check random scores becuz these functions are independent from each other.
checkWinner(576, 111)

//test 2 => override the values for test 2
 scoreDolphine = calcAverage(85, 54, 41);
 scoreKoalas = calcAverage(23, 34, 27);
 
```

  
  
  





