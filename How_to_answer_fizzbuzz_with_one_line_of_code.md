# An interesting way to answer FizzBuzz with one line of code
*or... how to excessively use array methods/ternary operators and alientate co-workers*

## WARNING: you probably shouldnt use this in an interview, but you might learn something cool (?)

> Question: Create a function that will console log numbers from 1 to N. Where a number is a multiple of 3 replace the
> number with the word "Fizz"; Where the number is a multiple of 5 replace it with the word "Buzz" and if the number is both a multiple of 3 AND 5 replace it with the word "FizzBuzz".

This is a classic that most people should know, the answer they are expecting should demonstrate a knowledge of basic flow control, loops and also some knowledge of the modulo operator (%) and should look something like the example below:

```javascript

function fizzBuzz(n){
  for(var i = 1; i < n; i++){
    if (i % 3 === 0 && i % 5 === 0) {
      console.log('FizzBuzz')
    } else if (i % 5 === 0) {
      console.log('Buzz')
    } else if (i % 3 === 0) {
      console.log('Fizz')
    } else {
      console.log(i)
    }
  }
}
```
### YAWWWNNN....

its boring, outdated, too many if statements and theres no delightful es6 syntax that we all love so much!

### its time for something new!

In our example though, were going to use some es6 syntax (Array methods, spread operator and ternary operators) and also some clever manipulation of 'truthy' values and ternary operators to replace the simple `if/else` yawn fest.

so lets have a look at the finished example first and then ill explain it step by step:

```javascript
const fizzBuzz = n => [...Array(n).keys()]
                                .map(i => ((i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz')|| i + 1)
                                .forEach(i => console.log(i));
```

muuuch better, as much es6 as would can want! , no age old `if/else`, super cool indenting of chained methods for big-boy points! and also some fairly clever manipulation of truthy values which is no doubt going to impress your future employer as well as the ladies (disclaimer: this probably wont impress anyone...)

### so how does this work?

This example can be broken down into three steps:
1. create an initial array with values 0-N at each index
2. map over the array and return values into a new array are either either 1) changed to the correct string or 2) return a normal number value
3. print out the new array to the console

___

### Step one: Create an array with values 0-N

first things first we declare our function using an arrow function, cause lets be honest !!!`arrow function`!!! sounds and looks `()=>{}` a lot cooler than a standard `function(){}` declaration *...boring*

```javascript
  const fizzBuzz = (n) => 
```

notice the use of `const` cause were were out to impress, no `let` and definately no `var` *vomits* ! we also dont use braces `{}` because were doing this on one line, and with out braces we can do this on one line and this makes the function implicetly return so we dont even need to use the `return` key word! super impressive 10/10!

ok first part, let make that array.

```javascript

  [...new Array(n).keys()]

```

here we use the `Array` constructor function, this will either take a multiple values as parameters and create an array out of those values eg:

```javascript
  new Array(1,2,3)
  
  // returns [1,2,3]

  /* note: you dont really need the new keyword here, it actually works 
           the same without but leaving it in makes your look like you know what youre doing!
  */

```

or if given a single numeric argument will return an array with N empty positions e.g

```javascript
  new Array(5)
  
  //returns [undefined,undefined,undefined,undefined,undefined]
```

brilliant! but an empty array is useless to us at the moment...


That is where the `Array.keys()` method comes in.

so... much like object literals where we use key value pairs e.g `{foo : "bar"}`, arrays also use key values pairs, but the keys are pre-defined to the items index in that array eg ` {0 : "bar", 1 : "baz"} ` and are always ordered `[0, ... , n.length - 1]` unlike an object literal where we have to define the key and theyre not ordered.

we can get the keys using the `Array.keys()` method which returns a new `array iterator` object containing the index values for each position in the array (which we set to n long).

```javascript
console.log(new Array(1).keys()) 

//returns 'new Array Iterator {}'
// wtf is this!?
```

but an iterator object isnt much use, we want the values it holds! to easily get the values from the iterator into an actual array we can use, we can use the es6 spread operator `[...foo]` to 'spread' out the values provided by the iterator to a new array 

```javascript
  [...new Array(5).keys()]
  
  // returns [0,1,2,3,4]
  
  // noice!
```

Step 1 complete! pat yourself on the back!

___

### Step two: map over the array and use some logic to return the desired result

Ok... if you got lost in the first section you might want to take this one a bit slower...

```javascript
  .map(i => ((i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz')||i + 1)
```

right...this...i might need to break this down a bit further..

___

### Step 2.1

ok crash course in Array.prototype.map()! Go!

  `Array.prototype.map()`
  
