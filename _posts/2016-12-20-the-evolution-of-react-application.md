---
layout: post
title: The Evolution of React Applications
---

Everyone knows that we live in the world of trends. We saw rise and falls of many frameworks and techs.

And the question is why do we need something new? Why do we need another tech? I’ve thought for much time and finally have decided that the answer is convenience.

This is the reason why we moved from JQuery and some direct DOM manipulations to the frameworks like Ember, Angular, Backbone, and the most exciting one - React. And of course I always say and think about no just React, but React/Redux.

In this post, I’m going to show why we do this, why we move further. I think, this article will be interesting not only for newbie in React, but for experienced React developers too.

Well, I prefer to imagine structure of everything, and what about React/Redux ecosystem I imagine the following roadmap:

JQuery/Direct DOM manipulations -> React -> React/Redux -> React/Redux + Thunk/Saga -> Deep into the React Ecosystem

Why do we need React, not just JQuery&co? You know the answer - convenience! Imagine you need to delete item from a cart. In the JQuery&co way you will do the following: find the DOM node via a scary selector, then remove it… sooo imperative...  In the React way you will do the following:

```javascript
this.setState({
  items: this.state.filter(
    item => item.id !== removedItemId
  ),
});
```

I think it’s simple enough. If you’re newbie to react, don’t worry,  we’ll discuss it later more accurate.

React first of all gives us a simple and smart way to rerender/redraw our UI. It consumes via the changing the state of our application. Something changed - rerender - repeat.

Let's move through this roadmap, but of course we skip the JQuery&co part. In this we tutorial we are going to create a simple todo app (I know that it is the mainstream).

I’d recommend you to use create-react-app to avoid useless preparation.

## Stage 1. React

In this part we’ll create an app using only React. I hope, you know what in JSX and how looks basic React app. If you don’t, it’d be better to start with official docs. It’s not about practice or something more than docs, but it gives you the basic understanding what is happening.

Let's start!

Firstly, we divide our components into two kinds: containers, which store the state of the app, and presentations, which only represent the state. We’ll have only one container: TodoContainer.

```javascript
import React, { Component } from "react";

class TodoContainer extends Component {
  constructor() {
    super();

    this.state = {
      Items: [],
    };
  }

  render = () => (
    <ItemList items={this.state.items} />
  );
}
```

The first version of our TodoContainer is ridiculously simple. Notice that we save the all app state in only one component - container. For the apps like this it’s OK, but when your app grow up, you will feel that was bad idea to save the state in one component.

Create one simple presentation component:

import React, { Component } form ‘react’;

```javascript
class ItemList extends Component {
  render = () => (
    <ul>
      {this.props.items.map(item => (
				<li key={Date.now()}>
					{item.content}
				</li>
			))}
    </ul>
  );
}
```

Well, I think, you are following the idea. Now, we’ll add an ability to create and delete items.

```javascript
class TodoContainer extends Component {
  constructor() {
    super();

    this.state = {
      items: [],
      newItemContent: ‘’,
    };
  }

  addItem = () => {
    const item = {
      content: this.state.newItemContent,
      id: Date.now()
    };

    this.setState({
      items: this.state.items.concat([item]),
    });
  }

  deleteItem = () => (itemId) => {
    this.setState({
      items: this.state.items.filter(
				item => item.id !== itemId
			),
    });
  }

  handleChange = (event) => {
    this.setState({
      newItemContent: event.target.value,
    });
  }

  render = () => (
    <div>
      <ItemList
        items={this.state.items}
        delete={this.deleteItem}
      />
      <input onChange={this.handleChange} />
      <button onClick={this.addItem}>+</button>
    </div>
  );
}
```

Let's refactor our ItemList component for beauty sake:

```javascript
const ItemList = ({ item }) => (
  <ul>
    {this.props.items.map(item => (
      <li key={item.id}>
        {item.content}
        <button
          onClick={this.props.deleteItem(item.id)}
        >
          DELETE
        </button>
      </li>
    ))}
  </ul>
);
```

I think that's better. Call it the pure component, because it just gets props and return the JSX snippet.

As you can see, in React components are talking to each other in two direction using two different types of communication:
  * Parent   -> Children: parent gives children props - static info, callbacks
  * Children -> Parent: call parent’s callbacks

Easy as pie ;)

There are some React best practice, such as a high-ordered function (deleteItem returns a new function), a pure component, and a controlled input.

React is awesome! The best thing about it is explicitness. Your pass the props in on your own, you control inputs (sometimes), you compose components - the tiny dictatorship :3 But it's not bug-prone. You always see what is happening. This is great!

Weeeeell, do you see where it's all going to? Yeah, straight to the HELL! Just kidding ;)
Our container component is growing, the of app is growing too... It's becoming less reusable and maintainable.
You'll hate it when you add a new feature...
You know that in development it is easy to write working code, but hard to write maintainable code.

It's time to Redux!

## Stage 2. React & Redux

Redux helps us to save the state of our application in one place - store.
When we change the store, our UI is redrawed.

In general, Redux consists in store, actions, and reducers.
Actions are plain objects, which tell reducers how to change state of our application.
Reducers are function, which get a current state and action, which we send to it, and
depending on the action type (action has a field 'type') reducer returns a new state.

Well, the picture is the following:
we click a button -> dispatch action to the reducer -> reducer returns a new state -> our app is redrawed

