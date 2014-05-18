---
title: Quick-Start Guide
template: page.ejs
---

Quick-Start Guide
=================

Once you have Fluxbox and React installed, it's time to build an app! Since this should be quick and simple, we'll build with a *very* basic todo app—namely, this one right here:

<div id="app"></div>
<script src="todo-bundle.js"></script>

Stores and Actions
------------------

First, let's create a store to keep track of our todo items. It will respond to the following actions:

* `"ADD_TODO"` - adds a new todo
* `"TOGGLE_TODO"` - completes or uncompletes a specific todo item
* `"CLEAR_TODOS"` - removes all complete todo items

```javascript
var TodoStore = Fluxbox.createStore({
  actions: {
    "ADD_TODO": "onAddTodo",
    "TOGGLE_TODO": "onToggleTodo",
    "CLEAR_TODOS": "onClearTodos"
  },

  initialize: function() {
    this.todos = [];
  },

  onAddTodo: function(payload) {
    this.todos.push({text: payload.text, complete: false});
    this.emit("change");
  },

  onToggleTodo: function(payload) {
    payload.todo.complete = !payload.todo.complete;
    this.emit("change");
  },

  onClearTodos: function() {
    this.todos = this.todos.filter(function(todo) {
      return todo.complete;
    });
    this.emit("change");
  },

  getState: function() {
    return {
      todos: this.todos
    };
  }
});
```

Let's create a few semantic actions to go along with our action types.

```javascript
var actions = {
  addTodo: function(text) {
    this.dispatch("ADD_TODO", {text: text});
  },

  toggleTodo: function(todo) {
    this.dispatch("TOGGLE_TODO", {todo: todo});
  },

  clearTodos: function() {
    this.dispatch("CLEAR_TODOS");
  }
};
```

Now we can instantiate our store and build a `Flux` instance:

```javascript
var stores = {
  TodoStore: new TodoStore()
};

var flux = new Fluxbox.Flux(stores, actions);
```

React Application
-----------------

Let's build out our UI with React. Don't forget `/** @jsx React.DOM */` at the top of the file if you're using JSX!

Our top-level `Application` component will use the [FluxMixin](/documentation/flux-mixin) as well as the [StoreWatchMixin](/documentation/store-watch-mixin) to make our lives a bit easier. It will iterate over the array of todos and emit a `TodoItem` component for each one.

We'll also add a quick form for adding new todo items, and a button for clearing completed todos.

```javascript
var FluxMixin = Fluxbox.FluxMixin(React),
    StoreWatchMixin = Fluxbox.StoreWatchMixin(React);

var Application = React.createClass({
  mixins: [FluxMixin, StoreWatchMixin("TodoStore")],

  getStateFromFlux: function() {
    var flux = this.props.flux;
    // normally one key per store, but we only have one store
    return flux.store("TodoStore").getState();
  },

  render: function() {
    return (
      <div>
        <ul>
          {this.state.todos.map(function(todo, i) {
            return <li key={i}><TodoItem todo={todo} /></li>;
          })}
        </ul>
        <form onSubmit={this.onSubmitForm}>
          <input ref="input" type="text" size="30" placeholder="New Todo" />
          <input type="submit" value="Add Todo" />
        </form>
        <button onClick={this.clearCompletedTodos}>Clear Completed</button>
      </div>
    );
  },

  onSubmitForm: function(e) {
    e.preventDefault();
    var node = this.refs.input.getDOMNode();
    this.props.flux.actions.addTodo(node.value);
    node.value = "";
  },

  clearCompletedTodos: function(e) {
    this.props.flux.actions.clearTodos();
  }
});
```

The `TodoItem` component will display and style itself based on the completion of the todo, and will dispatch an action to toggle its state.

```javascript
var TodoItem = React.createClass({
  propTypes: {
    todo: React.PropTypes.object.isRequired
  },

  contextTypes: {
    flux: React.PropTypes.object
  },

  render: function() {
    var style = {
      "text-decration": this.props.todo.complete ? "line-through" : ""
    };

    return <span style={style} onClick={this.onClick}>{this.props.todo.text}</span>;
  },

  onClick: function() {
    this.context.flux.actions.toggleTodo(this.props.todo);
  }
});
```

Bringing it Together
--------------------

Now that we have a `Flux` instance and all our components are defined, we can finally render our app. We'll put it inside a `div` in our HTML with an ID of "app".

```javascript
React.renderComponent(<Application flux={flux} />, document.getElementById("app"));
```

And that's it! We've created a (super simple) Flux application with React and Fluxbox. You can find the full source code [on GitHub](https://github.com/BinaryMuse/fluxbox/tree/master/examples/todo-basic).