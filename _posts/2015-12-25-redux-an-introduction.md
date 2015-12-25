---
layout: post
title: Redux&#58; A very simple introduction
date: 2015-12-25
author: Aaron Norby
categories: redux
---

Redux (with React) is pretty much the hottest thing in the world right now (if the
world consists in frontend JavaScript frameworks), so there are about a million
introductions to it. I don't intend this to be an even remotely thorough
introduction -- the point is to try to boil Redux down to its most basic ideas.
For some reason, more than any other framework I've used, the difference between
how impenetrable Redux seemed when I first learned it and how simple it seems now that
I know it, is huge. I don't know exactly why that is (maybe because the name is
not so great since, in addition to being Latin, it has no clear connection to what
Redux actually is?), but I hope this can serve to give some clarity and orientation
to those who are coming to Redux fresh. 

I'm not going to put this into the format of the familiar 'how to make an app with
technology *x*'. Instead, the goal is to get explain the conceptual building blocks
that make sense out of Redux. 

# Overview 

Whether or not you're going to use Redux with React, I think it's easiest to get a
sense for Redux by thinking of it in the context of React, since you can then have
an idea of how it can be put to use -- and generally it's hard to understand
something unless you get how it can be used. So I'll go into that at the end. 

Here's an analogy to get things started, before they get too abstract. If you don't
know the legend of Paul Revere, here's the basics. It's the beginning of the
American Revolution. The revolutionaries want to know when the British are on their
way, and they want to know if they're coming by sea in boats or by land on their
feet and horses. The idea is that if they're coming by land then a single lantern
should be lit. And if it's by sea, then two lanterns should be lit. 

Imagine you're in charge of logistics. So first, you write down some instructions: "If you
see them coming by land, light one lantern. If you see them coming by sea, light
two lanterns." That's not gonna do much on it's own. You need someone paying
attention and following the instructions -- someone to light the lanterns in
response to British troop activities. (In the story, Paul Revere was that person.)

If only the American rebels had had JavaScript, Redux could have organized this
whole situation for them. Redux has something called 'actions', which are just
dispatched events that kick off state changes -- one action could be 'the British
are coming by land', the other 'the British are coming by sea'. Redux applications
have state -- in this case, the state of the lanterns: one, two, or off. Redux has
something called a 'reducer' which maps from actions and prior states to new
states: those are the instructions. Finally, Redux has something called a 'store',
which holds the reducer and pays attention to what actions occur and then updates
the state on that basis. In this case, Paul Revere is the store: he's got the
instructions and watches for actions/events and updates the lantern state in
accordance. 

And that's really the core of Redux, boiled down to the parts of the story of Paul
Revere (excluding the ride, which I guess we can understand as Redux being used to
update React components). 

Here it is without the analogy. The basic idea of Redux is this. You have a bunch of predefined actions. When one
of these actions is performed somewhere, a state change is triggered. There's
something, called a reducer, that knows how state should be changed in response to
the action. A reducer is just a function that maps from actions and states to
states. In other words, if your application is in state *S* and action *A* is
performed, the reducer determines what state the application goes into next. 

The reducer doesn't do this on its own. On its own it's just a function. The
reducer is used by being wrapped inside of what's called a 'store'. A store takes a
reducer and an initial state (that is, the state your app is in at the beginning)
and what it then does is use the reducer to keep the state updated as actions are
performed (in Redux, actions are 'dispatched'). 

When you hook Redux up with React, what you're primarily doing is giving your React
components access to the store. This allows them (a) to have their props stay in
sync with what's going on in the store, and (b) to dispatch actions in order to
update the store (which will in turn update those props on your React components
that are receiving their props from the Redux store). 

If I had to sum it up, I'd say that the core of Redux is the reducer (the set of
instructions). Everything else can be understood as ways of utilizing the reducer
(Paul Revere is just an instrument for implementing the instructions). And the
reducer is nothing more than a function from states and actions to states. 

Now, a little more detail and some code examples to show how these things are
implemented. 

# Reducers

Here's an example of a reducer, with two action types specified at the top: 

