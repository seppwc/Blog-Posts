# FuchsiaJS under the hood

## Posts

1. Introduction - What is FuchsiaJS
1. How FuchsiaJS uses JSX in a NodeJS environment
1. Building JSX Controllers in FuchsiaJS
1. How Config/Settings are handled in FuchsiaJS
1. How database Models work with FuchsiaJS
1. Adding MVC in FuchsiaJS and How to create a Template Engine.
1. Building a CLI using OOP, Typescript and Commander

## Introduction

Like most creations, the founding thought was "I wonder if this would work...". And thus the birth of (yet another...) a new JS framework was born.

My thought was "I wonder if JSX would work in NodeJS", obviously there was no reason why it shouldn't, as JSX is essentially sytactic sugar which a compiler such as TSC or Babel, would convert to function calls. Then after some research on how to write a JSX pragma for JS on the frontend began experiments of how it would work in Node. After a while the thought turned into "I wonder how I could use this with express", and after some botched attempts I finally saw the words I was looking for appear in my [Postman](https://www.postman.com/) client... "Hello World"

And so my Journey down the open source rabbit hole began. To create a package im pretty sure no-one wanted or would use, but a project where the sole driving force are the words "Wouldn't it be cool if..."

[FuchsiaJS](https://github.com/Phl3bas/FuchsiaJS) is a web framework built upon Express to build declarative Routing and Models utilising JSX in a NodeJS environment.

In this first blog entry I will explain the basic API of the project. Then in subsequent entries I will show an under the hood look at how I went about creating each element of the framework in order to create this framework.

Lets have a look at how that looks from a user perspective with the age old "Hello World" app.

## Hello World Example

```javascript
import { JSX, FuchsiaFactory, useApplication, createModule } from "@fuchsiajs/core";
import { Controller, Route, HTTP } from "@fuchsiajs/common";
import { ExpressAdapter } from "@fuchsiajs/express"

const AppController = () => {

  const HelloWorld = (req): Pomise<string> => {
      return "hello world";
  };

  return (
    <Controller path="/">
      <Route method={HTTP.GET} path="/" callBack={HelloWorld} />
    </Controller>
  );
};


const AppModule = createModule({
    controllers: [AppController],
})

const main = async () => {
  const app: FuchsiaApplication = await FuchsiaFactory.create<ExpressAdapter>(
        AppModule,
        new ExpressAdapter(),
        {}
    );

    useApplication(app);
};

main();
```

Lets write boring list out the parts I had to create for this simple hello world to work

+ Compiling JSX in Node - We had to actually set this up using either Babel or TSC (typescript compiler), I went with TSC for the sole perpose that it didnt need any more dependancies to be downloaded.
+ Write a Generic JSX function that will accept props and children and be able to work with any function/class we or the user writes.
+ Routing using a Controller/Route component system.
+ Get the Routing system to work with a callback system that excepts a request object and returns a reponse body. 
+ Structuring an Application using a Module system.
+ create an abstraction for Express, so if I want to add an adapter for Fastify in future, it would work.
+ create an application layer to compose all these bits together.

things not shown in example:
+ create a outside closure scope to be able to use a hooks like system to pass data,objects,instances around the application.
+ create a way to load configuration of the application (not use in the example).
+ create a way to add global and scoped middleware (not use in the example).


This would normally be split into separate files but for demonstrative perposes (...and also cause im SUPER LAZY), I've thrown it all into one code block and youll just have to make believe while I explain the different parts.


## Imports
```javascript
import { JSX, FuchsiaFactory, useApplication, createModule } from "@fuchsiajs/core";
import { Controller, Route, HTTP } from "@fuchsiajs/common";
import { ExpressAdapter } from "@fuchsiajs/express"

```

FuchsiaJS uses the @FuchisaJS npm namespace to hold all the different packages.

The project is split into several different packages and all stored in a mono repo on [github](https://github.com/Phl3bas/FuchsiaJS), I though this was an interesting way to organise the project as initially i would be on only one working on it, and I though splitting into several packages would make working on and updating seperate parts easier (I was very wrong...).

The packages of the project at time of writing are

+ @fuchsiajs/core - main package of project contains, main instances for running a fuchsia application,  one key import to look at here is the "JSX" import, this is the function that the compiler will replace any JSX with, this is similar to in react where `import React from 'react'` is needed in any file that uses JSX, this 'JSX' function needs to be imported into any file that uses JSX.
+ @fuchsiajs/common - common functionality in a project, this contains controller/route components and various utils.
+ @fuchsiajs/express - the adapter package for using expressjs under the hood.
+ @fuchsiajs/orm - contains orm adapters for using mongoose and eventually sequalize packages, also contains components creating models using JSX.
+ @fuchsiajs/template - package for fuchsiajs templating engine


## Controller
```javascript
const AppController = () => {

  const HelloWorld = (req: Request): Promise<string> => {
      return "hello world";
  };

  return (
    <Controller path="/">
      <Route method={HTTP.GET} path="/" callback={HelloWorld} />
    </Controller>
  );
};

```

This is one of the two main JSX-y bits of the framework, the Controller. 

Controllers are a function that returns a configured controller component. Controllers are built of three parts, Controller Component, Route Components and Callbacks.

+ Callbacks: accept one arguement, the Request object, and anything returned get sent as a reponse.
+ Controllers: controllers only have one prop "Path" which sets the base path for the nest routes, and accepts children of Routes.
+ Routes: route components accept the type of method as a string (here we use an Enum the package provides as were using typescript), the path of the route, and the callback function. It also accepts a props of "json" to send a json repsonse, "render" to send a template response, "text" to send a text response, if none of this props are set it will default to express' "res.send()" reponse.


## Module
```javascript
const AppModule = createModule({
    controllers: [AppController],
})
```

The next part is the Module, Modules are a why of structuring a fuchsia project and is unabashedly an idea I took from NestJS. We create a module object by calling createModule and passing a configured object to it as a parameter. The passed object can have the following properties.

+ controllers: An array of controller functions for that module, these are global and will always be imported/exported.
+ services: An array of services, a service can be a model, service class to hold your callback functions, or middleware, these are all scoped unless also included in "exports"
+ imports: This is an array of Modules we wish to import into this module, we will only import their declared "exports"
+ exports: An array of declared exports which will be passed down to other modules that import this module.

Modules will eventually resolve down to one Module through importing and exporting to a Root App Module, which will get passed to our application instance.


## Application Instance
```javascript
const main = async () => {
  const app: FuchsiaApplication = await FuchsiaFactory.create<ExpressAdapter>(
        AppModule, // RootModule of project
        new ExpressAdapter(), // Adapter that alows fuchsia to use express under the hood
        {} // a project config object[optional]
    );

    useApplication(app);
};

main();
```
finally in our entry file, we have our async main function definition and function call.

here we have several components

+ FuchsiaFactory: a singleton class that is used to build a configured Fuchsia Application using the factory pattern.
+ ExpressAdapter: this is adapter that offers our application an abstraction over the express package.
+ AppModule: this is the root module we created in the previous step.
+ FuchsApplication: this is an instance of our configured fuchsia application
+ useApplication: this is a hook that provides a closure scope around our entire running application, it calls listen on our http server and instansiates our components. This also allows use to use something similar to Hooks in react in our components and services.
+ configuration object[optional]: this is an object literal providing various express and database configuration settings, this is optional as the application will also look for a Fuchsia.config.json or Fuchsia.config.js file in the project root. If one of those files exist, the object literal passed here is used to overide settings provided by the file.


Outside of the provided example, there is also the functionality of using JSX to declare our database models. using a sytax such as


```javascript
    import {Mongoose, Schema, String, Number} from '@fuchsiajs/orm';
    import {createModule} from '@fuchsjs/core';


    const User = () => (
        <Schema name="user">
            <String name="name" maxlength={30} />
            <Number name="age" min={18} max={9999} />
            <String name="email" match={\SOME_REGEX_MAGIC\} />
        </Schema>
    )

    const AppModule = createModule({
        service: [Mongoose.fromSchema(User)]
    })

```


which we can get get a singleton in our callback by using the "useModel" hook like below


```javascript
    import { useModel } from '@fuchsiajs/core';

    const callBack = (req) => {
        const user = useModel('user');

        const allUsers = user.findAll()

        return {users: allUsers};
    }
```

so that is the basic API of FuchsiaJS, In the next entry I will explore the project set up and how you can use JSX in node and how its implemented in FuchsiaJS.