I'd recommend to print the following picture and stick it somewhere:
![redux circle]('./redux_circle.png')

Let's practice!

We'll remove the logic from our awesome TodoContainer and divide it between action creators, which return actions, and the reducer:

Firstly, action creators:

```javascript
const todo = {};

todo.addItem = (content) => ({
	type: 'ADD_ITEM',
	payload: {
		content,
	},
});

todo.deleteItem = (itemId) => ({
	type: 'DELETE_ITEM',
	payload: {
		itemId,
	},
});
```

Reducer:

```javascript
  const initialState = {
		items: [],
	};

  const todoReducer = (state = initialState, action) => {
		switch (action.type) {
			case 'ADD_ITEM': {
				const { content } = action.payload;
				const item = { content, id: Date.now() };
				const items = state.items.concat([item]);

				return {
					...state,
					items,
				}
			}

			case 'DELETE_ITEM': {
				const { itemId } = action.payload;
				const items = state.items.filter(item => item.id !== itemId);

				return {
					...state,
					items,
				}
			}

			default:
				return state;
		}
	};
```

Now we need to do two things:
	* create store
	* connect Redux to the React app

First, create store:

```javascript
	import { createStore } from 'redux';

	const store = createStore(todoReducer);
```

Second, the connection:

```javascript
	import { connect } from 'react-redux';

	import actions from 'src/actions';

	import Databoard from 'src/components/Databoard/Personal';

	const mapStateToProps = state => ({
	  items: state.items,
	});

	const mapDispatchToProps = dispatch => ({
		addItem: (content) => dispatch(todo.addItem(content)),
	  deleteItem: (itemId) => dispatch(todo.deleteItem(content)),
	});

	TodoContainer = connect(mapStateToProps, mapDispatchToProps)(Todo);
```

For better understanding read the article about contexts on the official docs.

Add some changes to our react part (rebuild our TodoContainer component, and rename it):

```javascript
class Todo extends Component {
  constructor() {
    super();

    this.state = {
      content: ‘’,
    };
  }

  addItem = () =>
    this.props.addItem(this.state.content);

  handleChange = (event) => this.setState({
    newItemContent: event.target.value,
  });

  render = () =>  (
    <div>
      <ItemList
        items={this.props.items}
        deleteItem={this.props.deleteItem}
      />
      <input onChange={this.handleChange} />
      <button onClick={this.addItem}>
        +
      </button>
    </div>
  );
}
```

Notice, that the children components don't need to be changed. They live in piece, getting the info about the world from props.
They are reusable as much as we can do this.

I know what you're thinking about: "What was that? It's more complex that previous version". Yes, it's.
We more complex architecture, but it became more scalable. When you are able to add new feature like search or checking, just add a
new action creator and a new `case` to the reducer. The app will be like this: the React part is just a UI/view, the Redux part is just
a logic and the state of our application. Scalable, simple, perfect.

Let's go further. We'll add an ability to fetch our data from a server, send it, etc.

## Stage 3. React & Redux + Thunk

Well, where should we add a server request part? The answer is Thunk actions!
The basic algo of its' work process is the following:
	1. You invoke a Thunk action like any other.
	2. Within it you dispatch plain action, when server responses.
	3. Enjoy!

The basic Thunk action looks like this:
```javascript
	const action = data => ({
	  type: 'ACTION',
		payload: { data },
	});

	const thunkAction = data => dispatch =>
		fetch('path/to/server', { method: 'GET' body: data })
		.then(response => response.json())
		.then(data => dispatch(action(data)));
```

Let's implement some actions for our small project. I am so lazy to do all those, so that I'm gonna
implement only the index (get all items) and post (create the new one).

Get all todo's items:
```javascript
todo.getItems = () => dispatch =>
	fetch('/path/to/server')
	.then(response => response.json())
	.then(data => dispatch({
		type: 'FETCH',
		payload: {
			items: data,
		},
	}));
```

Create action:
```javascript
todo.createItem = content => dispatch =>
	fetch('/path/to/server', { method: 'POST', body: content})
	.then(() => dispatch(todo.addItem(content)));
```

Thus we need to update our reducer:
```javascript
	const todoReducer = (state = initialState, action) => {
		switch (action.type) {

			// enough code here

			case 'FETCH': {
				const { items } = action.payload;

				return {
					...state,
					items,
				}
			}
		}
	};
```

And update the container component:
```javascript
	//another code here

	const mapDispatchToProps = dispatch => ({
		fetchItems: () => dispatch(todo.getItems);
		addItem: (content) => dispatch(todo.createItem(content)),
	  deleteItem: (itemId) => dispatch(todo.deleteItem(content)),
	});

	TodoContainer = connect(mapStateToProps, mapDispatchToProps)(Todo);
```

That's all. Really. Actually not... we need to trigger an event to fetch data from server
let's do this at the Todo component:
```javascript
	class Todo extends Component {
		//code here

		componentWillMount() {
			this.props.fetchItems();
		}

		//and there
	}
```

We use here a lifecycle method `componentWillMount`. Read more at the official docs.
And that's all.

## Stage 4. And beyond...

There are countless libraries, component, frameworks for React. I think that the most essential are
the following:
	Redux, React Router, Thunk, Saga, Flow.

## Conclusion

I can include all volume of React's ecosystem at the one article. Nobody can.
Move further. Read official docs, watch screencasts, follow some React people.

As you can React application can evolve painless, and it's of course depends on how do you use React.
