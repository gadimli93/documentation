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
```
  
  
  





