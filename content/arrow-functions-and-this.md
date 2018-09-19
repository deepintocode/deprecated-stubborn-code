---
title: "JavaScript arrow functions and this"
date: 2018-09-19T23:29:16+01:00
draft: false
type: "post"
---

_Everyone likes arrow functions._

And what's not to like? They look more elegant and they have less characters to type.

So, do they replace the traditional function syntax?

Not really.

The main difference being that they lexically bind their context. 

???

In JavaScript, functions create their own execution context and they do that not when they are defined, but when they are called.

However, arrow functions do not behave in that way, but they take ```this``` from the outer context at the point of their definition.

Practically, that leads to a few solutions, as well as a few issues.

In the example below:

```javascript
class Timer {  
  constructor(sec) {
    this.time = sec;
  }
  countDown() {
    setInterval(()=> {
        this.time--;
        console.log(this.time);
    }, 1000);
  }
}

const timer = new Timer(60);

timer.countDown();
```

While when setInterval is called, it creates its own context, usually the window context, since the callback function is an arrow function, it gets the context from the 'countDown' method.

So, when you discover arrow functions and how great they are, you might be tempted to use them for everything.

But, let's look at the following example:

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.sayName = () => {
  console.log(this.name);
}

const person = new Person('Teo');
person.sayName();
```

While the sayName is method is defined on the Person contructor prototype and called on the person instance, the result comes back as undefined.

So, in this case the function expression would be the way to go. When the arrow function is defined, the context is the window object. 

No matter how that method is called, its ```this``` will always be bound to be the window object.

## <p style="text-align:center">__ALWAYS__</p>

When the arrow function is bound, the context cannot change. Not even using .bind, .call, or .apply.

This might be what you're after. But, be careful.

_There is a time and place for everything. Including arrow functions._ 