~~~
// file: reducer.js

const BY_LAND = 'BY_LAND';
const BY_SEA = 'BY_SEA';

const INITIAL_STATE = {
  byLand: false,
  bySea: false
};

function reducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    case 'BY_LAND':
      return Object.assign({}, state, {byLand: false, byLand: true});
    case 'BY_SEA':
      return Object.assign({}, state, {byLand: true, byLand: false});
    default:
      return state;
  }
}
~~~

`'BY_LAND'` and `'BY_SEA'` are action types. Obviously, they're just strings. The
reducer finds out that a particular action has occurred by having an action passed
in as an argument which has a `type` property which is just the string
corresponding to the appropriate action type (we'll see in the next section that
actions can have other properties as well, such as new data coming in). Also passed
in is the current application state. At first, we set it so that the default state
argument is the `INITIAL_STATE`, where here we set both properties to false to
indicate that we don't see them coming by land or by sea.  But, once we pass in an
action with type `BY_LAND`, the reducer produces a new state, and now our
application state now knows they're coming by land. If the action isn't one of our types, the state
doesn't change (that's what the `default` case is doing). 

Based on which action is passed in, and perhaps what the current state is, the
reducer determines what state is moved to next. Note that we're using
`Object.assign`, which we give an empty object as first argument. This copies
everything from the objects passed in as the rest of the arguments into the empty
object. As a result, the new state is a new object, and not a mutation of the old
state. This is a central aspect of Redux -- functional programming patterns and
immutable data. To achieve this, you can do something simple like `Object.assign`
or something more sophisticated like use the data structures provided by a library
like [Immutable.js](https://facebook.github.io/immutable-js/). 

At this point we can see why it makes sense to call reducers 'reducers'. They're
like `reduce` methods familiar from functional programming. A `reduce` function
takes something like an array and a function and goes through the array
element-by-element generating new values by passing the previous result and the
current element to the function, ultimately reducing the whole thing to a single
result value. If you imagine the actions that occur within your app over a period
of time, the reducer takes those actions and the initial state and reduces
it all to a single result value: the state at the end of the process. 


# Actions 

The strings we saw above -- `'BY_LAND'` and `'BY_SEA'` -- are action types, but
they aren't actions. Actions are objects that contain a `type` property (whose
value will be one of those strings and that's used in the switch statement in the
reducer function) and zero or more other properties. Those other properties can be
things like data that is relevant to calculating the new state -- for example,
something that's come in from user input or an API call. 

So, an action object might look like this: 

~~~
{
  type: BY_LAND,
  distance: '25 miles'
}
~~~

Now, we can tell our reducer that not only are they coming by land, but they're 25
miles away. 

Then the reducer can easily make use of this extra information so that the distance
is reflected in the new state: 

~~~
// file: reducer.js
const BY_LAND = 'BY_LAND';
const BY_SEA = 'BY_SEA';

const INITIAL_STATE = {
  byLand: false,
  byLand: false
};

function reducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    case 'BY_LAND':
      return Object.assign({}, state, {byLand: false, byLand: true, distance: action.distance});
    case 'BY_SEA':
      return Object.assign({}, state, {byLand: true, byLand: false, distance: action.distance});
    default:
      return state;
  }
}
~~~

So, actions: they say what action was performed, and maybe give some extra data.
In a real application, the action types can correspond to anything that might
happen in your app, `USER_CLICK`, `API_RETURN`, `LOGIN`, whatever makes sense. 

# Stores