.map is a super trendy array method which will along with .reduce and .filter will solve most of lifes problems, essentially what it does it creates a NEW array based on the returned results of calling a function (that we provide) to each item of the array

#### a simple example:
```javascript
const a = [1,2,3]

const a2 = a.map(i => i * 2)

console.log(a2)
// returns [2,4,6]
```

this map method takes in a callback function with i as a parameter (i being the value at each index of the loop), it then returns the value of i * 2 and pushes it to a new array giving us the new array with the values `[2,4,6]`

map is a really cool method and is very popular in the functional programming paradigm (which is the current flavour of the month!)

for more on map https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map

___

### Step 2.2

  so in our map function we are going to perform the below logic on each item and push the return value to a new array.

```javascript
  (i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz')
```

ok.. i might need to break this down futher...

___

### Step 2.2.1

```javascript
  (i + 1) % 3 ? '' : 'Fizz')
```

so this is a ternary operator, think of them as if/else statements for cool kids. 

They work like this:

```javascript
  expression ? ifTrue : ifFalse
```

we evaluate an expression, if that expression evaluates to true or "TRUTHY" (this is key in our example) return one thing if its false or "FALSY" return another thing. so back to our example

```javascript
  (i + 1) % 3 ? '' : 'Fizz')
```

for every item in our array (of values 0 - N), increment i by one (this is so we start from 1 instead of zero) and evaluate the expression `(i + 1) % 3`
___
### modulo crash course! go!

`%` or modulo operator returns the remainder of the first value when divided by the second value
so a brief look

```javascript
1 % 3
//returns 1
2 % 3
//returns 2
3 % 3
//returns 0
4 % 3
//returns 1
```
hopefully you've seen the pattern (if not you can try a few more values in your console)

if we `%` a number by 3 that is fully divisible by 3 we get a return value of `0`

hopefully now my stress on truthy and falsey values is hitting home? (if not I will explain it any way...)

lets reduce our expression down to a static values and walk it through

```javascript
  i = 0
  (i + 1) % 3 ? '' : 'Fizz'
  //becomes
  1 % 3 ? '' : 'Fizz'
  //which becomes
  1 ? '' : 'Fizz
```
number values on there own in an expression will be evaluated by their truthyness or falseyness

___
### truthy/false: number edition:
long story short: 0 equates to `false` and EVERY OTHER NUMBER IN EXISTANCE equates to `true`
___

so anyway our expression now becomes:

```javascript
  true ? '' : 'Fizz'
```

which is super simple to understand, if `true` (eg, the number is NOT a multiple of 3) return a empty string `''`, else return the word `'Fizz'`

brilliant! now that we understand that part explaining the other half is a doddle

```javascript

((i + 1) % 5 ? '' : 'Buzz')

```

its exactly the same but were now checking if the number is divisble by 5 and checking the truthyness/falseyness of the remainder (`0` === `false`, all other numbers === `true`), if true return `''` else return `Buzz`


### Step 2.2.2

so lets step back a bit and look at the expression again... we will ignore everthing right of the `||` for now

```javascript
((i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz')
```

if we take `i = 0` and reduce the expression down

```javascript
((i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz')

((1 % 3 ? '' : 'Fizz') + (1 % 5 ? '' : 'Buzz')

(1 ? '' : 'Fizz') + (1 ? '' : 'Buzz')

(true ? '' : 'Fizz') + (true ? '' : 'Buzz')

('') + ('')

```

now I dont feel I need to explain in great depth, that an empty string `''` plus and empty string `''` equates to an yet another empty string `''` (I told you, you would learn something cool ¬_¬)

but if either side of the expression reduces down to a falsey value (e.g. a number is divisible by 3, 5 or both 3 and 5)
we get some different results. lets have a look at some below

```javascript
i = 3

//reduces down to...
(0 ? '' : 'Fizz') + (3 ? '' : 'Buzz')

// which reduces down to...
(false ? '' : 'Fizz') + (true ? '' : 'Buzz')

// which reduces down to...
('Fizz') + ('')

//which reduces finally down to

'Fizz'

```

```javascript
i = 5

//reduces down to...
(3 ? '' : 'Fizz') + (0 ? '' : 'Buzz')

// which reduces down to...
(true ? '' : 'Fizz') + (false ? '' : 'Buzz')

// which reduces down to...
('') + ('Buzz')

//and we get

'Buzz'
```

