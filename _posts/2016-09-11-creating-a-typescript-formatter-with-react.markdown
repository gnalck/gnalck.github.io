---
layout:     post
title:      Creating a Typescript Formatter With React
date:       2016-09-01 15:31:19
summary:    A walkthrough of how I created typescript-formatter, my first React application.
categories: react typescript
---

I recently started [exploring React](https://facebook.github.io/react/docs/tutorial.html), and was looking for something simple to build myself, that was outside the realm of the tutorials I went through. Having also been [exploring Angular 2 and Typescript](https://angular.io/docs/ts/latest/quickstart.html) lately, I went looking for things that I could create that would be something functional and useful that merged these current.

We will walk through the steps needed to create a simple Typescript formatter. This will be coded up in Javascript, but we will leverage the Typescript library's formatting service. I assume you have already coded things in React before - if you haven't I highly recommend going through the excellent [official React tutorial](https://facebook.github.io/react/docs/tutorial.html) before jumping in here.

At the end, we will have a React application that we input Typescript code into, select the tab size we want, and be able to get formatted output, all within the browser.

### Want to just see the source?

You can see it in action over [here](http://gnalck.github.io/typescript-formatter). The code is, of course, all available on [Github](https://github.com/gnalck/typescript-formatter).

### Running the server

Since this is a static application, we only need a simple server to send us the static files. For this, we can run [http-server](https://www.npmjs.com/package/http-server) from the root of our application:

```bash
sudo npm install -g http-server
cd typescript-formatter/
http-server
```

### Getting started

Create a local folder and create two files, `index.html` and `app.js`. For the `index.html` file, we need to add the libraries that we will be using for this tutorial. In addition to [React and ReactDom](https://facebook.github.io/react/docs/top-level-api.html), we will be using [Babel](https://babeljs.io/) to compile JSX/ES6 in-browser, and of course [Typescript](https://www.typescriptlang.org/) for the formatting service it exposes. Lastly, we will be using [Bootstrap](http://getbootstrap.com/) to style the application.

The resulting `index.html` file will look like this:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>React Tutorial</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" >
    <script src="https://unpkg.com/react@15.3.1/dist/react.js"></script>
    <script src="https://unpkg.com/react-dom@15.3.1/dist/react-dom.js"></script>
    <script src="https://unpkg.com/babel-core@5.8.38/browser.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/typescript/2.0.2/typescript.js"></script>
  </head>
  <body>
    <div id="content"></div>
    <script type="text/babel" src="app.js"></script>
  </body>
</html>
```

## Creating the React UI

Let's start creating the React frontend. First, let's create the initial scaffolding of the application. Everything will be contained within the root component `FormatBox`:

```javascript
var FormatBox = React.createClass({
  render: function() {
    return (
      <div className="formatBox container">
        <div className="page-header">
          <h1>Typescript Formatter</h1>
        </div>
      </div>
    );
  }
});

ReactDOM.render(
  <FormatBox />,
  document.getElementById('content')
);
```

We can think of the application of consisting of a `FormatForm` component which will contain all the user-configurable information, a `FormattedCode` component which we will output the formatted code to.

Let's add the `FormattedCode` component, which will take in some text (the formatted code) and display it to the browser:

```javascript
var FormattedCode = React.createClass({
  render: function() {
    if (this.props.formattedCode) {
      return (
        <div className="formattedCode">
          <textarea name="formattedCode" className="form-control" value={this.props.formattedCode} readOnly />
        </div>
      );
    }
    return <div></div>;
  }
});
```

Next, we need to wire up our `FormatBox` component to render the `FormattedCode`.

```javascript
var FormatBox = React.createClass({
  render: function() {
    return (
      <div className="formatBox container">
        <div className="page-header">
          <h1>Typescript Formatter</h1>
        </div>
        // add in FormattedCode
        <FormattedCode formattedCode={}
      </div>
    );
  }
});
```

## Coding up the formatter


## Finished

And that's it - you now have a React application that can format Typescript code!

Of course, this can still be improved in many ways. If you want to continue hacking, here are a couple of recommendations for next steps:

* **Styling**: Create a `css` file and make it look a bit nicer, perhaps also utilizing [Font Awesome](http://fontawesome.io/) to get some fancy icons in there.
* **More configuration**: We only exposed the `TabSize` to the user, try exposing UI for some of the other options, or even all of them! 
* **[RxJS](https://github.com/Reactive-Extensions/RxJS)**: Ditch the button and instead update the formatted text in real time, or at least when the user temporarily stops typing.