---
layout: post
title: "Managing the State of React Apps with MobX"
description: "Learn how to manage states with MobX, a state management library"
date: "2018-09-06 08:30"
author:
  name: "Idorenyin Obong"
  url: "https://twitter.com/kingidee/"
  mail: "idee4ril@gmail.com"
  avatar: "https://twitter.com/kingidee/profile_image?size=original"
related:
- 2017-11-15-an-example-of-all-possible-elements
---

**TL;DR:** 
MobX is one of the popular state management libraries out there. It is frequently used with React also. In this article, you will learn how to manage the state of your React app with MobX.

## Prerequisites
Before diving into this article fully, it is expected that you have a prior knowledge of React already. If you need an explicit tutorial to get you started, you can find one [here](https://auth0.com/blog/react-tutorial-building-and-securing-your-first-app/). You also need to have [Node](https://nodejs.org/en/) and [NPM](https://www.npmjs.com/) installed on your machine.


## State Management in React

Before understanding the concept of state management, you have to understand what a state is. A state in this context, is the data layer of your application. Explaining further, you can say a state is an object that contains what is to displayed on your application. For instance, if you want to display a list of items on your app, your state will contain those items you want to display. States influence how a component behaves and how it is rendered. Yes! It is as simple as that. 

State management therefore means monitoring and managing the data(state) of your app. Definitely, almost all production-ready apps will have a state (data layer) and so, managing states has become one of the most important part of any modern app today. Basically, there are three alternatives towards managing states in React. They are Redux, React Context API and MobX. Let us take a brief look at the first two alternatives.

### Redux 
[Redux](https://redux.js.org/) is the most popular state management solution for React apps. Redux strictly abides to the 'single source of truth' principle. With this, the state is kept in one location (the store) and made a read-only entity. Read-only you say, how then do I handle new data? Redux revovles around three concepts: the store, the reducer, and actions. The store holds the state, the action represents the intent to change the state, and the reducer specify how the application's state changes in response to actions. The only way to change the state is to emit an action. The reducer listens to a set of actions and returns a new state. 

> The reducer does not mutuate the current state. It copies the current state, modiefies it based on actions emiited and returns a new state. This way, your state is not mutated in an unorderly manner, therby causing irregular bugs.

The reducer is seen as the most important of these concepts. You can check this [practical tutorial on Redux](https://auth0.com/blog/redux-practical-tutorial/) for further in-depth explanations on how Redux works.

### React Context API
The [React Context API]() is another alternative for state management in your React app. This is not a library like the earlier mentioned alternative. Rather, this is a framework in built solution. Actually, this API is not something new, it had existed in React a long while ago. It only reached a mature state when React 16.3 was released. In fact, Redux uses this API under the hood. This API provides a way to pass data down a React component tree without explictly passing it through all the child components. This API revolves around two components, the `Provider` - used in a component located in a higher hierarchy of the  `Component` tree, the `Consumer` component - used by a `Component` down the  hierarchy to consume data. You can read more about it at[this blog post](https://auth0.com/blog/react-context-api-managing-state-with-ease/) to learn more.

In the next section, you will have a more detailed look at the third alternative we have at our disposal, MobX.

## MobX Introduction

[MobX](https://mobx.js.org/) is another state management library available for React apps. It uses a more reactive process and it is slowly gaining popularity in the community. MobX is not just a library for React apps alone, it is also suitable for use with other javaScript libraries and frameworks that power the frontend of web apps. MobX is sponsored by reputable companies such as [Algolia](https://www.algolia.com/), [Coinbase](https://www.coinbase.com), etc. MobX hit 16,719 stars on [GitHub](https://github.com/mobxjs/mobx) at the time of writing. It is definitely becoming a solid choice state management solutions for React applications. To get up to speed, you can visit its official [documentation](https://mobx.js.org/).

Next, you will learn about the core concepts of MobX.

### Observable State on MobX

Observable state is one of the main concepts of MobX. The idea behind this concept is to make an object able to emit new changes on them to the observers. You can achieve this with the `@observable` decorator. Lets say you have a variable named `counter` that you expect to change with time. If you intend get updates as this variable changes, you can make it observable like so:

```javascript
@observable counter = 0
```

An alternative way of doing this is like this:

```javascript
decorate(ClassName, {
  counter: observable
})
```
Where `ClassName` is the name of the class where the `counter` object is. This decorator can be used on instance fields and property getters. 

### Computed Values on MobX

Computed values are another set of important concept of MobX. They are represented with the `@computed` decorator. Computed values work in hand with observable states. With computed values, you can automatically derive values. Say you have a snippet like this:

```javascript
class ClassName {
  @observable test = 0;
  @computed get computedTest() {
    return this.test + 100;
  }
}
```

In this snippet, if the value of `test` changes, the `computedTest` method is equally computed and the return value is updated automatically. So, with computed values, MobX can automatically compute values when any observable property the method/object changes. Computed values are derived from observables.


### Reactions on MobX

Reactions are very much similar to computed values. The difference here is that instead of computing and returning a value, a reaction simply triggers a side effect, more like it performs a side operation. Reactions occur as a result of changes on observables. Reactions  could affect the UI or they could be background actions. Mobx provides 3 main types of reaction functions when, autorun and reaction. Let us look at what these functions do:

- `when` : accepts two functions as parameters, the predicate and effect. It runs and observes the first function (the predicate) until it returns true, and then runs the effect function. After this, it disposes, and stops reacting observed property. Here is an example of how this function works:

```javascript
when(
	// predicate
	() => this.isEnabled,
	// effect
	() => this.exit()
);
```
This function works hand in hand with the observables and coputed values such that the `isEnabled` function located in the component class could be marked with `@computed` decorator so that its value would be computed automatically. And, of course, for it to be marked `@computed`, it is definitely listening to an observable. You might have a different use

The next reaction function is the autorun function. Unlike the `when` function, the `autorun` function takes in one function and keeps running it untill it is manually disposed. Here is how an `autorun` function can be used:

```javascript
@observable age = 10
const dispose = autorun(() => {
  console.log("My age is: ", age.get())
})
```
With this in place, anytime the variable `age` changes, the `autorun` function stored in `dispose` logs it out. This function is disposed once you call:

```javascript
dispose();
```

The `reaction` function mandatorily accepts two functions, (data function and side effect function) and an optional third argument. It is like the `autorun` function but gives you more control on which observables to track. Here, the data function is tracked and returns data that is used in side effect function. 
Whereas, an `autorun` function reacts to everything used in its function, the `reaction` function reacts to 

Here is a simple use case:

```javascript
const todos = observable([
  {
    title: "Read Auth0 Blog",
    done: true,
  },
  {
    title: "Write MobX article",
    done: false
  }
]); 

const reactionSample = reaction(
    () => todos.map(todo => todo.title),
    titles => console.log("Reaction: ", titles.join(", "))
);
```

This reaction function reacts to changes of the length and title.

Another reaction function available for React developers is the `observer` function. This is not provided by the main `mobx` package, but instead, provided by the `mobx-react` package. You can use it on a component by just adding the `@observer` decorator in front of it like so:

```javascript
@observer class ClassName {
  // [...]
}
```

With this reaction function,  if an object tagged with the `@observable` decorator is used in the `render` method of the component and that property changes, the component is automatically re-rendered. The `observer` function uses the `autorun` function internally.

### Actions on MobX

Actions are anything that modify the state. You can mark your actions using the `@action` decorator. It is advised to use the `@action` on any function that modifies observables or has side effects. A simple example is this:

```javascript
@observable variable = 0;

@action setVariable(newVariable){
  this.variable = newVariable;
}

```

## MobX and React in Practice

In this post, you will build a simple user review dashboard. In the review dashboard, a user will enter a review using an input field, select a rating from a dropdown and finally submit the review. The dashboard will show the total number of reviews, the average star rating, and a list of all the reviews. In this sample, MobX will be used to manage certain operations like updating the reviews in realtime on the dashboard, calculating the total number of reviews submitted and lastly, obtaining the average star rating. Once you are done, your app will look similar to this:

//Image of app missing

### Creating a new React app

You need the `create-react-app` CLI tool to quickly bootstrap your React app without the hassle of build configurations. You can install it by running:

```bash
npm install -g create-react-app
```

After installation, you will have access to the `create-react-app` command. Go ahead to create your app by running:

```bash
npx create-react-app react-mobx-tutorial
```

### Installing Dependencies

After creating your app, the next obvious step is to install the required dependencies. You need two dependencies, the `mobx-react` dependency to add MobX to your app and the `react-star-rating-component` dependency to easily implement a rating bar in the app. You can install them like so:

```bash
# move into app directory
cd react-mobx-tutorial

# install deps
npm install mobx mobx-react react-star-rating-component --save 
```

### Creating a Store with MobX

The first thing to add in your app is a store that will be powered by MobX. This will ensure that the app reads from and writes to a global state object instead of its own component state. To set this up, create a new folder named `Store` within the `src` folder and create a new file called `Store.js` inside of it. Set up the file like so:

```javascript

// ./src/Store/Store.js
class Store {
  reviewList = [
    {review: "This is a nice article", stars: 2},
    {review: "A lovely review", stars: 4},
  ];

  addReview(e) {
    this.reviewList.push(e);
  }

  get reviewCount() {
    return this.reviewList.length;
  }

  get averageScore() {
    let avr = 0;
    this.reviewList.map(e => avr += e.stars);
    return Math.round(avr / this.reviewList.length * 100) / 100;
  }
}

export default Store;
```

In this store, there is a `reviewList` containing some items already. This is the list your whole app will feed on. The store has some other custom methods created like `addReview()` - to add a new item to the list, getter methods `averageScore()` and `reviewCount()` to get the average score and size of the list respectively.

Next, you will expose these methods as observables so that other parts of your application can make use of it. MobX a set of decorators that defines how observable properties will behave. You have to declare them using the `decorate` keyword. Add this to your `App.js` file:

```javascript
import {decorate, observable, action, computed} from 'mobx';

decorate(Store, {
  reviewList: observable,
  clearReviews: action,
  addReview: action,
  averageScore: computed,
  reviewCount: computed
});
```

From this snippet, the decorators are imported from the mobx package and assigned to the various methods to be exposed. 

### Updating the Store on MobX

Next, you will create a new component - the form component that will collect the user's review response and update the contents already specified in the MobX store. For proper organisation, create a folder just for your components. Create a new folder named `components` within the `src` folder. After this, go ahead to create a new file called `Form.js` inside of it. Set up the file like so:

```javascript
// ./src/components/Form.js

import React, {Component} from 'react';

export default class Form extends Component {

  submitReview = (e) => {
    e.preventDefault();
    const review = this.review.value;
    const stars = Number(this.stars.value);
    this.props.store.addReview({review, stars})
  };

  render() {
    return (
      <div className="formSection">
        <div className="form-group">
          <p>Submit a Review</p>
        </div>
        <form onSubmit={this.submitReview}>
          <div className="row">
            <div className="col-md-4">
              <div className="form-group">
                <input type="text" name="review" ref={node => {
                  this.review = node;
                }} id="review" placeholder="Write a review" className="form-control"/>
              </div>
            </div>

            <div className="col-md-4">
              <div className="form-group">
                <select name="stars" id="stars" className="form-control" ref={node => {
                  this.stars = node;
                }}>
                  <option value="1">1 Star</option>
                  <option value="2">2 Star</option>
                  <option value="3">3 Star</option>
                  <option value="4">4 Star</option>
                  <option value="5">5 Star</option>
                </select>
              </div>
            </div>

            <div className="col-md-4">
              <div className="form-group">
                <button className="btn btn-success" type="submit">SUBMIT REVIEW</button>
              </div>
            </div>
          </div>
        </form>
      </div>
    )
  }
}
```

In this snippet, you created a form that contains an input field, a dropdown, and a button. On submission of the form, the review is added to the store via the `submitReview` method. 

### Reacting to Changes on MobX

Once the form has been submitted and the store has the updated contents, you need to immediately display the updated data to your users. For this purpose, you need a component that will display the average number of stars from reviews given. Navigate to the `components` folder and create a new file named `Dashboard.js`, set it up like so:  

```javascript
// ./src/components/Dashboard.js

import React, {Component} from 'react';
import {observer} from 'mobx-react'


class Dashboard extends Component {
  render() {
    const {store} = this.props;
    return (
      <div className="dashboardSection">
        <div className="row">
          <div className="col-md-6">
            <div className="card text-white bg-primary mb-6">
              <div className="card-body">
                <div className="row">
                  <div className="col-md-6">
                    <i className="fa fa-comments fa-5x" />
                  </div>
                  <div className="col-md-6 text-right">
                    <p id="reviewCount">{store.reviewCount}</p>
                    <p className="announcement-text">Reviews</p>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div className="col-md-6">
            <div className="card text-white bg-success mb-6">
              <div className="card-body">
                <div className="row">
                  <div className="col-md-6">
                    <i className="fa fa-star fa-5x" />
                  </div>
                  <div className="col-md-6 text-right">
                    <p id="averageScores">{store.averageScore}</p>
                    <p className="announcement-text">Average Scores</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    )
  }
}

Dashboard = observer(Dashboard);
export default Dashboard;
```
This component displays the total count of reviews and the average number of stars. Another important thing to note, is the use of the `observer()` function. This turns the component into a reactive component and smart. With this in place, any changes made to any content in store within the component above will be re-rendered thereby updating the user interface with new contents in realtime.

You won't just stop at giving the count of reviews and average rating, you need to display the comments made too. Create another file named `Reviews.js` and set it up like so:

```javascript
import React, {Component} from 'react';
import {observer} from 'mobx-react';
import StarRatingComponent from 'react-star-rating-component';

const List = (props) => {
  return (
    <li className="list-group-item">
      <div className="float-left">{props.data.review}</div>
      <div className="float-right">
        <StarRatingComponent name="reviewRate" starCount={props.data.stars}/>
      </div>
    </li>
  )
};

class Reviews extends Component {
  render() {
    const {store} = this.props;
    return (
      <div className="reviewsWrapper">
        <div className="row">
          <div className="col-12">
            <div className="card">
              <div className="card-header">
                <i className="fa fa-comments" /> Reviews
              </div>
              <ul className="list-group list-group-flush">
                {store.reviewList.map((e, i) =>
                  <List
                    key={i}
                    data={e}
                  />
                )}
              </ul>
            </div>
          </div>
        </div>
      </div>
    )
  }
}

Reviews = observer(Reviews);

export default Reviews;
```

In the snippet above, the `StarRatingComponent` installed earlier is used to display the number of stars selected by the user during review. Also, a `List` component is created here. This component is what will be rendered as each list item when you iterate over the total list of submitted reviews. The `Reviews` is also wrapped with an `observer()` function to make the component recieve and display changes in the MobX store as they come.


### Wrapping Up your MobX App

To wrap up, you will modify your `App.css` file. Open the file and replace the contents like so:

```css
.formSection {
  margin-top: 30px;
}

.formSection p {
  font-weight: bold;
  font-size: 20px;
}

.dashboardSection {
  margin-top: 50px;
}

.reviewsWrapper {
  margin-top: 50px;
}
```

Here, you declared classes with properties 

Next, you will finialise your `App.js` file. Open your `App.js` file and add this:

```javascript
import React, {Component} from 'react';
import './App.css';
import Form from './components/Form';
import Dashboard from './components/Dashboard';
import Reviews from './components/Reviews';
import Store from './Store/Store';

const reviewStore = new Store();

class App extends Component {
  render() {
    return (
      <div className="App">
        <Form store={reviewStore}/>
        <Dashboard store={reviewStore}/>
        <Reviews store={reviewStore}/>
      </div>
    );
  }
}

export default App;
```

Here in this snippet, the earlier created components are imported in order to render the complete UI. Lastly, the instance of the store created earlier - `reviewStore` is passed as a props to all the components.

Finally, update your `index.html` file as follows. First, add the following CDNs for font awesome and Bootstrap: 

```html
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
    crossorigin="anonymous">
<link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">

```
Secondly, change the text within the `<title>` tag like so:

```html
<title> React and MobX </title>
```

Now, navigate back to the project folder and run the command below to start the app:

```javascript
npm start
```

Visit the application on http://localhost:3000/


## Conclusion

In this post, you have learnt about state management in React apps. You learnt about the alternatives for managing state in React apps, most especially, MobX. You were able to build an app to show the most important concepts in MobX. I strongly believe in using the right tools depending on the project’s requirement. MobX  might not be the most popular, but by far it is very easy to start with and seamless to integrate into a new or an existing application. I do hope that you found this tutorial worthwhile. 