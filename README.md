# FAB Re-Frame Starter Project 

[Fenton](https://github.com/ftravers) and [Alex's](https://github.com/aeberts) re-frame starter project.

This is a full-stack, single-page application starter project based on re-frame and Datomic Free. 
The goal of this project is to explore a modern clj/cljs-centric web application architecture and to serve as a solid base project that can be used to quickly prototype ideas. This readme not only serves as a guide to using the project but also as a repository for notes, background and rationale for the choices we made during development.  

This is being developed on MacOS and Linux and while it may work on Windows, there are no plans to officially support it (sorry).   

- Build Tools
    - Leiningen
    - shadow-cljs

- Editors Supported (out of the box)
    - [Cursive](https://cursive-ide.com/)
    - [Emacs / Cider](https://docs.cider.mx/cider/index.html)

- Dev Tools
    - [CLJS DevTools](https://github.com/binaryage/cljs-devtools)
    - [Dirac]() (not yet implemented)

- Front End
    - ClojureScript, Re-Frame and Reagent
    - GardenCSS (not yet implemented)
    - Vanilla ReactJS components (via shadow-cljs) 
    
- Back End
    - [Datomic Free](https://my.datomic.com/downloads/free)
    - [Reitit](https://www.metosin.fi/blog/reitit/) 
    - [Integrant](https://github.com/weavejester/integrant) (not yet implemented)  

- Deployment
    - Deployment to Heroku (not yet implemented)

- Testing
    - Karma (not yet implemented)


### Getting Started
Install the dependencies in <a href="#environment-setup">environment setup</a> 

Install the javascript project dependencies:
```npm install```

Run the development environment:
```lein dev```

To get a clojure repl simply connect to the nrepl server running on port 8777

To get a clojurescript repl connect to a clojure repl and then in the repl run the following command:

```clojurescript
(shadow.cljs.devtools.api/nrepl-select :app)
```  

### Features

- Hot Code Reloading (via shadow-cljs)
- Configuration Management

### Environment Setup
<a name="environment-setup"/>

1. Install [JDK 8 or later](https://openjdk.java.net/install/) (Java Development Kit)
2. Install [Leiningen](https://leiningen.org/#install) (Clojure/ClojureScript project task &
dependency management)
3. Install [Node.js](https://nodejs.org/) (JavaScript runtime environment) which should include
   [NPM](https://docs.npmjs.com/cli/npm) or if your Node.js installation does not include NPM also install it.
4. Install [karma-cli](https://www.npmjs.com/package/karma-cli) (test runner):
    ```sh
    npm install -g karma-cli
    ```
5. Install [Chrome](https://www.google.com/chrome/)
6. Install [Datomic Free](https://my.datomic.com/downloads/free)
7. Install [clj-kondo](https://github.com/borkdude/clj-kondo/blob/master/doc/install.md) (linter)
8. Clone this repo and open a terminal in the `fab` project root directory
9. (Optional) Download project dependencies:
    ```sh
    npm install
    ```
### Project Architecture

What follows is a detailed explanation of each component of the system. The goal is to provide enough information so that a beginner web developer can understand the responsibilities of each component in this system and be able to swap them out with libraries of their own choosing if necessary.

The tech stack below was chosen based on the following criteria:
  * Support for modern (bleeding edge?) technologies and protocols (which may mean that it is not supported by all browsers)
  * Clojure and Java-centric choices (i.e. choose good Clojure(script) options when available e.g. Transit vs JSON)
  * Where possible prefer options that are data-oriented and declarative in nature (i.e. "say what should be done" vs. "do what I say")
  * Use simple, modular components where possible to make substitution of modules easier
  * Use components that favour ease of use over performance (i.e. ones that don't require extensive configuration and work OOTB) 

#### Build Tools

Leiningen

shadow-cljs

shadow-cljs allows easy use of javascript libraries without having to provide "externs" files. The primary maintainer of shadow-cljs Tomas Heller (@thheller) is very active in maintaining shadow-cljs and pushes useful new features regularly. 

Other development benefits of shadow-cljs are:
  * hot code reloading
  * cljs inspectors
  
BinaryAge cljs-devtools : https://github.com/binaryage/cljs-devtools
  * cljs-devtools provides some enhancements for cljs developers using Chrome:
    * Better formatting of cljs values in Chrome's DevTools console
    * More informative exceptions
    * Long stack traces of async calls
    
Unfortunately, the "custom formatter" feature that allows cljs Devtools to work is deprecated in future versions of Chrome and so the primary feature of cljs-devtools may not work in the medium term. There's no good alternative or workaround that we know of at present.     

#### Web Server: HTTP-Kit

_What is a web server?_

Most developers know what a web server is and how it works but bear with me, the discussion below is relevant and Clojure(script) related.

A web server is designed to store and deliver web pages to browsers (a.k.a. "Clients" or "User Agents") via HTTP. Web servers can refer to the physical hardware that responds to HTTP requests or the software running on a physical server somewhere. While the type of hardware your server is running on is important for performance we're mostly interested in the web server software and how it interacts with you Clojure(script) application.

Eric Normand did a great write-up of Clojure web servers here: https://purelyfunctional.tv/mini-guide/clojure-web-servers/

Peter Taoussains published some benchmarks of the major clojure web servers here: https://github.com/ptaoussanis/clojure-web-server-benchmarks/tree/master/results  The benchmarks are 5 years old but the results are still interesting and relevant.

While not the fastest of java or clojure web servers we chose http-kit for this project because it is simple to use, supports the Ring spec, has limited dependencies and can be extended to support more exotic configurations.

What is this Ring thing and do we need it?

If you are exploring developing web applications in Clojure you will no doubt have run across references to Ring. But what is Ring and why should I care about it? Ring is Clojure web application library which abstracts the details of HTTP into a simple Clojure API. You can think of Ring as more of an interface or specification of the components that are needed for a web application. The full spec is on Github, if you're interested in the details. The benefit of using Ring is that libraries that implement the Ring specification are modular and can all work together. 

Other Web Server Options:

Pedestal/Jetty (http://pedestal.io/index)

Pedestal/Jetty is another great web server option for Clojure apps but we went with http-kit for this project because it's a little simpler to configure out of the box and has fewer dependencies. 

#### Application Server

Simple clojure applications do not need a separate application per-se. The clojure code in clj and cljc files is called by handlers from the web server (in our case http-kit) and returns responses to the client via the web server. The clojure application is started by the "traditional" methods i.e. via leiningen. 

#### System Lifecycle Management : Integrant

What is system lifecycle management? (SLM)

Any client / server system contains multiple components: servers, databases, application servers, etc. Even in small systems, these separate components must be supplied with a particular configuration and started and stopped in a particular order. System lifecycle management libraries are useful to orchestrate the startup and shutdown of these separate compoents.

There are many clojure-based SLM libraries available (Integrant, Component, Mount, Duct, etc.) - what responsibilities do they have and how do they work?
     
(TBC)
     
Why did we choose Integrant?
  * Regularly maintained
  * Declarative and data-driven
  * Dependencies are resolved from the configuration before the system is initialized 

#### Back-End Router: Reitit (TBC)

_What is a back-end router?_

A back-end router is part of your application server and is the component that decides how to respond to web requests received from a client (for example the the client application running in a browser). Back-end routers can respond to simple "static" routes which respond to urls that have no additional details (e.g. "http://yoursite.com/home") or more complicated "dynamic" routes where details are provided (e.g. "http://yoursite.com/users/42"). Back in the "old days" of the internet when web pages were mostly rendered on the server, back-end routers translated inbound web requests into application logic, fetched whatever data was required, filled in HTML templates with the data and returned the response to the client browser to be displayed. 

As web applications became more interactive, more of the work for rendering the client UI was done in the client browser. 

Why do I need a back-end router?

* What features does it need to have?

* What are the "nice to have" features of a back-end router?

* Why did we choose reitit as our back-end routing library?

* What other options are there?

Compojure : https://github.com/weavejester/compojure

"A small routing library for Ring"

Secretary

Bidi

Clerk

Accountant

#### Back-End Database: Datomic 

(TBD)

#### Front-End Router: Reitit (TBC)

_What is a front-end router?_

Single Page Applications (SPAs) often only have one back-end route which supplies a simple index.html file and the applications javascript files. The application's UI and all logic is rendered by a javascript library in the browser (e.g. React) and the back-end is mostly responsible for other tasks such as interacting with databases or 3rd-party services (e.g. like marketing mail servers). In practice, front-end routers are primarily responsible for managing what views an application renders when the user requests a particular URL. There are other 

A front-end router is responsible for:
   * Interpreting or "resolving" URIs
   * Handling the case where the user clicks the browser's "back" or "forward" buttons
   * Managing HTML5 pushState

* What features does it need to have?

* What are the "nice to have" features?

* Why did we choose reitit as our front-end routing library?

* What is HTML5 pushState?

#### Front-End UI Framework : Re-frame


#### Deployment : Heroku

  * Push to git repo on Heroku and you're done. 
  * Free tier available
  * Supports custom domains

#### Testing

