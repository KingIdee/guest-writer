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
MobX is one of the popular state management libraries out there. It is frequently used with React also. In this article, you will learn how to manage your states with MobX in a React app. We will build a sample along to show the practical implementaions.

## Prerequisites
Before diving into this article fully, it is expected that you have a prior knowledge of React already. If you need an explicit tutorial to get you started, you can find one [here](https://auth0.com/blog/react-tutorial-building-and-securing-your-first-app/).You also need to have [Node](https://nodejs.org/en/) and [NPM](https://www.npmjs.com/) installed on your machine.


## State Management in React

Before understanding the concept of state management, you have to understand what a state is. A state is the data layer for your application. Explaining further, we can say a state is an object that contains what is to displayed on your application. For instance, if you want to display a list of items on your app, your state will contain the objects(items) you want to display. They influence how a component behaves and how itis rendered. Yes! It is as simple as that. 

State management therefore means monitoring and managing data(state) of our app. Definitely, almost all production-ready apps will have a state (data layer) and so managing states has become one of the most important part of any modern application today. 

Basically, there are three alternatives towards managing states in React. They are: 

- Redux
- React Context API and 
- MobX