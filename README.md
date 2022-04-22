# Crypto-Zombie Game using Solidity and Ethereum

<br/>

## 1. Making the Zombie Factory

### 1.1 Overview

In this section, we are going to build a "Zombie Factory" to build an army of zombies.

* Our factory will maintain a database of all zombies in our army
  
* Our factory will have a function for creating new zombies
  
* Each zombie will have a random and unique appearance

#### How Zombie DNA Works

The zombie's appearance will be based on its "Zombie DNA". Zombie DNA is simple — it's a 16-digit integer, like:

```
8356281049284737
```

Just like real DNA, different parts of this number will map to different traits. The first 2 digits (`83`) map to the zombie's head type, the second 2 digits (`56`) to the zombie's eyes, etc. We will keep things simple, and our zombies can have only 7 different types of heads (even though 2 digits allow 100 possible options). Later on we could add more head types if we wanted to increase the number of zombie variations. For example, the first 2 digits of our example DNA above are 83. To map that to the zombie's head type, we do 83 % 7 + 1 = 7. So this Zombie would have the 7th zombie head type.

### 1.2 Contracts

Solidity's code is encapsulated in contracts. A `contract` is the fundamental building block of Ethereum applications — all variables and functions belong to a contract, and this will be the starting point of all your projects.

An empty contract named `HelloWorld` would look like this:

```javascript

contract HelloWorld {

}
```

#### Version Pragma

All solidity source code should start with a "version pragma" — a declaration of the version of the Solidity compiler this code should use. This is to prevent issues with future compiler versions potentially introducing changes that would break your code.

We want to be able to compile our smart contracts with any compiler version in the range of 0.5.0 (inclusive) to 0.6.0 (exclusive). It looks like this: `pragma solidity >=0.5.0 <0.6.0;`.

To start creating our Zombie army, let's create a empty base contract called `ZombieFactory`

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

}
```

### 1.3 State Variables & Integers

`State variables` are permanently stored in contract storage. This means they're written to the Ethereum blockchain. Think of them like writing to a DB.

#### Unsigned Integers: `uint`

The `uint` data type is an unsigned integer, meaning its value must be non-negative. There's also an int data type for signed integers.

**Note**: In Solidity, `uint` is actually an alias for `uint256`, a 256-bit unsigned integer. We can declare uints with less bits — `uint8`, `uint16`, `uint32`, etc..

Our Zombie DNA is going to be determined by a 16-digit number:

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;

}
```

### 1.4 Math Operations

Math in Solidity is pretty straightforward. The following operations are the same as in most programming languages:

* Addition: x + y
* Subtraction: x - y,
* Multiplication: x * y
* Division: x / y
* Modulus / remainder: x % y (for example, 13 % 5 is 3, because if you divide 5 into 13, 3 is the remainder) 
* Solidity also supports an exponential operator (i.e. "x to the power of y", x^y):
 
    ```js
    uint x = 5 ** 2; // equal to 5^2 = 25
    ```

To make sure our Zombie's DNA is only 16 characters, let's make another `uint` equal to `10^16`. That way we can later use the modulus operator `%` to shorten an integer to 16 digits.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    // start here

}
```

### 1.5 Structs

Sometimes we need a more complex data type. For this, Solidity provides structs:

```js
struct Person {
  uint age;
  string name;
}
```

Structs allow you to create more complicated data types that have multiple properties.

Note that we just introduced a new type, `string`. Strings are used for arbitrary-length UTF-8 data. Ex. `string greeting = "Hello world!"`

In our app, we're going to want to create some zombies! And zombies will have multiple properties, so this is a perfect use case for a `struct`.

1. Create a `struct` named `Zombie`.

2. Our Zombie struct will have 2 properties: `name` (a `string`), and `dna` (a `uint`).

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }
```

### 1.6 Arrays

When you want a collection of something, we can use an `array`. There are two types of arrays in Solidity: `fixed` arrays and `dynamic` arrays:

```js
// Array with a fixed length of 2 elements:
uint[2] fixedArray;
// another fixed Array, can contain 5 strings:
string[5] stringArray;
// a dynamic Array - has no fixed size, can keep growing:
uint[] dynamicArray;
```

We can also create an array of `structs`. Using the previous section's `Person` struct:

```js
Person[] people; // dynamic Array, we can keep adding to it
```

Remember that state variables are stored permanently in the blockchain? So creating a dynamic array of structs like this can be useful for storing structured data in our contract, kind of like a database.

#### Public Arrays

We can declare an `array` as `public`, and Solidity will automatically create a `getter` method for it. The syntax looks like:

```js
Person[] public people;
```

Other contracts would then be able to read from, but not write to, this array. So this is a useful pattern for storing public data in your contract.

We're going to want to store an army of zombies in our app. And we're going to want to show off all our zombies to other apps, so we'll want it to be public.

