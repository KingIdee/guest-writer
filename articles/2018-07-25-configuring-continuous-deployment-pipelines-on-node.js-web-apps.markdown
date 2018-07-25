---
layout: post
title: "Configuring Continuous Deployment Pipelines on Node.js Web Apps"
description: "This article will put your through the process of deploying a simple Node.js application on Now.sh. You will use GitHub as your git hosting repository, and Travis CI as your continuous deployment server."
date: "2018-07-25 08:30"
author:
  name: "Idorenyin Obong"
  url: "kingidee"
  mail: "idee4ril@gmail.com"
  avatar: "https://twitter.com/kingidee/profile_image?size=original"
related:
- 2017-11-15-an-example-of-all-possible-elements
---

## TL;DR: 
In this article, you will learn how to configure a continuous deployment pipeline for a Node.js web application. For demonstration purposes, you will use Now.sh, GitHub, and TravisCI to automate the pipeline. However, the strategy that you will learn here can be used with other solutions as well (like BitBucket, AWS, and CircleCI, for example).


## Continuous Deployment Overview

Continuous deployment popularly know as CD is a modern software engineering approach that has to do with automating the release of softwares. Instead of the usual manual method of pushing out a software to production, continuous deployment aims to ease and automate this process with the use of pipelines. In continuous deployment, an update to the source code means an update to the production server too provided all tests are passed. Continuous deployment is often mistaken with continuous integration and continuous delivery. For you to properly get a hang of this concept, let us distinguish the other two concepts.

Continuous Integration(CI) - In continuous integration, when a new code is checked in, a build is generated and tested. The aim is to test every new code to be sure that it doesn’t break the software as a whole. This will require writing test for every update that is pushed. The importance of CI is to ensure a stable codebase at all times especially when there are multiple developers in a team. With this, bugs are discovered easily when the automated tests fail.

Continuous Delivery - Continuous delivery moves a step ahead of CI. After testing, the release process is also automated. The aim is to generate a releasable build i.e a build that is stable enough to go into production. This helps to reduce the hassle of preparing for release. In continuous delivery, since there are regular releases, there is a faster feedback.

The major difference between continuous delivery and continuous deployment is the way releases are done.  One is manual, while the other is automated. With continuous delivery your software is always at a state where it can be pushed out to production manually.  Whereas, with continuous deployment, any stable working version of the software is pushed to production immediately. Continuous deployment needs continuous delivery, but the reverse is not applicable.

In all of these, you need a repository where your code will reside and a CI server to monitor the repository where the code is checked into. When the server notices an update in the code, it triggers the pipeline. A pipeline in this context is a script/file that contains commands and tasks to be executed on the project. When typically setting up your CI server, you setup your pipeline alongside it. Some common CI servers include Travis CI, Jenkins, TeamCity, etc.

In this post, you will learn how to setup a CI server together with a GitHub repository to show continuous deployments. Continuous deployments are important for the following reasons: better integration for large teams, faster and easier releases, faster feedback, increased development productivity as a whole, etc.

## Preparing a Node.js App for Continuous Deployment

You will build a simple hello word app with Node.js. Ensure that you have Node.js installed on your machine before moving ahead. However, if you don’t have it yet, follow this [link](https://nodejs.org/en/download/) to install it. 

### Scaffolding a Node.js Web App

Here, you will setup the structure of your Node.js application. For easy setup, you can run the `npm init` command in the project directory. This will guide you through generating a `package.json` file for your Node.js app. The `package.json` file contains basic information about your app coupled with the name of the dependencies used. If you run the command, you should see something like this on your screen,
 

![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532383163132_Screen+Shot+2018-07-23+at+10.59.02+PM.png)


Type your package name, lets say — **hello-world** and press enter afterwards. You will be required to fill in other fields. Your responses can look like this after inputing all fields:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532383655202_Screen+Shot+2018-07-23+at+11.06.57+PM.png)


After you enter the last field(license), the `package.json` file will be generated for you. You will be asked to confirm the details of your app, simply type **yes** in the terminal. If you open the directory for your app, you will see a `package.json` file. The file will look like this:


    {
      "name": "hello-world",
      "version": "1.0.0",
      "description": "A Node.js app to show continuous deployment",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [
        "Node.js",
        "Express",
        "TravisCI"
      ],
      "author": "KingIdee",
      "license": "ISC"
    }


### Installing Dependencies

Haven created the `package.json` file, you will need to install the dependencies needed to build your project. You particularly need two dependencies for this one - `express` and `body-parser`. You can install all these dependencies at once by running this command:


    npm install express body-parser --save

Once the installation is complete you should see a `node_modules` folder. Additionally, your `package.json` file will contain the dependencies installed and their versions.

### Creating a Web Page

During the setup of the app, the `index.js` file was declared as the entry point of the app. Now, you need to create the file. Still in the app directory, run this command to create the file:


    touch index.js

Next, you have to create the html file. It is usually a good practice to create a folder for your views. So, you would do same here. You can create the folder and file by running these commands in the directory of your app:
 

    mkdir pages

This command creates a new directory named `pages`.


    touch pages/index.html

While this command creates a html file named `index.html` in pages folder. Now, you have to serve the html file when the user visits the `URL` of your app. Open the `index``.js` file and set it up like so: 

```node
    // import dependencies
    const express = require('express');
    const bodyParser = require('body-parser');
    var path = require('path');
    
    // initialise express
    const app = express();
    
    // root endpoint
    app.get('/', (req, res) => {
        res.sendFile(path.join(__dirname + '/pages/index.html'));
    });
    
    // select the port in which node server will run
    const port = 5000;
    
    // listen to the selected port and log a message once connection is established
    app.listen(port, () => console.log(`Server is running on port ${port}`));
```    

This snippet contains all the server logic required for this app. Just one endpoint is declared which loads the `index.html` page. The app will run on port `5000`. Now, you need to add some content to your `index.html` file. Open the file and paste this:

```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Title Page</title>
        <!-- Bootstrap CSS -->
        <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
      <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.3/html5shiv.js"></script>
    <script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
    </head>
    <body>
        <h1 class="text-center">Hello World</h1>
    
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    </body>
    </html>
```

This is a basic HTML code with Bootstrap and Jquery referenced via CDN (Content Delivery Network). It contains a **hello world** text in a `h1` tag to make the text a heading. You can run your server now with this command: 


    node index.js

If you visit `http://localhost:5000` , you should see something like this:


