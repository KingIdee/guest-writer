---
layout: post
title: "Configuring Continuous Deployment Pipelines for Web Apps"
description: "Learn how to configure a Continuous Deployment pipeline for your web applications."
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
In this article, you will learn how to configure a continuous deployment pipeline for web apps. For demonstration purposes, you will use Now.sh, GitHub, and TravisCI to automate the pipeline. Actually, the strategy can be used with other programming languages (e.g. Python, Java, .NET Core) and tools (like BitBucket, AWS, and Circle CI).


## Continuous Deployment Overview

[Continuous deployment](https://www.scaledagileframework.com/continuous-deployment/) (popularly known as CD) is a modern software engineering approach that has to do with automating the release of software. Instead of the usual manual method of pushing out a software to production, continuous deployment aims to ease and automate this process with the use of pipelines. In continuous deployment, an update to the source code means an update to the production server too provided all tests are passed. Continuous deployment is often mistaken with continuous integration and continuous delivery. For you to properly get a hang of this concept, let us distinguish the other two concepts.

[Continuous Integration](https://www.atlassian.com/continuous-delivery/continuous-integration-intro) (CI) - In continuous integration, when a new code is checked in, a build is generated and tested. The aim is to test every new code to be sure that it doesn’t break the software as a whole. This will require writing test for every update that is pushed. The importance of CI is to ensure a stable codebase at all times especially when there are multiple developers in a team. With this, bugs are discovered easily when the automated tests fail.

[Continuous Delivery](https://aws.amazon.com/devops/continuous-delivery/) - Continuous delivery moves a step ahead of CI. After testing, the release process is also automated. The aim is to generate a releasable build (i.e a build that is stable enough to go into production). This helps to reduce the hassle of preparing for release. In continuous delivery, since there are regular releases, there is a faster feedback.

The major difference between continuous delivery and continuous deployment is the way releases are done.  One is manual, while the other is automated. With continuous delivery your software is always at a state where it can be pushed out to production manually.  Whereas, with continuous deployment, any stable working version of the software is pushed to production immediately. Continuous deployment needs continuous delivery, but the reverse is not applicable.

In all of these, you need a repository where your code will reside and a CI server to monitor the repository where the code is checked into. When the server notices an update in the code, it triggers the pipeline. A pipeline in this context is a script/file that contains commands and tasks to be executed on the project. When typically setting up your CI server, you setup your pipeline alongside it. Some common CI servers include Travis CI, Jenkins, TeamCity, etc.

In this post, you will learn how to setup a CI server together with a GitHub repository to show continuous deployments. Continuous deployments are important for the following reasons: better integration for large teams, faster and easier releases, faster feedback, increased development productivity as a whole, etc.

## Preparing a Node.js App for Continuous Deployment

You will build a simple hello world app with Node.js. Ensure that you have Node.js installed on your machine before moving ahead. However, if you don’t have it yet, follow this [link](https://nodejs.org/en/download/) to install it. 

### Scaffolding a Node.js Web App

Here, you will setup the structure of your Node.js application. For easy setup, you can run the `npm init` command in the project directory. This will guide you through generating a `package.json` file for your Node.js app. The `package.json` file contains basic information about your app coupled with the name of the dependencies used. If you run the command, you should see something like this on your screen,
 

![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532383163132_Screen+Shot+2018-07-23+at+10.59.02+PM.png)


Type your package name, lets say — **hello-world** and press enter afterwards. You will be required to fill in other fields. Your responses can look like this after inputing all fields:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532383655202_Screen+Shot+2018-07-23+at+11.06.57+PM.png)


After you enter the last field(license), the `package.json` file will be generated for you. You will be asked to confirm the details of your app, simply type **yes** in the terminal. If you open the directory for your app, you will see a `package.json` file. The file will look like this:

```json
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
```

### Installing Dependencies

Haven created the `package.json` file, you will need to install the dependencies needed to build your project. You particularly need two dependencies for this one - `express` and `body-parser`. You can install all these dependencies at once by running this command:

```
npm install express body-parser --save
```

Once the installation is complete you should see a `node_modules` folder. Additionally, your `package.json` file will contain the dependencies installed and their versions.

### Creating a Web Page

During the setup of the app, NPM declared the `index.js` file as the entry point of the app. Now, you need to create the file. Still in the app directory, run this command to create the file:

```
touch index.js
```

Next, you have to create the html file. It is usually a good practice to create a folder for your views. So, you would do same here. You can create the folder and file by running these commands in the directory of your app:
 
```
mkdir pages
```

This command creates a new directory named `pages`.

```
touch pages/index.html
```

While this command creates a html file named `index.html` in the pages folder. Now, you have to serve the html file when the user visits the `URL` of your app. Open the `index.js` file and set it up like so: 

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

```
node index.js
```

If you visit `http://localhost:5000` , you should see something like this:

![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532524887723_Screen+Shot+2018-07-25+at+2.21.11+PM.png)


## Introducing GitHub

Earlier in this article, I mentioned the need for repository to demonstrate this subject matter. A repository is simply a place where files are stored. You will need to store your project on a remote (online) repository. The term repository is commonly used when talking about version control systems. Version control systems are systems that keep record of files and changes on them. 

Git is one of the most popular version control systems out there. And so, a web service that can host a git repository is what we will opt for. There are many available out there but we will use the most popular of them, GitHub. [GitHub](https://www.github.com) is a web service where git repositories are hosted. Actually, they offer more than this and you can read more about GitHub [here](https://github.com/features).

### Creating a GitHub Account

If you don’t have an account with GitHub, visit the [website](https://www.github.com) and create an account If you have an account already, login to your profile. Creating an account requires a unique username, email with any password of your choice. 


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532375325519_Screen+Shot+2018-07-23+at+8.48.26+PM.png)


After registration, you will be required to verify your account through your email address to gain full access.

### Creating a GitHub Repository

GitHub offers public and private repositories. For this demo, you only need a public repository. To create a new repository, open your profile, click the plus button and select **New repository**. Your profile should look like this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532387287747_Screen+Shot+2018-07-24+at+12.07.29+AM.png)


After selecting a new repository, you will be presented with a form like this

![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532387443030_Screen+Shot+2018-07-24+at+12.10.07+AM.png)


Fill it with the name of your repository and the description. Click the **Create repository button** after you are done. When your repository is created, you are given a `URL` that resembles this `https://github.com/KingIdee/hello-world.git` . Copy it to your clipboard, you will need it soon.

### Pushing your Node.js Project to GitHub

Haven setup your Node.js app and installed dependencies, it is time to push your project online to GitHub. Still in your root directory run the following commands: 

```
git init
```

This initialises git in the project directory by creating a `.git` folder which is most times invisible.

```
git remote add origin REPO_URL
```

This command connects the local folder to the remote (online) repository.


> Replace **REPO_URL** with your own repository `URL` which you copied earlier.

Sometimes, there are particular files you need to ignore when pushing a project to the repository. For this project and most Node projects you need to ignore the `node_modules` folder. Create a `.gitignore` file in the directory like so:

```
touch .gitignore
```

Open the file and paste this line: 

```
/node_modules
```

> You can add other files you want to ignore as well

Next, you need to commit the changes made to the project folder and push to your remote repository. You can do so by running the following  commands:

```
git add -A
```

This command adds all files affected by changes to git except the ones explicitly listed in `.gitignore` file.

```
git commit -m "first commit"
```

This command adds a message to the added/changed files to note what changes that occurred. 

```
git push -u origin master
```

This command pushes the local changes made to the project to the remote repository and sets up the local project to track remote changes.

If everything works fine you should see your project online when you visit the repository you created. 

## Introducing Now.sh

[Now](http://now.sh) is a Platform as a Service (Paas) which allows you deploy your Node.js or Docker powered websites to the cloud with ease. Now aims to make continuous deployments easier for developers. Naturally, deploying websites built with Node.js require a sound knowledge of  server configurations and management together with a good command of the terminal. With Now, you can focus more on your app logic and worry less on deployments.

Some of the amazing features of Now include:

- Free unique URL -  for every deployment made, there is a unique `URL`  generated usually in the form `<appname>-<random string>.now.sh` E.g `helloworld-hddnhdvhsd.now.sh`
- Process logging - every process from the point of running the command to the point of starting the server for the deployed app is logged on the screen and can be viewed by clicking on any of the deployment instance link found on your dashboard.
- SSL certificate management - Now uses [Let's Encrypt](https://letsencrypt.org/) to provide your deployments with SSL at no cost. Etc.

Now also offers you the option of purchasing a custom domain. You can read more about Now in the official [docs](https://zeit.co/docs) page. 

### Creating a Now.sh Account

Go to [Now.sh](https://zeit.co/signup) and create an account. You can easily use the **Signup with GitHub** option.

![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532614163877_Screen+Shot+2018-07-26+at+3.08.44+PM.png)



If you chose the Signup with GitHub option, a verification mail will be sent to your email address. After verifying your email, you can now [login with GitHub](https://zeit.co/login). If you are a new user, after logging in, your dashboard should look like this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532391681715_Screen+Shot+2018-07-24+at+1.20.18+AM.png)



### Obtaining a now token

Once registration is complete login with GitHub, click on **settings** and select **Tokens** tab. 


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532392002811_Screen+Shot+2018-07-24+at+1.26.14+AM.png)


Click on **copy** for any tokens listed there and your token will be copied to the clipboard for you. You can also choose to create a new token, you can do this by entering a name in the **Create a new token** hint input field and hit enter. You will use this token for deployments. 

## Introducing Travis CI

[Travis CI](https://travis-ci.org/) is a continuous integration server. CI servers are used to monitor repositories when there is a change and trigger a pre-configured process on the repository. Travis CI particularly supports only projects on GitHub. The project is usually configured by adding a pipeline file named -  `.travis.yml` file to the root directory of the repository.  

### Creating a Travis Account

To move ahead, you have to signup on [Travis CI](https://travis-ci.org/). Travis CI requires only GitHub signup. Since you 
have a GitHub account, open the website and signup.

### Configuring Travis CI for your GitHub Repo

After a successful signup, you can see a list of your public repositories on your profile [here](https://travis-ci.org/) while your private repositories are found [here](http://travis-ci.com.). Travis stores public and private repos in separate domains. Go to your public repository list. It should look like this


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532392825500_Screen+Shot+2018-07-24+at+1.39.51+AM.png)


Select the repository you created earlier and activate it. 


![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532393198638_Screen+Shot+2018-07-24+at+1.45.28+AM.png)


### Configuring Travis in your Application

Go back to the project folder, create a `.travis.yml` file and paste the following code:

```
language: node_js
node_js:
- node
cache:
  directories:
  - node_modules
before_deploy: npm install now --no-save # installs now cli
deploy:
- provider: script
  # deploys the application to now.sh if all tests pass
  script: now --public --token $NOW_TOKEN && now alias --token $NOW_TOKEN
  skip_cleanup: true
  on:
    master: true
- provider: script
# starts the node server with the entry point script server.js
  script: npm run server
  skip_cleanup: true
```


In `language:` the technology used for the app is specified, in this case -  Node.js. The `before_deploy:` section specifies the command to be executed before deployment. Here, the Now CLI is installed. The first `script:` in the `deploy` section deploys the project to Now.sh using Now CLI, the `on:` tells Travis which branch to work with and the second `script` starts the node application.

Next, open your `package.json` file too and add this in the file:
 
```json
{
 [...]
 "scripts": {
   "start": "npm run server",
   "server": "node index.js"
  }
 }
```
 
The `npm run server` which was defined in `.travis.yml` didn’t exist in `package.json` file until you just added it now.  

### Securing Now.sh token with Travis CLI

Security is important when deploying your application as secret keys may be involved. For instance, in your case, the now token is a secret key. Setting the keys as environment variables in Travis CI may seem as the run to approach. Unfortunately, this is not entirely secure. To secure your keys properly, you will need to use the Travis Command Line Interface. To use this, you need to install  ruby and `gem` on your machine. Follow this [documentation](https://github.com/rubygems/rubygems#installation) to achieve that. After you have done that, you can now install Travis CLI using this command:

```
gem install travis
```

Congrats! You now have Travis CLI installed. You can now encrypt your `now token` by running this command in your project directory:

```
travis encrypt NOW_TOKEN=YOUR_TOKEN --add
```

> Replace YOUR_TOKEN with the token with the token you copied from your Now.sh dashboard.


This snippet encrypts `YOUR_TOKEN` and adds the encoded version to `.travis.yml` file under  the`.env` section. If the above command is successful you should see some configurations appear on `.travis.yml` file as shown below:

```
env:
    global:
    secure: DZC4XpjVPoPl4oXKPD2QATP4++vbPUXllQW0XZaUnSp4S/U9zQamciEMPjvwot324CC/nWm8eL5EyY4WIEyhvhbWwtl85GIYJJeSVhJbkOcwX9Z8Z95aE+9ajI0INNgE9xS0f7jHYqUOSGTDz0aagGrl8ZgA/qI7qL7QZKLgX07e3nh5Zgyjtrgvyukhchtyiuhoetryuiozb4EoUUc8LJAZJPXBcok/qAmuxPQe6vZt5OTmhNPeL0efdRt861dql45A2qHKOGREYm3Ma0aV1IuqeCLrmoJkT5u7oGd+pG+OWh7LlgA1bjFbTufT/2YiGqCKDNLwbsX8OzCqOluOSnm8Rb32Yr6VJIw/ulVweg+ZRsEIdNaY=
``` 

The key generated above will be used for deploying your application to Now.

## Testing the Continuous Deployment Pipeline

To test your deployment pipeline all you need to do is add or change files, commit and push to your remote repository which you created in the course of this tutorial.

### Pushing Your First Change

Change the text in your `index.html` file to something like - **Hello World! Watch this space.** Commit your changes and push to GitHub by running the following commands individually:

```bash
git add -A
    
git commit -m "configured deployment pipeline with travis"
    
git push
```

Once the changes has been pushed to your repository a deployment process is triggered and Travis CI takes over the build process. It handles this process based on the configurations in `.travis.yml` file which you created in your root directory. As this process is going on you should see the log on your Travis dashboard.

Once the build is completed, visit your Now dashboard you should see the current deployment instance created with a unique URL which you can visit to view your deployed application on the browser.

### Pushing Your Second Change

Next, you can make more changes to your `index.html` file, commit  the changes and push again. Watch your changes go live in minutes. 

```bash
git add -A
    
git commit -m "made adjustment to index.html files"
    
git push
```


Once again visit your Now dashboard you should see the current deployment instance created with a unique URL which you can visit to view your deployed application on the browser. The deployment instances are displayed as shown below:

![](https://d2mxuefqeaa7sj.cloudfront.net/s_256435711D8498B15897840D6DBA9A5C15B103EC205218F06CA3BF9F3DF56283_1532681757775_Screen+Shot+2018-07-27+at+9.55.14+AM.png)


## Conclusion

In this article you have learnt about one of the buzzing terms in modern software development - continuous deployments. You learnt about git hosting web services, CI servers, a PaaS, and their respective duties in continuous deployments. Particularly, you used tools like GitHub, Travis CI, and Now.sh. You even went ahead to create a Node.js app and deploy by yourself. Isn’t that awesome? 

With this knowledge, you can go ahead and apply continuous integration to your much more complex projects. You can even decide to try tools similar to what is used here, like trying a different CI server, or a new git hosting web service. I look forward to seeing what you will build. Cheers! 