1. Create a public array of `Zombie` `structs`, and name it `zombies`.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies

}
```

### 1.7 Function Declarations

A function declaration in solidity looks like the following:

```js
function eatHamburgers(string memory _name, uint _amount) public {

}
```
This is a function named `eatHamburgers` that takes 2 parameters: a `string` and a `uint`. For now the body of the function is empty. Note that we're specifying the function visibility as `public`. We're also providing instructions about where the `_name` variable should be stored- in `memory`. This is required for all reference types such as arrays, structs, mappings, and strings.

What is a reference type we ask?

Well, there are two ways in which we can pass an argument to a Solidity function:

* By value, which means that the Solidity compiler creates a new copy of the parameter's value and passes it to our function. This allows our function to modify the value without worrying that the value of the initial parameter gets changed.

* By reference, which means that your function is called with a reference to the original variable. Thus, if your function changes the value of the variable it receives, the value of the original variable gets changed.

**Note**: It's convention (but not required) to start function parameter variable names with an underscore (_) in order to differentiate them from global variables. We'll use that convention throughout our project.

We would call this function like so:

```js
eatHamburgers("vitalik", 100);
```

In our app, we're going to need to be able to create some zombies. Let's create a function for that.

1. Create a `public` function named `createZombie`. It should take two parameters: _name (a `string`), and _dna (a `uint`). We will pass the first argument by value by using the `memory` keyword

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function createZombie (string memory _name, uint _dna) public {
        // start here
    }

}
```

### 1.8 Working With Structs and Arrays

#### Creating New Structs

Remember our `Person` struct in the previous example?

```js
struct Person {
  uint age;
  string name;
}

Person[] public people;
```

Now we're going to learn how to create new `Person`s and add them to our `people` array.

```js
// create a New Person:
Person satoshi = Person(172, "Satoshi");

// Add that person to the Array:
people.push(satoshi);

```

We can also combine these together and do them in one line of code to keep things clean:

```js
people.push(Person(16, "Vitalik"));

```
Note that `array.push()` adds something to the end of the array, so the elements are in the order we added them. See the following example:

```js
uint[] numbers;
numbers.push(5);
numbers.push(10);
numbers.push(15);
// numbers is now equal to [5, 10, 15]
```

Let's make our createZombie function do something!

1. Fill in the function body so it creates a new `Zombie`, and adds it to the `zombies` array. The `name` and `dna` for the new Zombie should come from the function arguments.
2. Let's do it in one line of code to keep things clean.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function createZombie(string memory _name, uint _dna) public {
        zombies.push(Zombie(_name, _dna));
    }

}
```

### 1.9 Private / Public Functions

In Solidity, functions are `public` by default. This means anyone (or any other contract) can call your contract's function and execute its code.

Obviously this isn't always desirable, and can make your contract vulnerable to attacks. Thus it's good practice to mark your functions as `private` by default, and then only make `public` the functions we want to expose to the world.

Let's look at how to declare a private function:

```js
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

This means only other functions within our contract will be able to call this function and add to the `numbers` array.

As we can see, we use the keyword `private` after the function name. And as with function parameters, it's convention to start private function names with an underscore (_).

Our contract's `createZombie` function is currently public by default — this means anyone could call it and create a new Zombie in our contract! Let's make it private.

1. Modify `createZombie` so it's a `private` function. Don't forget the naming convention!

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string memory _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

}

```

### 1.10 More on Functions

In this section, we're going to learn about function `return values`, and function modifiers.

#### Return Values

To return a value from a function, the declaration looks like this:

```js
string greeting = "What's up dog";

function sayHello() public returns (string memory) {
  return greeting;
}
```

In Solidity, the function declaration contains the type of the return value (in this case `string`).

#### Function modifiers

The above function doesn't actually change state in Solidity — e.g. it doesn't change any values or write anything.

So in this case we could declare it as a `view` function, meaning it's only viewing the data but not modifying it:

```js
function sayHello() public view returns (string memory) {
}
```    

Solidity also contains `pure` functions, which means we are not even accessing any data in the app. Consider the following:

```js
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}

```

This function doesn't even read from the state of the app — its return value depends only on its function parameters. So in this case we would declare the function as `pure`.
`
**Note**: It may be hard to remember when to mark functions as pure/view. Luckily the Solidity compiler is good about issuing warnings to let you know when you should use one of these modifiers.

We're going to want a helper function that generates a random DNA number from a string.

1. Create a private function called `generateRandomDna`. It will take one parameter named _`str` (a `string`), and return a `uint`. Don't forget to set the data location of the `str` parameter to `memory`.

2. This function will view some of our contract's variables but not modify them, so mark it as `view`.

3. The function body should be empty at this point — we'll fill it in later.

```js
pragma solidity ^0.4.25;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string memory _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        // will include later
    }

}
```

### 1.11 Keccak256 and Typecasting

We want our `_generateRandomDna` function to return a (semi) random `uint`. How can we accomplish this?