Back to Paul Revere: you write your instructions down on a piece of paper: 'if you see them
coming by land, light one lantern. If you see them coming by sea, light two
lanterns'. You've got your action types -- 'BY_LAND' and 'BY_SEA' -- and your
reducer (that's the instructions taken as a whole). But a piece of paper with
instructions on it doesn't do much. So you give the instructions to someone
reliable and tell them to stand in a tower watching for the British. That person
with the instructions in a tower is your Redux store. Actions are dispatched to
them (in this case through vision) and the store changes state accordingly (in this
case, how many lanterns are lit, with an initial value of 0). 

Just like you need to pass your instructions to Paul Revere to get anything done,
to use your reducer to control state-changes of your application,
you give your reducer to a store, and let the store do the state-changing. Here's
the pattern (suppose our reducer is in the current namespace under `reducer`): 

~~~
import { createStore } from 'redux';

const store = createStore(reducer);
~~~

What `store` is pointing to after this is an object with a few useful methods on
it, including `dispatch`. `dispatch` is how actions are triggered -- that is,
to get the reducer to compute and send the application into a new state. For
example: 

~~~
// state => {byLand: true, bySea: false, distance: '30 miles'}
let action = {type: BY_SEA, distance: 88 miles};
store.dispatch(action);
// state => {byLand: false, bySea: true, distance: '88 miles'}
~~~

In this case, Paul Revere saw something (an 'action') that causes a change of mind. 

(Note: in practice one generally doesn't pass actions directly to stores, but
rather gets at actions via 'action creator' functions. However, this is supposed to
be a *very* basic Redux intro, and once you've got this all in your head action
creators are extremely straightforward.)

Like I said, there are other methods on `store` (like `getState()` -- I'll let you
guess what that one does), but `dispatch` is the heart of it all. It's how you get
the store to do what Redux stores are meant to do: update state. 

# Redux wrapped around React

(This assumes that you already know React to some degree.)

Finally, how do you get this together with React? There are two main moves you need
to make. First, you can connect up particular React components to your Redux store
by mapping the props of the component to all or some of the properties of the
store. Second, you pass the store itself to a special component that wraps your
whole app (more technically, that wraps the root component of your app). The first move syncs up
particular components to Redux (which makes them 'smart' components); the second
allows the store to be available to the function that actually turns your
components into smart (that is, connected to Redux) components. Both make use of the
[react-redux](https://github.com/rackt/react-redux) library, which is designed to
ease connection of React and Redux.

So, suppose we want to create a Distance component and connect it up with Redux
(the first move). Here's how we can do that: 

~~~
import React, { Component } from 'react';
import { connect }          from 'react-redux';

class Distance extends Component {
  render() {
    return (
      <div>
        {this.props.distance}
      </div>  
    )
  }
}

function mapStateToProps(state) {
  return {
    distance: state.distance
  }
}

const ConnectedTransport = connect(mapStateToProps)(Transport);
~~~

First, we import React and Component from React and then the connect function from
React-Redux. Then, we make our basic component, which should display the distance
we're traveling. Then we write a function that maps the Redux state to props of the
component. We can map just some of the state (as we've done here) or the whole
state (by just returning `state`), which will make everything in the state
available through `this.props`. And then finally we call `connect`. This gives us a
new 'smart' component -- that is, a component that will stay in sync with the
state, because when the state changes, `mapStateToProps` will be used to
appropriately update the component's props. Instead of exporting the 'dumb'
component, now we can export the smart component, by adding this at the end of what
we've written: 

~~~
export ConnectedTransport;
~~~

Note that it is generally recommended that you only make high-level components
smart components, and give any props that lower components need from the Redux
store to them by having their parents pass them in. 

Second, we give the store to our app at the top level. Suppose we've got an `App`
component that is the root component of our React app. Here's what we do: 

~~~
import React           from 'react';
import ReactDom        from 'react-dom';
import { createStore } from 'redux';
import { Provider }    from 'react-redux';
import reducer         from 'reducer';
import App             from 'App';

const store = createStore(reducer);

ReactDom.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('app');
);
~~~

If you're familiar with React, you can see that `Provider` is a component, but to
this component we give our Redux store, which is what makes the particular instance
of our store available to the `connect()` function we saw earlier, which connects
particular components to the Redux Store. It also allows us to connect components
to actions so that they can dispatch actions which will in turn update the Redux
store (however, this pattern involves action creators, which we haven't covered). 

There's of course a whole lot more to using Redux with React, and a whole lot more
to Redux itself, but this about does it for the conceptual underpinnings, and
hopefully makes figuring out the rest of it a bit easier going. 
