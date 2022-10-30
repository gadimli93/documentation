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

## Values and Variables
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
## Type Conversion and Coercion

- Type conversion : when manually converted from to another type (explisit)
- Type coercion : when JS automatically convert type to another (implisit)

```js
const inputYear = '1991';
    console.log(Number(inputYear), inputYear);
    console.log(Number(inputYear) + 18);   
```
- not every dat type can be converted to another

```js
// NaN - not a Number, which has Number data type and which is invalid
console.log(Number('john'));
   console.log(typeof NaN);
```