Ethereum has the hash function `keccak256` built in, which is a version of SHA3. A hash function basically maps an input into a random 256-bit hexadecimal number. A slight change in the input will cause a large change in the hash.

It's useful for many purposes in Ethereum, but for right now we're just going to use it for pseudo-random number generation.

Also important, `keccak256` expects a single parameter of type `bytes`. This means that we have to "pack" any parameters before calling `keccak256`:

Example:

```js
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256(abi.encodePacked("aaaab"));
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256(abi.encodePacked("aaaac"));
```

As you can see, the returned values are totally different despite only a 1 character change in the input.

**Note**: Secure random-number generation in blockchain is a very difficult problem. Our method here is insecure, but since security isn't top priority for our Zombie DNA, it will be good enough for our purposes.

#### Typecasting

Sometimes we need to convert between data types. Take the following example:

```js
uint8 a = 5;
uint b = 6;
// throws an error because a * b returns a uint, not uint8:
uint8 c = a * b;
// we have to typecast b as a uint8 to make it work:
uint8 c = a * uint8(b);
```

In the above, `a * b` returns a `uint`, but we were trying to store it as a `uint8`, which could cause potential problems. By casting it as a `uint8`, it works and the compiler won't throw an error.

Let's fill in the body of our `_generateRandomDna` function! Here's what it should do:

1. The first line of code should take the `keccak256` hash of `abi.encodePacked(_str)` to generate a pseudo-random hexadecimal, typecast it as a `uint`, and finally store the result in a `uint` called `rand`.

2. We want our DNA to only be 16 digits long (remember our `dnaModulus`?). So the second line of code should return the above value modulus (`%`) `dnaModulus`.

```js
pragma solidity  >=0.5.0 <0.6.0;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string memory _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    } 

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }
}
```

### 1.12 Putting It Together

We're close to being done with our random Zombie generator! Let's create a public function that ties everything together.

We're going to create a public function that takes an input, the zombie's name, and uses the name to create a zombie with random DNA.

1. Create a `public` function named `createRandomZombie`. It will take one parameter named `_name` (a `string` with the data location set to `memory`). (Note: Declare this function `public` just as you declared previous functions `private`)

2. The first line of the function should run the `_generateRandomDna` function on `_name`, and store it in a `uint` named `randDna`.

3. The second line should run the `createZombie` function and pass it `_name`and `randDna`.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    // will declare our event here

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string memory _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
        // will fire it here
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### 1.13 Events

Now let's add an `event`.

`Events` are a way for your contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and take action when they happen.

Example:

```js
// declare the event
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public returns (uint) {
  uint result = _x + _y;
  // fire an event to let the app know the function was called:
  emit IntegersAdded(_x, _y, result);
  return result;
}
```

Our app front-end could then listen for the `event`. A javascript implementation would look something like:

```js
OurContract.IntegersAdded(function(error, result) {
  // do something with result
})
```

We want an `event` to let our front-end know every time a new zombie was created, so the app can display it.

1. Declare an `event` called `NewZombie`. It should pass `zombieId` (a `uint`), `name` (a `string`), and `dna` (a `uint`).

2. Modify the `_createZombie` function to fire the `NewZombie` `event` after adding the new Zombie to our `zombies` array.

3. We are going to need the zombie's `id. array.push()` returns a `uint` of the new length of the array - and since the first item in an array has index 0, `array.push() - 1` will be the index of the zombie we just added. Store the result of `zombies.push() - 1` in a `uint` called `id`, so you can use this in the `NewZombie` event in the next line.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string memory _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        emit NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

Our ***Solidity contract*** is complete! Now we need to write a javascript frontend that interacts with the contract.

Ethereum has a Javascript library called `Web3.js`.

```js
// Here's how we would access our contract:
var abi = /* abi generated by the compiler */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory` has access to our contract's public functions and events

// some sort of event listener to take the text input:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  // Call our contract's `createRandomZombie` function:
  ZombieFactory.createRandomZombie(name)
})

// Listen for the `NewZombie` event, and update the UI
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  generateZombie(result.zombieId, result.name, result.dna)
})

// take the Zombie dna, and update our image
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // pad DNA with leading zeroes if it's less than 16 characters
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // first 2 digits make up the head. We have 7 possible heads, so % 7
    // to get a number 0 - 6, then add 1 to make it 1 - 7. Then we have 7
    // image files named "head1.png" through "head7.png" we load based on
    // this number:
    headChoice: dnaStr.substring(0, 2) % 7 + 1,
    // 2nd 2 digits make up the eyes, 11 variations:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 6 variations of shirts:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    // last 6 digits control color. Updated using CSS filter: hue-rotate
    // which has 360 degrees:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
