---
layout: post
title: Traits in JavaScript
date: September 29, 2021
author: Álvaro José Agámez Licha
overview: As a developer who worked with PHP for many years, I sometimes miss PHP functionalities in JavaScript, one of those functionalities is Traits, a feature that allows code reuse in a really easy way.
permalink: traits-in-javascript.html
tags:
- JavaScript
- Object Oriented Programming
- Software Development
- Patterns
---

As a developer who worked with PHP for many years, I sometimes miss PHP functionalities in JavaScript, one of those functionalities is <a href="https://www.php.net/manual/en/language.oop5.traits.php" target="_blank">Traits</a>, a feature that allows code reuse in a really easy way.

Code reuse is one of the most important aspects of object-oriented programming, we move the common functionality of classes to methods of the parent class,however, inheritance makes the code very tightly coupled. Therefore, overuse of inheritance may cause the code very hard to maintain.

A trait is similar to a class, but it is only for grouping methods in a fine-grained and consistent way. The idea is not to instantiate the traits or use the traits directly.

In the JavaScript implementation that I will present to you today, in addition to grouping methods, you will also be able to group properties, this will allow great flexibility when using traits to reuse code in our applications.  This is in line with the definition of PHP traits, additionally when traits are applied to a class, all the classes that inherit from it will have access to the methods or properties that the traits add to the base class.

```javascript
const use = (traitTarget, ...traits) => {
  if (traits.length === 0) {
    return traitTarget
  }

  const traitTargetPrototype = Object.getPrototypeOf(
    Object.getPrototypeOf(traitTarget)
  )

  traits.forEach(trait => {
    Object.setPrototypeOf(trait, traitTargetPrototype);

    Object.setPrototypeOf(traitTarget, Object.assign(
      Object.getPrototypeOf(traitTarget),
      trait
    ))
  })

  return traitTarget
}
```

The previous function takes as the first parameter an object that would be the object to which we want to apply the traits and as the second parameter the list of traits that we will apply.

Next we will see how to use the function to apply a trait; each trait has to be a literal object, which will group the properties and functions to be added:

```javascript
const SayWorld = {
  message: 'This is a Trait message',

  sayHello() {
    super.sayHello()
    console.log('World!')
  },

  otherMethod() {
    console.log('otherMethod()')
  },

  showMessage() {
    console.log(this.message)
  }
}

const SayBye = {
  sayBye() {
    console.log('Good Bye!')
  },
}

class Base {
  sayHello() {
    console.log('Hello ')
  }
}

class MyHelloWorld extends Base {
  constructor() {
    super()

    use(this, SayWorld, SayBye)
  }
}

class DerivedClass extends MyHelloWorld {
}

const instance = new MyHelloWorld()
instance.sayHello()

const derived = new DerivedClass()
derived.otherMethod()
derived.showMessage()
derived.sayBye()
```
As we can see, we have a trait that is a literal object that groups a property and 3 methods. The traits are applied in the classes's constructor, passing as the first parameter the object `this` that represents the current object and 2 different traits.

The trait pattern is a powerful tool to increase our code reuse, and I hope that my implementation help you to improve your codebases.