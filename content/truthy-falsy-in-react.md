---
title: "JavaScript truthy and falsy values (in React)"
date: 2018-07-18T17:24:12+01:00
draft: false
type: "post"
---

JavaScript, as any other programming language, has its quirks. Some might argue that it might have a few too many, but that's a long discussion and not in the scope of this post.

On the other hand, most people would agree that, in order to learn a JavaScript library such as React, one should first be comfortable with vanilla JavaScript. Especially high order functions, the ternary operator and the relatively new ES6 features, namely classes, arrow functions, enhanced object literals.

Adding to these, there is also JSX. React's closest thing to a template language. To be exact, a syntax extension to JavaScript. It is a powerful syntax that produces concise code that is fast to write, but it can sometimes confuse a React newcomer.

One thing that got me a while to wrap my head around was conditional rendering.

If statements, the ternary operator and the logical && operator all work. 

However:

If statements need to be outside the return statement of a React component and the logical && operator takes advantage of the way that JavaScript evaluates such an expression, often called short-circuiting.

JavaScript has truthy and falsy values. In plain English, the values

```javascript
false, 0, "", null, undefined, NaN
```

all evaluate to false, thus called falsy.

All other values evaluate to true.

This means that in the code below:

```javascript
if (0) {
  return 'whatever';
}
```

The code within the block will not run.

It also means that running the code below:

```javascript
if (42) {
  return 'Well, hello there!';
}
```

The block will return 'Well, hello there!', since 42 is a truthy value and will evaluate to true.

# If statements

If statements are not valid inside JSX syntax, but the code below should work just fine, since they are outside the return statement.

```javascript
class ConditionalComponent extends React.Component {
  state = {
    isLoggedIn: false
  }
  render() {
    let greeting;
    if (this.state.isLoggedIn) {
      greeting = `Welcome back ${user}`;
    } else {
      greeting = 'You need to be logged in, in order to view this page';
    }
    return (
      <div>
        <p>{greeting}</p>
      </div>
    )
  }
}
```

The ConditionalComponent isLoggedIn state is evaluated and the appropriate message is displayed to the user.


# The ternary operator

The ternary operator works in a straightforward way as well.

```javascript
render() {
  return (
    <div>
      You are <b>{this.state.isLoggedIn ? 'now' : 'not'}</b> logged in.
    </div>
  );
}
```


# The logical && operator

The logical && operator works in a bit of a different way.

In JavaScript, 

```javascript 
true && expression
```
 evaluates to ```expression```, while
 ```javascript 
false && expression
```
evaluates to ```false```.



Based on the above, 
```javascript
if (0 && 42) {
  console.log(5);
}
```
Should not log anything to the console, since the expression 0 && 42 evaluates to 0, which is falsy.

Now, what happens, for example, if we want to display the number of messages a user has received in a React component?

One would maybe think that the code below would work. Well, it works, but maybe not as someone would expect it to.

```javascript
{numberOfMessages && <p>{numberOfMessages} messages received</p>}
```

If the numberOfMessages variable is set to 0, the JSX expression evaluates to 0, which a falsy value.

But this is JSX and a falsy value is not necessary converted to false. Since, 0 is a perfectly fine number to evaluate, if the number of messages are 0, JSX will return the number 0.

In order for the above code to work as expected, it needs a little tweak.

```javascript
{numberOfMessages > 0 && <p>{numberOfMessages} messages received</p>}
```
numberOfMessages > 0 will evaluate to either false or true and in the case it turns out to be false, JSX will not render anything.

It makes sense once you think about it, but it's something that one can miss if there is little familiarity with JSX.

React is fun and JSX sure makes things easier, but it might take a while to click for a beginner.