```

What our javascript then does is take the values generated in `zombieDetails` above, and use some browser-based javascript magic (we're using `Vue.js`) to swap out the images and apply CSS filters.

## 2. Zombies Attack Their Victims

### 2.1 Overview

In Section 1, we created a function that takes a name, uses it to generate a random zombie, and adds that zombie to our app's zombie database on the blockchain.

In this section, we're going to make our app more game-like: We're going to make it multi-player, and we'll also be adding a more fun way to create zombies instead of just generating them randomly.

How will we create new zombies? By having our zombies "feed" on other lifeforms!

#### Zombie Feeding

When a zombie feeds, it infects the host with a virus. The virus then turns the host into a new zombie that joins your army. The new zombie's DNA will be calculated from the previous zombie's DNA and the host's DNA.

### 2.2 Mappings and Addresses

Let's make our game multi-player by giving the zombies in our database an owner.

To do this, we'll need 2 new data types: `mapping` and `address`.

#### Addresses

The Ethereum blockchain is made up of `accounts`, which we can think of like bank accounts. An account has a balance of `Ether` (the currency used on the Ethereum blockchain), and you can send and receive Ether payments to other accounts, just like our bank account can wire transfer money to other bank accounts.

Each account has an address, which you can think of like a bank account number. It's a unique identifier that points to that account, and it looks like this:

```
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```

So, an address is owned by a specific user (or a smart contract).

So we can use it as a unique ID for ownership of our zombies. When a user creates new zombies by interacting with our app, we'll set ownership of those zombies to the Ethereum address that called the function.

#### Mappings

In Section 1 we looked at `structs` and `arrays`. `Mappings` are another way of storing organized data in Solidity.

Defining a `mapping` looks like this:

```js
// For a financial app, storing a uint that holds the user's account balance:
mapping (address => uint) public accountBalance;
// Or could be used to store / lookup usernames based on userId
mapping (uint => string) userIdToName;
```

A mapping is essentially a key-value store for storing and looking up data. In the first example, the key is an `address` and the value is a `uint`, and in the second example the key is a `uint` and the value a `string`.

To store zombie ownership, we're going to use two mappings: one that keeps track of the address that owns a zombie, and another that keeps track of how many zombies an owner has.

1. Create a mapping called `zombieToOwner`. The key will be a `uint` (we'll store and look up the zombie based on its id) and the value an `address`. Let's make this mapping `public`.

Create a mapping called `ownerZombieCount`, where the key is an `address` and the value a `uint`.

```js

pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string memory _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        emit NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### 2.3 Msg.sender

Now that we have our mappings to keep track of who owns a zombie, we'll want to update the `_createZombie` method to use them.

In order to do this, we need to use something called `msg.sender`.

#### msg.sender

In Solidity, there are certain global variables that are available to all functions. One of these is `msg.sender`, which refers to the `address` of the person (or smart contract) who called the current function.

***Note***: In Solidity, function execution always needs to start with an external caller. A contract will just sit on the blockchain doing nothing until someone calls one of its functions. So there will always be a `msg.sender`.

Here's an example of using `msg.sender` and updating a `mapping`:

```js
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // Update our `favoriteNumber` mapping to store `_myNumber` under `msg.sender`
  favoriteNumber[msg.sender] = _myNumber;
  // ^ The syntax for storing data in a mapping is just like with arrays
}

function whatIsMyNumber() public view returns (uint) {
  // Retrieve the value stored in the sender's address
  // Will be `0` if the sender hasn't called `setMyNumber` yet
  return favoriteNumber[msg.sender];
}
```

In this trivial example, anyone could call `setMyNumber` and store a `uint` in our contract, which would be tied to their `address`. Then when they called `whatIsMyNumber`, they would be returned the `uint` that they stored.

Using `msg.sender` gives you the security of the Ethereum blockchain — the only way someone can modify someone else's data would be to steal the private key associated with their Ethereum address.

Let's update our `_createZombie` method from Section 1 to assign ownership of the zombie to whoever called the function.

1. First, after we get back the new zombie's `id`, let's update our `zombieToOwner` mapping to store `msg.sender` under that `id`.

2. Second, let's increase `ownerZombieCount` for this `msg.sender`.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string memory _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        emit NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### 2.4 Require 

In Section 1, we made it so users can create new zombies by calling `createRandomZombie` and entering a `name`. However, if users could keep calling this function to create unlimited zombies in their army, the game wouldn't be very fun.

Let's make it so each player can only call this function once. That way new players will call it when they first start the game in order to create the initial zombie in their army.

How can we make it so this function can only be called once per player?

For that we use `require`. `require` makes it so that the function will throw an error and stop executing if some condition is not true:

```js
function sayHiToVitalik(string memory _name) public returns (string memory) {
  // Compares if _name equals "Vitalik". Throws an error and exits if not true.
  // (Side note: Solidity doesn't have native string comparison, so we
  // compare their keccak256 hashes to see if the strings are equal)
  require(keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Vitalik")));
  // If it's true, proceed with the function:
  return "Hi!";
}
```

If we call this function with `sayHiToVitalik("Vitalik")`, it will return "Hi!". If we call it with any other input, it will throw an error and not execute.

Thus `require` is quite useful for verifying certain conditions that must be true before running a function.

