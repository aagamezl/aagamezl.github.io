---
layout: post
title: Abstract Classes in JavaScript
date: September 20, 2021
author: Álvaro José Agámez Licha
overview: JavaScript does not follow the traditional object-oriented paradigm, but a paradigm called Prototypical Inheritance, this is a big difference compared to languages like C++, C#, Java or PHP, which has its props and cons, but what generates a big difference is the absence of some functionalities of languages that follow the traditional OOP paradigm, such as abstract classes.
permalink: abstract-classes-in-javascript.html
tags:
- JavaScript
- Object Oriented Programming
- software development
- Patterns
---

JavaScript does not follow the traditional object-oriented paradigm, but a paradigm called <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain" target="_blank">Prototypical Inheritance</a>, this is a big difference compared to languages like C++, C#, Java or PHP, which has its props and cons, but what generates a big difference is the absence of some functionalities of languages that follow the traditional OOP paradigm, such as abstract classes.

This was something that bothered me quite a bit until I discovered that this functionality can easily be implemented in JavaScript with just a few lines of code.

Here are two classes, one called Animal will be an abstract class and cannot be instantiated and another called Dog that would be a concrete class that will inherit from the first one.

```javascript
class Animal {
  constructor(classification) {
    // This check make the trick
    if (new.target === Animal) {
      throw new Error('Cannot create an instance of an abstract class.')
    }

    this.classification = classification
  }
}

class Dog extends Animal {
}

const dogClassification = {
  kingdom: 'Animalia',
  phylum: 'Chordata',
  order: 'Carnivora',
  family: 'Canidae',
  genus: 'Canis',
  specie: 'Canis lupus familiaris'
}

// This will throw an exception
const animal = new Animal(dogClassification)

// This will work
const dog = new Dog(dogClassification)
```

The trick, if we want to call it that way, is accomplished with the three lines of code `if` statement inside the constructor. Basically what the `if` does is check if the object being created is of type Animal, if so, then an exception is thrown informing that an abstract class cannot be instantiated, otherwise the constructor continues executing and in the end, we will obtain the instance of the concrete class.
