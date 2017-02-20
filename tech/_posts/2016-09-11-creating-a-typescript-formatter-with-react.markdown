---
layout:     post
title:      Creating a Typescript Formatter With React
date:       2016-09-11 15:31:19
---

I recently started [exploring React](https://facebook.github.io/react/docs/tutorial.html), and was looking for something simple to build myself, that was outside the realm of the tutorials I went through. Having also been [exploring Angular 2 and Typescript](https://angular.io/docs/ts/latest/quickstart.html) lately, I went looking for things that I could create that would be something functional and useful that merged these two things.

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

### FormattedCode component

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

Next, we need to wire up our root `FormatBox` component to render the `FormattedCode` component. We do this by adding it to our JSX object we return in `render()`. Since the `FormattedCode` component expects a `formattedCode` prop as input, we will also set up the state object and pass it in to the child component.

```javascript
var FormatBox = React.createClass({

  // set up state
  getInitialState: function() {
    return {formattedCode: ''};
  },

  render: function() {
    return (
      <div className="formatBox container">
        <div className="page-header">
          <h1>Typescript Formatter</h1>
        </div>

        // add in FormattedCode component
        <FormattedCode formattedCode={this.state.formattedCode} />
      </div>
    );
  }
});
```

Now that we have the `FormattedCode` component, lets dive into the `FormatForm`. This component will contain the `<textarea/>` to hold the code that needs to be formatted, the options (e.g., tab space size configuration), and a button the user can press to kick off the formatting process.

### FormatForm component

For simplicity's sake, we will keep all of the state in the root component. In addition to the `formattedCode` state property we added above, let's add two more: `code` that will hold the code-to-be-formatted, and `indentSize` which will hold the user-selected indent size. We can do this in the `getInitialState()` function over in our root component:


```javascript
var FormatBox = React.createClass({
  getInitialState: function() {
    return {
      formattedCode: '',
      indentSize: 4,
      code: initText,
    };

    // render, etc
  }
});
```

To update the state in these variables, we will need to code up the handlers that we will use later on. Let's create a seperate one for each state property:

```javascript
var FormatBox = React.createClass({
  handleCodeChange: function(e) {
    this.setState({
      code: e.target.value,
    });
  },

  handleIndentChange: function(e) {
    this.setState({
      indentSize: e.target.value
    });
  },

  handleSubmit: function(e) {
    e.preventDefault();

    var formattedCode = this.state.code; // TODO: actually format it!

    this.setState({
      formattedCode: formattedCode
    })
  },
  
  // getInitialState, render, etc
});
```

Lastly, we need to change the `render` to pass these down into our yet-to-be-created child component. To me, the beauty of React is being able to just throw together the jsx before we even code up the backing component, like so:

```javascript
var FormatBox = React.createClass({
  render: function() {
    return (
      <div className="formatBox container">
        <div className="page-header">
          <h1>Typescript Formatter</h1>
        </div>

        // add in the FormatForm component
        <FormatForm
          code={this.state.code}
          handleCodeChange={this.handleCodeChange}
          indentSize={this.state.indentSize}
          handleIndentChange={this.handleIndentChange}
          handleSubmit={this.handleSubmit} />

        <FormattedCode formattedCode={this.state.formattedCode} />
      </div>
    )
  },

  // getInitialState, handlers, etc
});
```

Now let's create the `FormatForm` component! Note that we create a new child component for the FormatIndent selection. Of course, this is not needed, but is done for greater modulariy and seperation of concerns. Just as we back the textarea with `this.code.props` and its associated handler, we will also back the yet-to-be-created `FormatIndent` child component in the same way.

```javascript
var FormatForm = React.createClass({
  render: function() {
    return (
      <form className="formatForm" onSubmit={this.props.handleSubmit}>
        <div className="form-group">
          <textarea className="form-control" name="code" value={this.props.code} onChange={this.props.handleCodeChange} />
        </div>
        <FormatIndent indentSize={this.props.indentSize} handleIndentChange={this.props.handleIndentChange}/>
        <input type="submit" className="btn btn-primary center-block" value="Format"/>
      </form>
    );
  }
});
```

Next up, we need to add the `FormatIndent` child component. This will consist of a `<select/>` whose selection is backed by the `indentSize` and its associated handler, which are passed in as `props` to the component.

### FormatIndent component

This component will be similar to the `FormattedCode`, or really all the other components (except the root `FormatBox` one) that we have coded up so far. All the state and application logic is in the root component, so here we only need to code up the simple reactive UI.

Let's add the option for 2, 3, and 4 space tab-width into the `<select/>`:

```javascript
var FormatIndent = React.createClass({
  render: function() {
    return (
      <div className="form-group">
        <select className="form-control" 
          value={this.props.indentSize} 
          onChange={this.props.handleIndentChange} >
          <option value="4">4 Space Tab</option>
          <option value="3">3 Space Tab</option>
          <option value="2">2 Space Tab</option>
        </select>
      </div>
    )
  }
})
```

Pretty simple, though perhaps not as simple as the textual components we created till now. Basically, if `value` is `4`, then the corresponding option will be selected, you can read more about this over in the [React Forms documentation](https://facebook.github.io/react/docs/forms.html).

## Coding up the formatter

We now got a nice compositional UI set up, time to actually make it functional. Let's start by creating a function for formatting our code, and changing `handleSubmit` to call it. We will pass in the current state of the code, and the selected indentSize, and we want to get the formatted code back as output:

```javascript
var formatCode = function(code, indentSize) {
  return code; // TODO: format it!
}

var FormatBox = React.createClass({
  handleSubmit: function(e) {
    e.preventDefault();

    // add call to our new function
    var formattedCode = formatCode(this.state.code, this.state.indentSize);

    this.setState({
      formattedCode: formattedCode
    })
  },

  // handlers, getInitialState, render, etc...
});
```

Now we need to flesh out our `formatCode` function to leverage Typescript library to return nicely formatted code back to us. This formatted code will be set to the `formattedCode` state, and everything will then be displayed to the user via `render`.

### Creating the formatCode function

Now we must dig into the exposed Typescript APIs. We will be using the example from this excellent peice of documentation over in the [Typescript Wiki](https://github.com/Microsoft/TypeScript/wiki/Using-the-Compiler-API).

Essentially, there is a type of `FormatCodeOptions` whose properties exposes the configuraiton we give to the Typescript `formatting` service. We then pass those in to a function that formats the code and returns the formats back as `TextChanges`, essentially an object that describes a change and its place within the original text. We then aply those changes to the unformaatted code, and return back the result - which is the newly formatted code!

Let's start off by simply creating this `FormatCodeOptions` object. For this, I just copied over the [default options object](https://github.com/Microsoft/TypeScript/blob/e16cf96b4138c602daebc304287814c1fc15451a/src/server/editorServices.ts#L1582) Typescript uses, since it is not exposed:

```javascript
var formatCode = function(code, indentSize) {
  var options =  {
    BaseIndentSize: 0,
    IndentSize: indentSize, // our override
    TabSize: indentSize, // our override
    NewLineCharacter: "\n",
    ConvertTabsToSpaces: true,
    IndentStyle: ts.IndentStyle.Block,
    InsertSpaceAfterCommaDelimiter: true,
    InsertSpaceAfterSemicolonInForStatements: true,
    InsertSpaceBeforeAndAfterBinaryOperators: true,
    InsertSpaceAfterKeywordsInControlFlowStatements: true,
    InsertSpaceAfterFunctionKeywordForAnonymousFunctions: false,
    InsertSpaceAfterOpeningAndBeforeClosingNonemptyParenthesis: false,
    InsertSpaceAfterOpeningAndBeforeClosingNonemptyBrackets: false,
    InsertSpaceAfterOpeningAndBeforeClosingTemplateStringBraces: false,
    InsertSpaceAfterOpeningAndBeforeClosingJsxExpressionBraces: false,
    InsertSpaceAfterTypeAssertion: false,
    PlaceOpenBraceOnNewLineForFunctions: false,
    PlaceOpenBraceOnNewLineForControlBlocks: false,
  }

  //...
}
```

Now we need to do the song-and-dance needed to get back the diff of changes needed to format the code. Again, this is from the documentation over at [Using the Compiler API](https://github.com/Microsoft/TypeScript/wiki/Using-the-Compiler-API):

```javascript
var formatCode = function(code, indentSize) {
  var options =  {
    //...
  };

  var format = function(input, options)  {
    var rulesProvider = new ts.formatting.RulesProvider();
    rulesProvider.ensureUpToDate(options);

    var sourceFile = ts.createSourceFile("input", input, ts.ScriptTarget.Latest);
    var edits = ts.formatting.formatDocument(sourceFile, rulesProvider, options);

    // TODO: create this!
    return applyEdits(input, edits);
  };

  return format(code, options);
};
```

Lastly, we create an `applyEdits` function that applies the changes to the original code. For each individual `TextChange`, we have a `span.start` field that indicates where in original code this specific change begins at, and a `span.length` field indicating how long the change is. We use these to cut out the code before the edit (the `head`) and the code after the edit (the `tail`) and put the code inbetween, and we do this for each edit.

```javascript
var formatCode = function(code, indentSize) {
  var options =  {
    //...
  };

  var applyEdits = function(text, edits) {
    // Apply edits in reverse on the existing text
    let result = text;
    for (let i = edits.length - 1; i >= 0; i--) {
        let change = edits[i];
        let head = result.slice(0, change.span.start);
        let tail = result.slice(change.span.start + change.span.length)
        result = head + change.newText + tail;
    }
    return result;
  };

  var format = function(input, options)  {
    //...
  };

  return format(code, options);
};
```

#### Why in reverse?

When we apply the change, the structure of the string _after_ the beginning of that change is possibly altered, either growing longer or being cut shorter. So we go backwards, since changes can only affect changes that come after them.

## Finished

And that's it - you now have a React application that can format Typescript code!

Of course, this can still be improved in many ways. If you want to continue hacking, here are a couple of recommendations for next steps:

* **Styling**: Create a `css` file and make it look a bit nicer, perhaps also utilizing [Font Awesome](http://fontawesome.io/) to get some fancy icons in there.
* **More configuration**: We only exposed the `TabSize` to the user, try exposing UI for some of the other options, or even all of them! 
* **[RxJS](https://github.com/Reactive-Extensions/RxJS)**: Ditch the button and instead update the formatted text in real time, or at least when the user temporarily stops typing.