In our zombie game, we don't want the user to be able to create unlimited zombies in their army by repeatedly calling `createRandomZombie` — it would make the game not very fun.

Let's use `require` to make sure this function only gets executed one time per user, when they create their first zombie.

Put a `require` statement at the beginning of `createRandomZombie`. The function should check to make sure `ownerZombieCount[msg.sender]` is equal to `0`, and throw an error otherwise.

```js
pragma solidity >=0.5.0 <0.6.0;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string memory _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        emit NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### 2.5 Inheritance

Sometimes it makes sense to split our code logic across multiple contracts to organize the code.

One feature of Solidity that makes this more manageable is contract `inheritance`:

```js
contract Doge {
  function catchphrase() public returns (string memory) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string memory) {
    return "Such Moon BabyDoge";
  }
}
```

`BabyDoge` `inherits` from `Doge`. That means if you compile and deploy `BabyDoge`, it will have access to both `catchphrase()` and `anotherCatchphrase()` (and any other public functions we may define on `Doge`).

This can be used for logical inheritance (such as with a subclass, a `Cat` is an `Animal`). But it can also be used simply for organizing your code by grouping similar logic together into different contracts.

We're going to be implementing the functionality for our zombies to feed and multiply. Let's put this logic into its own contract that inherits all the methods from `ZombieFactory`.

Make a contract called `ZombieFeeding` below `ZombieFactory`. This contract should inherit from our `ZombieFactory` contract.

```js
pragma solidity >=0.5.0 <0.6.0;

// put import statement here

contract ZombieFeeding is ZombieFactory {

}
```

### 2.6 Import 

When we have multiple files and you want to import one file into another, Solidity uses the `import` keyword:

```js
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

So if we had a file named `someothercontract.sol` in the same directory as this contract (that's what the `./` means), it would get imported by the compiler.

Now that we've set up a multi-file structure, we need to use `import` to read the contents of the other file:

1. Import `zombiefactory.sol` into our new file, `zombiefeeding.sol`.

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract ZombieFeeding is ZombieFactory {
}
```

### 2.7 Storage vs Memory (Data location)

In Solidity, there are two locations you can store variables — in `storage` and in `memory`.

`Storage` refers to variables stored permanently on the blockchain. `Memory` variables are temporary, and are erased between external function calls to your contract. Think of it like your computer's hard disk vs RAM.

Most of the time you don't need to use these keywords because Solidity handles them by default. State variables (variables declared outside of functions) are by default `storage` and written permanently to the blockchain, while variables declared inside functions are `memory` and will disappear when the function call ends.

However, there are times when you do need to use these keywords, namely when dealing with `structs` and `arrays` within functions:

```js
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ Seems pretty straightforward, but solidity will give you a warning
    // telling you that you should explicitly declare `storage` or `memory` here.

    // So instead, you should declare with the `storage` keyword, like:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...in which case `mySandwich` is a pointer to `sandwiches[_index]`
    // in storage, and...
    mySandwich.status = "Eaten!";
    // ...this will permanently change `sandwiches[_index]` on the blockchain.

    // If you just want a copy, you can use `memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...in which case `anotherSandwich` will simply be a copy of the 
    // data in memory, and...
    anotherSandwich.status = "Eaten!";
    // ...will just modify the temporary variable and have no effect 
    // on `sandwiches[_index + 1]`. But you can do this:
    sandwiches[_index + 1] = anotherSandwich;
    // ...if you want to copy the changes back into blockchain storage.
  }
}
```

Solidity compiler will also give you warnings to let you know when you should be using one of these keywords.

It's time to give our zombies the ability to feed and multiply!

When a zombie feeds on some other lifeform, its DNA will combine with the other lifeform's DNA to create a new zombie.

1. Create a function called `feedAndMultiply`. It will take two parameters: `zombieId` (a `uint`) and `targetDna` (also a `uint`). This function should be `public`.

2. We don't want to let someone else feed our zombie! So first, let's make sure we own this zombie. Add a `require` statement to verify that `msg.sender` is equal to this zombie's owner (similar to how we did in the `createRandomZombie` function).

**Note**: Although here we expect `msg.sender` to come first, but, normally when we're coding, we can use whichever order we prefer — both are correct.

3. We're going to need to get this zombie's DNA. So the next thing our function should do is declare a local `Zombie` named `myZombie` (which will be a `storage` pointer). Set this variable to be equal to index `_zombieId` in our `zombies` array.

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
  }

}
```

### 2.8 Zombie DNA

The formula for calculating a new zombie's DNA is : the average between the feeding zombie's DNA and the target's DNA.

1. First we need to make sure that `targetDna` isn't longer than 16 digits. To do this, we can set `_targetDna` equal to `targetDna % dnaModulus` to only take the last 16 digits.

2. Next our function should declare a `uint` named `newDna`, and set it equal to the average of `myZombie`'s DNA and `_targetDna` (as in the example above).

