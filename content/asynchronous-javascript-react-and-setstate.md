---
title: "Asynchronous JavaScript, React and setState"
date: 2018-08-12T20:01:37+01:00
draft: false
type: "post"
---

One aspect of JavaScript that can confuse programmers that have experience with other programming languages is asynchronous code. 

_But, what exactly is asynchronous code?_

It's probably easier to explain what synchronous code is, which is code that is executed line by line. While JavaScript also has code that runs synchronously, the main reason for its asynchronicity is probably the Internet and Ajax calls. The code sends a request to an IP address and in order to process the requested information, the aforementioned address needs to send a response containing that information.

In the meantime, the next lines of code might run. The JavaScript developer needs to make sure that the code won't run before it has acquired the necessary information it has requested and that can sometimes lead to confusion, or even frustration.

Today, I've had one of these moments.

Thankfully, it was quite easy to fix.

I was building a Markdown Previewer in React and I had the code below:

```javascript
class App extends Component {
  state = {
    text: '',
  }
  convertText = (e) => {
    this.setState({ text: e.target.value });
    document.getElementById('preview').innerHTML = marked(this.state.text);
  }
  render() {
    return (
      <div>
        <h1 id="title">Minimal Markdown Previewer</h1>
        <div id="container">
          <textarea name="" rows="30" id="editor" onChange={this.convertText}>
          </textarea>
          <div id="preview"></div>
        </div>
      </div>
    );
  }
}

export default App;
```

This code, I thought, would take the text from the textarea node, update the text value of the App's state and display the Markdown preview using the marked npm module.

It looks simple enough. However, it wasn't working like that.

As mentioned in [React docs](https://reactjs.org/docs/react-component.html#setstate):

> setState() does not always immediately update the component. It may batch or defer the update until later. This makes reading this.state right after calling setState() a potential pitfall. 

So, what was happening here:

```javascript
convertText = (e) => {
    this.setState({ text: e.target.value });
    document.getElementById('preview').innerHTML = marked(this.state.text);
  }
```

was that one letter was constantly missing from the innerHTML that was set in the div with the id 'preview'. React may delay the setState function for performances issues and that was the case here. setState was updating the state after the next line was setting the innerHTML of the div.

[React docs](https://reactjs.org/docs/react-component.html#setstate) also mention:

> Instead, use componentDidUpdate or a setState callback (setState(updater, callback)), either of which are guaranteed to fire after the update has been applied.

So, the above code can be re-written as:

```javascript
convertText = (e) => {
    this.setState({ text: e.target.value }, 
    () => document.getElementById('preview').innerHTML = marked(this.state.text);
  });
```

Ensuring that the callback function:

```javascript
() => document.getElementById('preview').innerHTML = marked(this.state.text)
```

will be executed after the setState function.

_Asynchronous code is not hard, it just requires a bit of attention and correct planning._