---
layout: post
title: "Building GraphQL APIs with Kotlin, Spring Boot and MongoDB "
description: "Learn how to build GraphQL APIs with modern tools."
date: "2019-03-04 08:30"
author:
  name: "Idorenyin Obong"
  url: "https://twitter.com/kingidee/"
  mail: "idee4ril@gmail.com"
  avatar: "https://cdn.auth0.com/blog/guest-authors/idee.png"
related:
- 2017-11-15-an-example-of-all-possible-elements
---

**TL;DR:** In this article, you will learn how to build GraphQL APIs with Kotlin and Spring Boot. The API will make use of MongoDB for database operations and of course, it will be secured by Auth0. You can find the final code developed in this part in this GitHub repository.

## Prerequisites
Before proceeding, there are some tools you need to ensure you have on your machine in order to follow the article seamlessly. They include:

### JDK 
This is the Java Platform, Standard Edition Development Kit. Normally, this environment is used for developing Java applications, but since Kotlin runs on the JVM just like Java, you need it. You can download it [here](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html).

### Intellij IDEA

This is an IDE ( Integrated Development Environment) built by [JetBrains](https://www.jetbrains.com) and used for developing Java and Kotlin applications. It is a paid software, but you can make use of the free community edition. You can download the IDE [right here](https://www.jetbrains.com/idea/download).


## Introduction
By now, you may be slightly anxious about the technologies you will use in this article. In this part, you will get a brief overview of the stacks you will use for the article.

### Kotlin
According to the official [docs](http://kotlinlang.org/docs/reference/faq.html#what-is-kotlin):

> Kotlin is an OSS statically typed programming language that targets the JVM, Android, JavaScript and Native.

Breaking this down into pieces, you can say:

- It is an Open Source Software (OSS) - this means the source code is available to everybody and you can contribute if you so wish. The source code is stored on [GitHub](https://github.com/JetBrains/kotlin).
- It is a statically typed programming language - this means that the language performs type checking at compile-time (during compilation). Type checking is the process of verifying and enforcing the operations a type can perform. Examples of types include: `String`, `Int`, etc.
- It runs on the JVM(Java Virtual Machine) just like Java, It can be [compiled to JavaScript](https://kotlinlang.org/docs/reference/js-overview.html), and it can be compiled directly to native code (machine code). Compiling directly to native code means [Kotlin can run without the need for a virtual machine](https://blog.jetbrains.com/kotlin/2017/04/kotlinnative-tech-preview-kotlin-without-a-vm/). 

Outside that definition, here are some other points you should note about Kotlin:

- It is primarily developed by [JetBrains](http://www.jetbrains.com/), although, it has multiple open-source contributors too. 
- Kotlin is designed to interoperate with Java. You can use both together in a project without any hassle.
- At the time of this writing, Kotlin is currently on version 1.3 with 26,384 stars on GitHub. 

Kotlin is becoming increasingly popular especially in the Android development community. In fact, Kotlin is now the official language for Android development in Google! This was made known by a Googler is [a recent tweet](https://twitter.com/jmslau/status/1087827632752738304).


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1550528707622_Screenshot+2019-02-18+at+11.24.07+PM.png)


Many other large companies have adopted Kotlin. Some include: [Square](https://medium.com/square-corner-blog/square-open-source-loves-kotlin-c57c21710a17), [Pinterest](https://www.youtube.com/watch?v=mDpnc45WwlI), etc.

### Spring Boot
Spring Boot is a framework built on top of the earlier existed Spring framework. Spring is primarily used to build Java-based enterprise applications. Here are some cool features you enjoy when you use Spring Boot:


- Auto-configuration: Previously, if you are building Spring applications, you would have to configure a lot of things by yourself. But with Spring Boot, your application auto-configures a lot of that for you based on the dependencies you add.  
- Lesser starter dependencies:  For your previous Spring projects, you need a couple of dependencies to get your application up and running, such as: `spring-core`, `spring-web`, `spring-webmvc`, `tomcat-embed-core`, etc. With Spring Boot instead, just one dependency - `spring-boot-starter-web` takes care of everything.
- Spring Boot Initializer: The Spring initializer is an online tool that helps you easily create Spring Boot applications and also add dependencies to them. The initializer can be accessed here -  [https://start.spring.io/.](https://start.spring.io/) You will use this tool in the course of this article.
- Spring Boot is opinionated. This means that the framework choses a default way of doing things. This may sound like a bad thing, but opinionated frameworks seem to save time because you’ll be building in a way many other developers have done before.

In case you want to read further on how Spring differs from Spring Boot, you can use this [resource](https://paper.dropbox.com/doc/Building-GraphQL-APIs-with-Kotlin-and-Spring-Boot--AYWi~oyOgcyrPs4VadJSREzPAg-AefcRzEmuLmy29OcMVeN5#configure-embed).

### GraphQL
GraphQL is a query language for APIs. It was developed by Facebook in 2012 and used internally
at first but was later open-sourced in 2015. It is seen by some as a replacement for REST because of some the awesome qualities it poses. Here are a few points you should note about GraphQL:


- In GraphQL, you have just one endpoint as compared to REST which has multiple endpoints.
- GraphQL uses a type system to describe data. With this, the clients know exactly what data is available and in what form.
- In GraphQL, a client asks for what it wants during a request.

You will get to see the use of the above listed features when you dive deeper in the article. GraphQL has attracted top patronizers such as GitHub, Twitter, Pintrest, Shopify etc. If GraphQL is pretty much new to you, you can use [this resource](https://graphql.org/learn/) to get up to speed with it.

### MongoDB
MongoDB is a non-relational database management system. MongoDB is document-oriented and stores data in a [Binary JSON (BSON)](http://bsonspec.org/) manner.  One of the advantages of this db solution is that it does not have a predefined schema, hence, it is easy to scale overtime.

You can confirm if you have Mongo installed on your machine by running this:

```bash
mongo -version
```

If you don’t have, you can follow [this manual](https://docs.mongodb.com/manual/installation/) to install it.


## What you will build
In this article, you will build a GraphQL API that performs typical CRUD operations. The API is focused on snacks and reviews. The API will serve you with a list of snacks and their reviews. But beyond just querying the API, you will be able to update the API by creating, updating, or deleting a snack.


## Bootstrapping your app
As mentioned earlier, Spring Boot has an initializer that helps you bootstrap your applications faster. Open the [initializer](https://start.spring.io/) and fill in the options as seen in the image below:

![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551308262819_Screenshot+2019-02-27+at+11.57.19+PM.png)

Here, you are generating a  gradle project with Kotlin and Spring Boot `2.1.3`. The package-name for the app is `com.auth0.kotlingraphql`. You will need to search for the `Web` and `MongoDB` dependency and add them as done above.  

The web dependency is a starter dependency for building web applications while the MongoDB dependency is for your database solution. After that, click the *Generate Project* button. This will download a zipped file that contains your project. Extract the project, and open it in IntelliJ IDEA.

Next, you have to add some more dependencies. Open your `build.gradle` file and add these:

```gradle
// /kotlingraphql/build.gradle
dependencies {
    // other dependencies
    implementation 'com.graphql-java:graphql-spring-boot-starter:5.0.2'
    implementation 'com.graphql-java:graphiql-spring-boot-starter:5.0.2'
    implementation 'com.graphql-java:graphql-java-tools:5.2.4'
}
```

These dependencies will give GraphQL functionalities to your app. After adding them, import changes so that gradle will download these dependencies to your project.

After that, open the `application.properties` file located in the `kotlingraphql/src/main/resources/` directory and add this:

```
server.port=9000
spring.data.mongodb.database=kotlin-graphql
spring.data.mongodb.port=27017
```

In this file, you have set the port where the API will be served, name of the database you would use for the app, and the port where the database will run on. Now, you are ready to start adding logic to your application! 

## Creating your Mongo entities
An entity is a term often associated with a model class persisted, and since you are dealing with APIs to perform CRUD operations, you need to persist data in one way or the other. 
In this section, you will create entities that you’ll use in the course of building your API.

First, create a new package called `entity`. You will keep all the entities you need here in this package. Next, create a new class called `Snack` inside the package and add this snippet to it:

```kotlin
// ./src/main/kotlin/com/auth0/kotlingraphql/entity/Snack.kt

import org.springframework.data.annotation.Id
import org.springframework.data.mongodb.core.mapping.Document

@Document(collection="snack")
data class Snack (
    var name: String,
    var amount: Float
){
    @Id
    var id: String = ""

    @Transient
    var reviews:List<Review> = ArrayList()

}
```


> Data classes in Kotlin derive getters, setters, and other utility functions for you by default!

This class is a model of a single snack that will be stored and retrieved. Each snack has `name`, `amount`, `id` (a unique identifier), `reviews`. The `reviews` variable will hold all the reviews peculiar to a particular snack.

You will use this model when storing to the database, hence the use of the `@Document` annotation. The name of the collection is specified using the `collection` variable in the annotation. If it was not specified, it automatically uses the class name. 

The `id` is annotated with `@Id` to tell MongoDB that the variable will hold the unique identifier for the entity. It goes beyond just naming the variable `id` you have to use this annotation. The `@Transient` annotation on `reviews` means the `reviews` variable will not be persisted to the database.

Next, create another class called `Review` still under the `entity` package and add this snippet: 

```kotlin
// ./src/main/kotlin/com/auth0/kotlingraphql/entity/Review.kt

import org.springframework.data.mongodb.core.mapping.Document

@Document(collection = "reviews")
data class Review(
    var snackId: String,
    var rating: Int,
    var text: String
)
```

This is very similar to what you created. This class is a model for a single review and here, the `snackId` (the id of the snack the review is meant for), `rating`, and `text` is required. 


## Creating your Mongo repositories
According to the official Spring documentation:
> [*A repository is a mechanism for encapsulating storage, retrieval, and search behaviour which emulates a collection of objects"  (Evans, 2003)*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Repository.html)

You can simply say, a repository is a class responsible for some form of data storage and retrieval. You will now create repositories to match the two entities you created earlier.

First, create a new package called `repository`. Inside the package, create a Kotlin interface called `SnackRepository` and add this snippet:

```kotlin
// ./src/main/kotlin/com/auth0/kotlingraphql/repository/SnackRepository.kt

import com.auth0.kotlingraphql.entity.Snack
import org.springframework.data.mongodb.repository.MongoRepository
import org.springframework.stereotype.Repository

@Repository
interface SnackRepository : MongoRepository<Snack, String>
```

The interface you have just created implements the `MongoRepository` interface which provides some predefined methods that you need.  Some of these methods include: `findAll`, `saveAll`, `findById` among many others.

The `MongoRepository` interface takes in two parameters types, `Snack` and `String`. The first parameter (`Snack`) is the data type that will be saved and retrieved from this repository while the second parameter (`String`) is the data type of the `ID`.  `String` is used since the data type for the `id` variable in the `Snack` entity is a `String`.

The `@Repository` annotation is used to indicate that the class is a repository. Although creating the `SnackRepository` without the `@Repository` annotation still works as expected, the annotation has its benefits as follows:


- It helps to clarify the role of the interface/class in the application.
- It helps you to catch specific native exceptions. These exceptions depend on the technology used. For instance, if you use Hibernate, you can encounter an error like `HibernateException`, so the annotation intercepts it and applies an appropriate translation on the exception.

Next, create another Kotlin interface named `ReviewRepository` still in the `repository` package and add this:

```kotlin
// ./src/main/kotlin/com/auth0/kotlingraphql/repository/ReviewRepository.kt

import com.auth0.kotlingraphql.entity.Review
import org.springframework.data.mongodb.repository.MongoRepository
import org.springframework.stereotype.Repository

@Repository
interface ReviewRepository : MongoRepository<Review, String>
```

This is very similar to the first repository created. Since this repository is to manipulate data for the `Review` entity, the first parameter for `MongoRepository` is changed to `Review`.


## Writing your GraphQL schema
Unlike REST APIs where you have to declare endpoints based on the resources they return, in GraphQL, you need to define a schema. The schema is used to:

- Declare the types available and their relationships.
- Declare how data can be mutated or queried. 

While you have `POST`, `GET`, `PUT` and others as request methods in a REST API, for GraphQL, you have just `Query` (equivalent of `GET` in REST) and `Mutation` (equivalent of `PUT`, `POST`, `PATCH` and `DELETE` in REST). You will now learn how to write a GraphQL schema. The schema will be written in Schema Definition Language (SDL). 

Create a new file named `snack.graphqls` in the `kotlingraphql/src/main/resources` directory and add this snippet:

```
type Query {
    snacks: [Snack]
}

type Snack {
    id: ID!
    name: String
    amount: Float
    reviews: [Review]
}

type Mutation {
    newSnack(name: String!, amount: Float!) : Snack!
    deleteSnack(id: ID!) : Boolean
    updateSnack(id:ID!, amount: Float!) : Snack!
}
```

In this file, you declared three types with their respective fields. The `Query` type is a standard type used by a client to request data. The `Query` type has a field `snacks` which returns a  `Snack` list. The `Snack` type as declared above mimics the snack entity you created.

The `Mutation` type is another standard type used by a client to add, update or delete data as seen in the `Mutation` type declared.


> A schema is like a contract between the client and the server. Anything the client tries to do with the server that is outside the schema will not work.

Notice that the fields in the types declared can have [Scalar return types](https://graphql.org/learn/schema/#scalar-types) or a custom declared type.

Still in the `kotlingraphql/src/main/resources` directory, create another file named `review.graphqls` and add this snippet:

```
extend type Query {
    reviews(snackId: ID!): [Review]
}

type Review {
    snackId: ID!
    rating: Int
    text: String!
}

extend type Mutation {
    newReview(snackId: ID!, rating: Int, text:String!) : Review!
}
```

In this file, the keyword `extend` is attached to the `Query` and `Mutation` to extend the types declared in the other file. This file is also similar to the last one you created. 


## Writing your GraphQL resolvers
A resolver is a function that provides a value for a field or types declared in the schema. This means, you now have to create corresponding functions for the fields you declared in the last section: `snacks`, `newSnack`, `deleteSnack`, `updateSnack`, `reviews`, `newReview`. These functions must have the same return type as the fields themselves.

For clarity sake, you will need to create multiple Kotlin classes to house the resolvers. Create a package named `resolvers`. Then create a class named `SnackQueryResolver` inside the package. After creating it, add this snippet:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/resolvers/SnackQueryResolver.kt

import com.auth0.kotlingraphql.entity.Review
import com.auth0.kotlingraphql.entity.Snack
import com.auth0.kotlingraphql.repository.SnackRepository
import com.coxautodev.graphql.tools.GraphQLQueryResolver
import org.springframework.data.mongodb.core.MongoOperations
import org.springframework.data.mongodb.core.query.Criteria
import org.springframework.data.mongodb.core.query.Query
import org.springframework.stereotype.Component

@Component
class SnackQueryResolver (val snackRepository: SnackRepository,
                          private val mongoOperations: MongoOperations) : GraphQLQueryResolver {

    fun snacks(): List<Snack>{
        val list = snackRepository.findAll()
        for(item in list){
            item.reviews = getReviews(snackId = item.id)
        }
        return list
    }

    private fun getReviews(snackId:String) : List<Review> {
        val query = Query()
        query.addCriteria(Criteria.where("snackId").`is`(snackId))
        return mongoOperations.find(query, Review::class.java)
    }

}
```

This class is specifically created for queries in the `snack.graphqls` file, hence the name `SnackQueryResolver`. The class implements an interface `GraphQLQueryResolver` provided by the graphql dependency you added earlier on. The class is also annotated with `@Component` to show that the class is a component, meaning that the class will be auto detected for dependency injection by Spring.

Remember that the query type in the  `snack.graphqls` looks like this:

```
type Query {
    snacks: [Snack]
}
```

And so, the `SnackQueryResolver` class contains one public function named `snacks` which should return a list of snacks. Notice that the field name corresponds to the function name. This is how it should be else, the function won’t be recognised as a resolver for any field.

In the `snacks` function, the `snackRepository` is used to fetch all the snacks from the database, and for each snack, all the reviews are fetched alongside.

The next class you should create is called `SnackMutationResolver`. Create it inside the `resolvers` package. After creating the class, add this snippet to it:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/resolvers/SnackMutationResolver.kt

import com.coxautodev.graphql.tools.GraphQLMutationResolver
import com.auth0.kotlingraphql.entity.Snack
import com.auth0.kotlingraphql.repository.SnackRepository
import org.springframework.stereotype.Component
import java.util.*

@Component
class SnackMutationResolver (private val snackRepository: SnackRepository): GraphQLMutationResolver {

    fun newSnack(name: String, amount: Float): Snack {
        val snack = Snack(name, amount)
        snack.id = UUID.randomUUID().toString()
        snackRepository.save(snack)
        return snack
    }

    fun deleteSnack(id:String): Boolean {
        snackRepository.deleteById(id)
        return true
    }

    fun updateSnack(id:String, amount:Float): Snack {
        val snack = snackRepository.findById(id)
        snack.ifPresent {
            it.amount = amount
            snackRepository.save(it)
        }
        return snack.get()
    }

}
```

This class is created for the mutation field resolvers in `snack.graphqls`. This class has three methods for the three fields in the schema, namely:


- `newSnack` - This function takes the name and amount of the snack and creates a new snack in the database. Before saving the snack to the database, a random unique id is generated for that snack. This function returns the new snack created.
- `deleteSnack` - This function is meant to remove a snack from the db based on the id. This function returns a boolean value.
- `updateSnack` - This function is used to update a snack if only the snack was initially present. This function returns the snack that was just updated .

You will now create resolvers for the `review.graphqls` schema. Create another class inside the `resolvers` package named `ReviewQueryResolver` and add this snippet:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/resolvers/ReviewQueryResolver.kt

import com.coxautodev.graphql.tools.GraphQLQueryResolver
import com.auth0.kotlingraphql.entity.Review
import org.springframework.data.mongodb.core.MongoOperations
import org.springframework.data.mongodb.core.query.Criteria
import org.springframework.data.mongodb.core.query.Query
import org.springframework.stereotype.Component

@Component
class ReviewQueryResolver (val mongoOperations: MongoOperations): GraphQLQueryResolver {

    fun reviews(snackId:String) : List<Review> {
        val query = Query()
        query.addCriteria(Criteria.where("snackId").`is`(snackId))
        return mongoOperations.find(query, Review::class.java)
    }

}
```

This class is supposed to handle any resolver relating to the query type in `review.graphqls` file. This class has one function - `reviews`. This function returns a list of reviews from the db depending on the `snackId` passed in. 

Finally, you will create the last class in this section. Create a class named `ReviewMutationResolver` and add this snippet:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/resolvers/ReviewMutationResolver.kt

import com.coxautodev.graphql.tools.GraphQLMutationResolver
import com.auth0.kotlingraphql.entity.Review
import com.auth0.kotlingraphql.repository.ReviewRepository
import org.springframework.stereotype.Component

@Component
class ReviewMutationResolver (private val reviewRepository: ReviewRepository): GraphQLMutationResolver {

    fun newReview(snackId: String, rating: Int, text:String): Review {
        val review = Review(snackId, rating, text)
        reviewRepository.save(review)
        return review
    }

}
```

The resolver here is for the mutation field in the `review.graphqls` schema. In this function, a new review is added to the db using a snack id, rating value and text.

With these resolvers you’ve just created, anytime the client constructs a query, your functions will provide results for those fields requested. Whoop whoop! 🎉 


## Wrapping up your app
Now that you have created your entities, repositories, schema, and resolvers, you need finalize everything in the `KotlinGraphQlApplication` class.  

Add these imports in the import section of the class:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/KotlingraphqlApplication.kt

import com.auth0.kotlingraphql.repository.ReviewRepository
import com.auth0.kotlingraphql.repository.SnackRepository
import com.auth0.kotlingraphql.resolvers.ReviewMutationResolver
import com.auth0.kotlingraphql.resolvers.ReviewQueryResolver
import com.auth0.kotlingraphql.resolvers.SnackMutationResolver
import com.auth0.kotlingraphql.resolvers.SnackQueryResolver
import org.springframework.context.annotation.Bean
import org.springframework.data.mongodb.core.MongoOperations
```

Then add these methods below the `main` method in the class:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/KotlingraphqlApplication.kt

@Bean
fun snackQuery(snackRepository: SnackRepository,mongoOperations: MongoOperations): SnackQueryResolver {
    return SnackQueryResolver(snackRepository,mongoOperations)
}

@Bean
fun reviewQuery(mongoOperations: MongoOperations): ReviewQueryResolver {
    return ReviewQueryResolver(mongoOperations)
}

@Bean
fun snackMutation(snackRepository: SnackRepository): SnackMutationResolver {
    return SnackMutationResolver(snackRepository)
}

@Bean
fun reviewMutation(reviewRepository: ReviewRepository): ReviewMutationResolver {
    return ReviewMutationResolver(reviewRepository)
}
```

From the snippet above, you have declared the resolvers as Spring beans. A bean is an object that is instantiated, assembled, and managed by a Spring IoC container. Now, you can run your app using the green run icon at the toolbar of the IDE. You can also run using this command on the root directory:

```bash
For Mac
./gradlew bootRun

For windows
gradle bootRun
```

When you run your app, you should see logs like this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551364660067_Screenshot+2019-02-28+at+3.37.01+PM.png)


Now, open http://localhost:9000/graphiql on your browser, you should have something like this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551364855220_Screenshot+2019-02-28+at+3.40.29+PM.png)


To add a snack, clear all the comments on the left hand side and add this snippet:

```
mutation {
    newSnack(
    name: "French Fries",
    amount: 40.5) {
        id name amount
    }
}
```

Run the mutation with the run icon seen in the interface, your result should be similar to this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551365282152_Screenshot+2019-02-28+at+3.47.46+PM.png)


You just created a new snack! Now you will create a review for this snack. To do that, run this query:


```
mutation {
    newReview(snackId:"SNACK_ID",
    text: "Awesome snack!", rating:5 
    ){
        snackId, text, rating
    }
}
```


> Replace SNACK_ID with the `id` generated for you when you created a snack.

When you run that query, you should have this response:


```json
{
    "data": {
        "newReview": {
            "snackId": "c72aaba6-7cbe-4af0-8f5a-783eb803f088",
            "text": "Awesome snack!",
            "rating": 5
            }
    }
}
```


## Securing endpoint with Auth0
Your API works perfectly as expected but you need to add a little more spice to it. You will secure it with Auth0.

To get started, the first thing is to login to your [Auth0 dashboard](https://manage.auth0.com) or create an account if you have none. It is very easy to get started [here](http://auth0.com/signup)! Your dashboard should look like this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551034026117_Screenshot+2019-02-24+at+7.46.44+PM.png)


Select the **APIs** item on the options as seen on the left from the image above and create a new API as follows:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551038096847_Screenshot+2019-02-24+at+8.54.28+PM.png)


Set the friendly name of the API to **Kotlin GraphQL API** and the identifier to `**https://www.kotlin-graphql-api.com**`. Keep the signing algorithm at `RS256`. 

Next, open your `build.gradle` file and add the Spring OAuth2 dependency like this:

```gradle
implementation 'org.springframework.security.oauth.boot:spring-security-oauth2-autoconfigure:2.0.6.RELEASE'
```

If you are running JDK 8, your app will run smoothly, otherwise, you might encounter an [OAuth2 spring error creating a bean with name 'springSecurityFilterChain'](https://stackoverflow.com/questions/47866963/oauth2-spring-error-creating-bean-with-name-springsecurityfilterchain). You can fix it by adding these dependencies:

```gradle
implementation 'javax.xml.bind:jaxb-api:2.3.0'
implementation 'com.sun.xml.bind:jaxb-core:2.3.0'
implementation 'com.sun.xml.bind:jaxb-impl:2.3.0'
implementation 'javax.activation:activation:1.1.1'
```

After adding them, sync your gradle files. 

Next, open the `application.properties` file in the `kotlingraphql/src/main/resources/` directory and add these two properties:

```
security.oauth2.resource.id=https://www.kotlin-graphql-api.com
security.oauth2.resource.jwk.keySetUri=https://DOMAIN_NAME/.well-known/jwks.json
```

> Replace the **DOMAIN_NAME** placeholder with your own Auth0 tenant name. It is usually something like `idee.auth0.com`

The properties you added will be used to ensure the security of your API endpoint. Now, create a new class called `SecurityConfig` and add this:

```kotlin
// .src/main/kotlin/com/auth0/kotlingraphql/SecurityConfig.kt

import org.springframework.beans.factory.annotation.Value
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer

@Configuration
@EnableResourceServer
class SecurityConfig : ResourceServerConfigurerAdapter() {

    @Value("\${security.oauth2.resource.id}")
    private lateinit var resourceId: String

    @Throws(Exception::class)
    override fun configure(http: HttpSecurity) {
        http.authorizeRequests()
            .mvcMatchers("/graphql").authenticated()
            .anyRequest().permitAll()
    }

    @Throws(Exception::class)
    override fun configure(resources: ResourceServerSecurityConfigurer) {
        resources.resourceId(resourceId)
    }
}
```

This class will be auto detected by Spring thanks to the annotations. This class ensures that the `/graphql` endpoint is provided with a token before it can be accessed.


> Note that although you were testing the API on http://localhost:9000/graphiql, the main endpoint for the API  is http://localhost:9000/graphql.  The later only provides an interface for easy testing.

In the previous section, you tested the mutations. Now, you will test the queries. Open [Postman](https://www.getpostman.com/downloads/) on your machine and enter this query in the body:

```json
{
"query":" { snacks { id name amount reviews { snackId rating text } } }"
}
```

Now, if you try accessing the endpoint now, you would encounter an authorization error:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551368299659_Screenshot+2019-02-28+at+4.38.05+PM.png)


To get a valid access token, open the API section on Auth0 dashboard, select the client you created earlier, and then click under the *Test* section. There you can copy a valid access token:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551368539635_Screenshot+2019-02-28+at+4.40.43+PM.png)


Go back to Postman and insert your token like this:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551368825664_Screenshot+2019-02-28+at+4.46.23+PM.png)


Now, if you execute your query, you would get a response:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_76143C7D23A4EC4FF528BCDF839EB5A4B87C99992ADE34CF23D3BB3AB4442886_1551368866615_Screenshot+2019-02-28+at+4.46.44+PM.png)

## Conclusion
In this article, you have learned how to build GraphQL APIs using a trending language - Kotlin, a solid framework - Spring Boot and a popular database solution - MongoDB. Feel free to explore the GitHub repository and play around it.