**Note**: We can access the properties of `myZombie` using `myZombie.name` and `myZombie.dna`

3. Once we have the new DNA, let's call `_createZombie`. Note that it requires a name, so let's set our new zombie's name to `"NoName"` for now — we can write a function to change zombies' names later.

**Note**: For you Solidity whizzes, we may notice a problem with our code here! Don't worry, we'll fix this later.

```js
pragma solidity >=0.5.0 <0.6.0;

import "./zombiefactory.sol";

contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```

### 2.9 More on Function Visibility

The code in our previous sub section has a mistake!

If we try compiling it, the compiler will throw an error.

The issue is we tried calling the `_createZombie` function from within `ZombieFeeding`, but `_createZombie` is a `private` function inside `ZombieFactory`. This means none of the contracts that inherit from `ZombieFactory` can access it.

#### Internal and External

In addition to `public` and `private`, Solidity has two more types of visibility for functions: `internal` and `external`.

`internal` is the same as `private`, except that it's also accessible to contracts that inherit from this contract. (Hey, that sounds like what we want here!).

`external` is similar to `public`, except that these functions can ONLY be called outside the contract — they can't be called by other functions inside that contract. We'll see why we might want to use `external` vs `public` later.

For declaring `internal` or `external` functions, the syntax is the same as `private` and `public`:

```js
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string memory) {
    baconSandwichesEaten++;
    // We can call this here because it's internal
    eat();
  }
}
```

1. Change `_createZombie()` from `private` to `internal` so our other contract can access it.

```js
function _createZombie(string memory _name, uint _dna) internal {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        emit NewZombie(id, _name, _dna);
    }
```    

### 2.10 What Do Zombies Eat?

We assume that our Zombies eat CryptoKitties!

In order to do this we'll need to read the kittyDna from the CryptoKitties smart contract. We can do that because the CryptoKitties data is stored openly on the blockchain.

#### Interacting with other contracts

For our contract to talk to another contract on the blockchain that we don't own, first we need to define an `interface`.

Let's look at a simple example. Say there was a contract on the blockchain that looked like this:

```js
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

This would be a simple contract where anyone could store their lucky number, and it will be associated with their Ethereum address. Then anyone else could look up that person's lucky number using their address.

Now let's say we had an external contract that wanted to read the data in this contract using the `getNum` function.

First we'd have to define an `interface` of the `LuckyNumber` contract:

```js
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

Notice that this looks like defining a contract, with a few differences. For one, we're only declaring the functions we want to interact with — in this case `getNum` — and we don't mention any of the other functions or state variables.

Secondly, we're not defining the function bodies. Instead of curly braces (`{` and `}`), we're simply ending the function declaration with a semi-colon (`;`).

So it kind of looks like a contract skeleton. This is how the compiler knows it's an interface.

By including this interface in our dapp's code our contract knows what the other contract's functions look like, how to call them, and what sort of response to expect.

We'll get into actually calling the other contract's functions in the next lesson, but for now let's declare our interface for the CryptoKitties contract.

Let us see the the CryptoKitties source code. There is a function called `getKitty` that returns all the kitty's data, including its "genes" (which is what our zombie game needs to form a new zombie!).

The function looks like this:

```js
function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
) {
    Kitty storage kit = kitties[_id];

    // if this variable is 0 then it's not gestating
    isGestating = (kit.siringWithId != 0);
    isReady = (kit.cooldownEndBlock <= block.number);
    cooldownIndex = uint256(kit.cooldownIndex);
    nextActionAt = uint256(kit.cooldownEndBlock);
    siringWithId = uint256(kit.siringWithId);
    birthTime = uint256(kit.birthTime);
    matronId = uint256(kit.matronId);
    sireId = uint256(kit.sireId);
    generation = uint256(kit.generation);
    genes = kit.genes;
}
```

The function looks a bit different than we're used to. We can see it returns... a bunch of different values. If you're coming from a programming language like Javascript, this is different — in Solidity you can return more than one value from a function.

Now that we know what this function looks like, we can use it to create an `interface`:

1. Define an `interface` called `KittyInterface`. Remember, this looks just like creating a new contract — we use the `contract` keyword.

2. Inside the `interface`, define the function `getKitty` (which should be a copy/paste of the function above, but with a semi-colon after the returns statement, instead of everything inside the curly braces.

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}
contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```

### 2.11 Using an Interface

Continuing our previous example with `NumberInterface`, once we've defined the `interface` as:

```js
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

We can use it in a contract as follows:

```js
contract MyContract {
  address NumberInterfaceAddress = 0xab38... 
  // ^ The address of the FavoriteNumber contract on Ethereum
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // Now `numberContract` is pointing to the other contract

  function someFunction() public {
    // Now we can call `getNum` from that contract:
    uint num = numberContract.getNum(msg.sender);
    // ...and do something with `num` here
  }
}
```

In this way, we contract can interact with any other contract on the Ethereum blockchain, as long they expose those functions as `public` or `external`.

