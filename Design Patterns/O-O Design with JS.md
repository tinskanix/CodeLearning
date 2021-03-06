## FreeCodeCamp Object-Oriented design with javascript
Notes from the O-O module. Some things I have skipped through as I have un grasp de los conceptos already :)
Further material from [this dev article]()  

Use of prototype to share properties around multiple objects
Constructors of an object can be used to check what type of object it is

```
let duck = new Bird();
let beagle = new Dog();

console.log(duck.constructor === Bird); //prints true
console.log(beagle.constructor === Dog); //prints true
```

Manually setting a prototype will erase the constructor property so it has to be manually set 

```
Bird.prototype = {
  constructor: Bird, // define the constructor property
  numLegs: 2,
  eat: function() {
    console.log("nom nom nom");
  },
  describe: function() {
    console.log("My name is " + this.name);
  }
};
```

With inheritance, it must be explicitly declared on child object initialization for inheritance to take effect  
``Object.create`` allows you to create an object which will delegate to another object on failed lookups (it can consult another object to check if it has that property),
in this case the parent objects
```
function Animal() { }

Animal.prototype = {
  constructor: Animal, 
  eat: function() {
    console.log("nom nom nom");
  }
};

// Add your code below this line

let duck = Object.create(Animal.prototype);

```

Once inheritance is declared though, the top-most parent constructor takes precedence, so the parent constructor must be set manually (line 55)
```
(same as above lines 33 - 40)

function Dog() { }
Dog.prototype =  Object.create(Animal.prototype);

let beagle = new Dog();
beagle.eat();  // Should print "nom nom nom"
``` 

You can use the ``new`` keyword to replace ``Object.create``, and the object itself being returned with ``this`` keyword 
```
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)

```

Another good example is Array and how you're p. much creating an instantiation of the Array class when using ``[]``

### Static functions
Static functions can be used as a method specific to that class, but that doesn't have to be shared across instances ! In the following example ``nextToEat(animals){...}`` functions like that
```
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
  static nextToEat(animals) {
    const sortedByLeastEnergy = animals.sort((a,b) => {
      return a.energy - b.energy
    })

    return sortedByLeastEnergy[0].name
  }
}
```
the function does *not* live under the prototype, so it must be called as ``Animal.nextToEat``
With ES5, it's the same as above, but while the non-static functions are added to ``prototype``, the static function is added directly to ``Animal`` object

```
Animal.nextToEat = function (nextToEat) {
  const sortedByLeastEnergy = animals.sort((a,b) => {
    return a.energy - b.energy
  })

  return sortedByLeastEnergy[0].name
}
```
You can easily get an object's prototype sin importar how it was created. The ``prototype`` object will have a ``constructor`` property on the prototype by defalt, with any instances able to access their constructor via ``instance.constructor``.
```
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

const leo = new Animal('Leo', 7)
const prototype = Object.getPrototypeOf(leo)

console.log(prototype)
// {constructor: �, eat: �, sleep: �, play: �}

prototype === Animal.prototype // true
```

Another take away from above es que el code on line 157: ``prototype === Animal.prototype // true`` demonstrates that ``.getPrototypeOf`` returns the prototype which was indeed created from the ``Animal`` object
AND in this case, a separate  prototype for the "leo" object has not been created

### Determine if a property lives on the prototype
There are cases when you  need to find out if a property is from the isntance or the prototype the object delegates to. This can be done with a ``for`` loop, but it would show ALL properties including functions
becasue any property you add to a prototype is enumerable. To fix this, you either specify all prototype mehotds are non-enumerable or use specific function to check if that propery is on the leo object itself. That function is ``hasOwnProperty`` !!
So the ``for`` loop would look like this:
```

const leo = new Animal('Leo', 7)

for(let key in leo) {
  if (leo.hasOwnProperty(key)) {
    console.log(`Key: ${key}. Value: ${leo[key]}`)
  }
}
```

### Check if an object is an instance of a class
This one's pretty straight forward - you can use ``instanceof`` if an object is an instance of a class
```
const leo = new Animal('Leo', 7)

leo instanceof Animal // true
leo instanceof User // false
```
*How does it work?* it checkes for the presence of ``constructor.prototype`` in the object's prototype chain (by using ``.getPrototypeOf`` above!!)

### Creating new agnostic constructor functions
how do you enforce use of "new" for object instantiation? you can use ``instanceof`` in the constructor!
```
function Animal (name, energy) {
  if (this instanceof Animal === false) {
    return new Animal(name, energy)   
  }

  this.name = name
  this.energy = energy
}
```
by the ``if`` statement, the ``this`` keyword is already an ``instanceof`` the constructor function itself 
when it's not, it calls the same function again, but this time with the ``new`` keyword, so it will work esta vez

### Recreating ``Object.create``
To repasar, ``Object.create`` creates objects which delegate to the constructor function's prototype. But how does it work under the hood? hay que recrearlo! Sabemos lo siguiente:
1) it takes an argument that is an object
2) it creates an object that delegates the arument object on failed lookups
3) it returns the new created object

So, with 1) it takes an argument that is an object:
```
Object.create = function (objToDelegateTo){

}
```
With 2) it creates an object that delegates the arument object on failed lookups & 3) it returns the new created object
```
Object.create = function(objToDelegateTo){
  function Fn(){}                 // create empty function, comes with a prototype property
  Fn.prototype = objToDelegateTo  // override prototype of that empty function to the argument object
  return new Fn()                 // return new object, by invoking the empty function using new & return
}
```

### Final Notes

Arrow functions do not have a ``this`` keyword, which result in no constructor functions and will throw an error on ``new``. Arrow functions also do not have a prototype object!
```
const Animal = () => {}

const leo = new Animal() // Error: Animal is not a constructor

```