```javascript
i = 15

//reduces down to...
(0 ? '' : 'Fizz') + (0 ? '' : 'Buzz')

// which reduces down to...
(false ? '' : 'Fizz') + (false ? '' : 'Buzz')

// which reduces down to...
('Fizz') + ('Buzz')

//and if using + with two strings concatenates them together so we get

'FizzBuzz'
```

### Step 2.3

ok now back to the expression...

((i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz')||i + 1)

hopefully this now makes alot more sense now!

the last bit we need to look at is the right side of the `||` otherwise known as the `Or` operator

### logical operators && (AND) and || (OR)
  
so a brief description of these logical operators are they look at the boolean value of multiple expressions and check if either ALL are truthy when using the And operator `&&` or one of them is truthy with the Or operator `||`
  
our expression uses only 'Or' `||` so I will focus on that, as we are using it a bit differently than normal.

typically an or `||` operator can be used like this

```javascript
// typical use in 'if' statements

  let i = 4
  
  if (i === 3 || i === 4){
    console.log('i did the thing!');
  }
  
  // return 'i did the thing'
  
  /* but it can also be used to conditionally return (this is what we use it for in our expression!) */
  
  function foo (bar) {
    return bar - 1 || 'baz'
  }
  
  foo(2) // returns 1
  foo(1) // returns 'baz' (as bar - 1 === 0 which is falsey!)
  
```
lets explain breifly our OR

`||` operators evaluate the left most expression first, and checks if its truthy, if its false, will move onto the next and check if that is truthy and so on (we can chain multiple `||`). The first expression that equates to `true` will cause the entire expression to equal `true` (as its only looking for one truthy value). the expression will equate to `false` only if no value in the expression evaluates as `true`.

if used with a `return` like in the above example, it will return the first truthy result or if all are falsey will return the final expression in a chain

ok...back to our expression!

```javascript
  ((i + 1) % 3 ? '' : 'Fizz') + ((i + 1) % 5 ? '' : 'Buzz') || i + 1)
```

our expression has one `||`, our ternary operators are the left most expression so they will all be evaluated first.

weve seen from our previous examples that our expression will return either
1. `Fizz` if `(i + 1)` is fully divisible by 3 but not divisible by 5.
2. `Buzz` if `(i + 1)` is fully divisible by 5 but not divisible by 3.
3. `FizzBuzz` if `(i + 1)` is full divisble by both 3 AND 5.
4. `''` if `(i + 1)` is not fully divisble by 3 OR 5

so our expression can be reduced down to

```javascript
// divisible by 3
'Fizz' || i + 1

// divisible by 5
'Buzz' || i + 1

// divisible by 3 and 5
'FizzBuzz' || i + 1

// NOT divisible by 3 or 5
'' || i + 1
```

going back to truthy and falsey, I explained that numbers can be either truthy (every number except 0) or falsey (0).

but of course strings can also be evaluated to Booleans

```javascript
'any non empty string' === true

'' === false
```

so with our `||` statements can be reduced down to the following expressions

```javascript
// divisible by 3
true || i + 1

// divisible by 5
true || i + 1

// divisible by 3 and 5
true || i + 1

// NOT divisible by 3 or 5
false || i + 1
```

and because we used the super cool arrow function `()=>{}` in our `.map()` callback without braces the function will implictly return a value from the above.

if any of the left hand expressions equate to `true` the string value of that expression will be returned from the callback, replacing the the number in our new array created by the `.map()` method. If it equates to `false` we will return `(i + 1)`


### Step 3: final stretch! logging result to the console!

so this step should be super straight forward to explain!

```javascript
.map(/*our expression*/).forEach(i => console.log(i));
```

weve created our array of numbers, weve mapped a new array based on the values of the array and our new array now has `Fizz` instead of  multiples of 3, `Buzz` instead of multiples of 5 and `FizzBuzz` instead of multiples of 3 and 5. But, we just have an array in memory, the final step is to console.log each item so we can see the results.

after the map method we "chain" or "pipe" the result (from our new array courtesy of `.map()`) into a `.forEach()`

`.forEach()` is similar to `.map()` in that it takes a callback function which is called on each item of the array, unlike `.map()` though it doesnt create a new array instead we use this to simply `console.log` each item in the array.

and there we have it, weve answered the age old FizzBuzz interview question with one line 

again, I personally dont suggest using this example in an actual interview, while it is a little bit 'showey' the simple `if/else` with a `for` loop example is much more readable. but by creating this we've learnt some cool new ways of answering a stale question with some interesting syntax.

if you have any refinements id love to see it, (my favourite so far is not using any array methods and just using curried functions all the way down!)

im going to try and do more of these for other interview questions, if you have any suggestions send them to me on my twitter @Phl3bas