Let's set up our contract to read from the CryptoKitties smart contract!

1. Save the address of the CryptoKitties contract in the code under a variable named `ckAddress`. In the next line, create a `KittyInterface` named `kittyContract`, and initialize it with `ckAddress` — just like we did with `numberContract` above.

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}
contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```

### 2.12 Handling Multiple Return Values

This `getKitty` function is the first example we've seen that returns multiple values. Let's look at how to handle them:

```js
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // This is how you do multiple assignment:
  (a, b, c) = multipleReturns();
}

// Or if we only cared about one of the values:
function getLastReturnValue() external {
  uint c;
  // We can just leave the other fields blank:
  (,,c) = multipleReturns();
}
```

Let's make a function that gets the kitty genes from the contract:

1. Make a function called `feedOnKitty`. It will take 2 `uint` parameters, `zombieId` and `_kittyId`, and should be a `public` function.

2. The function should first declare a `uint` named `kittyDna`.

* **Note**: In our `KittyInterface`, `genes` is a `uint256` — but if we remember back to Section 1, `uint` is an alias for `uint256` — they're the same thing.

3. The function should then call the `kittyContract.getKitty` function with `_kittyId` and store genes in `kittyDna`. Remember `— getKitty` returns a ton of variables (10 to be exact). But all we care about is the last one, `genes`. Count commas carefully!

4. Finally, the function should call `feedAndMultiply`, and pass it both `_zombieId` and `kittyDna`.

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}
contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna);
  }

}
```

### 2.13 Bonus: Kitty Genes

Our function logic is now complete... but let's add in one bonus feature.

Let's make it so zombies made from kitties have some unique feature that shows they're cat-zombies.

To do this, we can add some special kitty code in the zombie's DNA.

If we recall from Section 1, we're currently only using the first 12 digits of our 16 digit DNA to determine the zombie's appearance. So let's use the last 2 unused digits to handle "special" characteristics.

We'll say that cat-zombies have `99` as their last two digits of DNA. So in our code, we'll say `if` a zombie comes from a cat, then set the last two digits of DNA to `99`.

### If statements

If statements in Solidity look just like javascript:

```js
function eatBLT(string memory sandwich) public {
  // Remember with strings, we have to compare their keccak256 hashes
  // to check equality
  if (keccak256(abi.encodePacked(sandwich)) == keccak256(abi.encodePacked("BLT"))) {
    eat();
  }
}
```

Let's implement cat genes in our zombie code.

1. First, let's change the function definition for `feedAndMultiply` so it takes a 3rd argument: a `string` named `_species` which we'll store in `memory`.

2. Next, after we calculate the new zombie's DNA, let's add an `if` statement comparing the `keccak256` hashes of `_species` and the string `"kitty"`. We can't directly pass strings to `keccak256`. Instead, we will pass `abi.encodePacked(_species)` as an argument on the left side and `abi.encodePacked("kitty")` as an argument on the right side.

3. Inside the `if` statement, we want to replace the last 2 digits of DNA with `99`. One way to do this is using the logic: `newDna = newDna - newDna % 100 + 99;`.

* **Explanation**: Assume `newDna` is `334455`. Then `newDna % 100` is `55`, so `newDna - newDna % 100` is `334400`. Finally add `99` to get `334499`.

4. Lastly, we need to change the function call inside `feedOnKitty`. When it calls `feedAndMultiply`, add the parameter `"kitty"` to the end.

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}
contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

### 2.14 Javascript implementation

Once we're ready to deploy this contract to Ethereum we'll just compile and deploy `ZombieFeeding` — since this contract is our final contract that inherits from `ZombieFactory`, and has access to all the public methods in both contracts.

Let's look at an example of interacting with our deployed contract using Javascript and web3.js:

```js
var abi = /* abi generated by the compiler */
var ZombieFeedingContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFeeding = ZombieFeedingContract.at(contractAddress)

// Assuming we have our zombie's ID and the kitty ID we want to attack
let zombieId = 1;
let kittyId = 1;

// To get the CryptoKitty's image, we need to query their web API. This
// information isn't stored on the blockchain, just their webserver.
// If everything was stored on a blockchain, we wouldn't have to worry
// about the server going down, them changing their API, or the company 
// blocking us from loading their assets if they don't like our zombie game ;)
let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
$.get(apiUrl, function(data) {
  let imgUrl = data.image_url
  // do something to display the image
})

// When the user clicks on a kitty:
$(".kittyImage").click(function(e) {
  // Call our contract's `feedOnKitty` method
  ZombieFeeding.feedOnKitty(zombieId, kittyId)
})

// Listen for a NewZombie event from our contract so we can display it:
ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  // This function will display the zombie, like in lesson 1:
  generateZombie(result.zombieId, result.name, result.dna)
})
```

## 3. Advanced Solidity Concepts

### Immutability of Contracts

Up until now, Solidity has looked quite similar to other languages like JavaScript. But there are a number of ways that Ethereum DApps are actually quite different from normal applications.

To start with, after we deploy a contract to Ethereum, it’s `immutable`, which means that it can never be modified or updated again.

The initial code you deploy to a contract is there to stay, permanently, on the blockchain. This is one reason security is such a huge concern in Solidity. If there's a flaw in our contract code, there's no way for us to patch it later. We would have to tell our users to start using a different smart contract address that has the fix.

But this is also a feature of smart contracts. The code is law. If we read the code of a smart contract and verify it, we can be sure that every time you call a function it's going to do exactly what the code says it will do. No one can later change that function and give us unexpected results.

#### External dependencies

In Section 2, we hard-coded the CryptoKitties contract address into our DApp. But what would happen if the CryptoKitties contract had a bug and someone destroyed all the kitties?

It's unlikely, but if this did happen it would render our DApp completely useless — our DApp would point to a hardcoded address that no longer returned any kitties. Our zombies would be unable to feed on kitties, and we'd be unable to modify our contract to fix it.

For this reason, it often makes sense to have functions that will allow us to update key portions of the DApp.

For example, instead of hard coding the CryptoKitties contract address into our DApp, we should probably have a `setKittyContractAddress` function that lets us change this address in the future in case something happens to the CryptoKitties contract.

Let's update our code from Section 2 to be able to change the CryptoKitties contract address.

1. Delete the line of code where we hard-coded `ckAddress`.

2. Change the line where we created `kittyContract` to just declare the variable — i.e. don't set it equal to anything.

3. Create a function called `setKittyContractAddress`. It will take one argument, `_address` (an `address`), and it should be an `external` function.

4. Inside the function, add one line of code that sets `kittyContract` equal to `KittyInterface(_address)`.

* **Note**: there is a security hole with this function, we'll fix it in the next sub section

```js
pragma solidity >=0.5.0 <0.6.0;
import "./zombiefactory.sol";
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}
contract ZombieFeeding is ZombieFactory {

  KittyInterface kittyContract;

  function setKittyContractAddress(address _address) external {
    kittyContract = KittyInterface(_address);
  }

  function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

### Ownable Contracts

There is a security hole in the previous section!

`setKittyContractAddress` is `external`, so anyone can call it! That means anyone who called the function could change the address of the CryptoKitties contract, and break our app for all its users.

We do want the ability to update this address in our contract, but we don't want everyone to be able to update it.

To handle cases like this, one common practice that has emerged is to make contracts `Ownable` — meaning they have an owner (you) who has special privileges.

#### OpenZeppelin's Ownable contract

Below is the `Ownable` contract taken from the `OpenZeppelin` Solidity library. OpenZeppelin is a library of secure and community-vetted smart contracts that we can use in your own DApps. 

```js
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address private _owner;

  event OwnershipTransferred(
    address indexed previousOwner,
    address indexed newOwner
  );

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  constructor() internal {
    _owner = msg.sender;
    emit OwnershipTransferred(address(0), _owner);
  }

  /**
   * @return the address of the owner.
   */
  function owner() public view returns(address) {
    return _owner;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(isOwner());
    _;
  }

  /**
   * @return true if `msg.sender` is the owner of the contract.
   */
  function isOwner() public view returns(bool) {
    return msg.sender == _owner;
  }

  /**
   * @dev Allows the current owner to relinquish control of the contract.
   * @notice Renouncing to ownership will leave the contract without an owner.
   * It will not be possible to call the functions with the `onlyOwner`
   * modifier anymore.
   */
  function renounceOwnership() public onlyOwner {
    emit OwnershipTransferred(_owner, address(0));
    _owner = address(0);
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    _transferOwnership(newOwner);
  }

  /**
   * @dev Transfers control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function _transferOwnership(address newOwner) internal {
    require(newOwner != address(0));
    emit OwnershipTransferred(_owner, newOwner);
    _owner = newOwner;
  }
}
```

 few new things here we haven't seen before:

Constructors: constructor() is a constructor, which is an optional special function that has the same name as the contract. It will get executed only one time, when the contract is first created.
Function Modifiers: modifier onlyOwner(). Modifiers are kind of half-functions that are used to modify other functions, usually to check some requirements prior to execution. In this case, onlyOwner can be used to limit access so only the owner of the contract can run this function. We'll talk more about function modifiers in the next chapter, and what that weird _; does.
indexed keyword: don't worry about this one, we don't need it yet.
So the Ownable contract basically does the following:

When a contract is created, its constructor sets the owner to msg.sender (the person who deployed it)

It adds an onlyOwner modifier, which can restrict access to certain functions to only the owner

It allows you to transfer the contract to a new owner

onlyOwner is such a common requirement for contracts that most Solidity DApps start with a copy/paste of this Ownable contract, and then their first contract inherits from it.

Since we want to limit setKittyContractAddress to onlyOwner, we're going to do the same for our contract.