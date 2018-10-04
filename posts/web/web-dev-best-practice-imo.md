# 我眼中的web开发最佳实践

## 前提
这份最佳实践，针对的是个人/小团队的web端敏捷开发，适合个人或者小团队迅速完成前端、客户端、后台等一系列产品，而没有针对高并发场景。

开发者需要了解前端开发的基本知识，了解HTML / CSS / JavaScript，要了解ES6语法以及node。

> 在高并发场景中，后端需要引入微服务技术，通过微服务框架(如 `istio / linkerd / dubbo`)等结合服务网格技术(service mesh)来支持服务发现、负载均衡、流量控制等功能，再通过统一的api gateway对外进行服务，并以docker及k8s作为资源调度层。这部分内容不是本文的主题。
>
> 对于`graphql`, `serverless`等技术，会通过专门的文章来说明。由于这篇最佳实践是针对当前环境，尽管这些技术很优秀，但是当前还不够成熟，因此我暂时不将它们归入最佳实践。

## 技术栈
* 前端页面采用诸如`react/vue`等框架编写**SPA**，通过webpack等编译打包工具打包成静态资源，部署在web服务器上。
* web服务器使用nginx, S3等。
* 后台api服务器采用node编写，可以使用`express/koa2/egg`等。本实践中采用egg作为api server，主要是考虑到egg提供的约束使得开发过程更少走坑。
* 数据库采用开源数据库mysql。

## 工作流程
1. 客户通过浏览器访问相应地址后，**web服务器**返回所需要的静态资源(html/js/css代码)，浏览器渲染出页面。在这个过程中，为了提高用户体验，可以对资源进行分割，减少首屏渲染时间。也可以通过**CND**加速访问。
2. 当客户点击页面某些按钮时，web页面将判断是否需要与后台进行交互，对于无需与后台交互的操作，可以利用SPA的特性，通过js代码直接在前端进行处理；对于需要的操作，将发起http请求(可以通过fetch/axios发起请求)与后台服务器交互。
3. 后台服务器收到请求后，进行相应逻辑处理，有需要可以访问数据库、缓存、消息中间件等服务，并将处理的结果通过http协议返回。
4. web页面收到返回的结果，更新dom状态(通过redux等状态管理库)，渲染出新界面。

## 前端
* React
  - [React](https://reactjs.org/)是 Facebook 开源的一套前端开发框架，与vue作为当前最流行的前端框架。核心原理是将网页页面拆分成一系列组件，通过操作组件的状态来管理页面的渲染。
* Ant-Design / Pro
  - [Ant-Design](https://ant.design)是由阿里巴巴-蚂蚁金服开源的一套前端组件库，它构建于React之上，通过React提供的组件化和状态管理特性，实现了一套颜值很高的UI组件库(再次感谢FB大佬给前端带来了福音)。Ant-Design在React中扮演的是**Component**(组件)的角色，开发者亦可基于这些组件构建出更上层的组件，并通过组合这些组件，来表现出页面。
  - [Ant-Design-Pro](https://pro.ant.design/)是蚂蚁金服基于 Ant-Design 推出的官方后台管理案例，给我们提供了很好的示范。
* Redux / Dva
  - React专注的是组件内部的状态管理，并不擅于全局状态管理，因此需要引入状态管理库。Facebook推出的`Flux`架构通过单向数据流的思想，增强了前端框架的状态管理能力。
  - [Redux](https://redux.js.org/)是Flux的一种实现和改进，并吸取了`Elm`之长。用官方的话来说：
    - > Redux is a predictable state container for JavaScript apps.
  - [Dva](https://dvajs.com/)是基于Redux与Elm的状态管理库，它采用redux-saga思想，采用generator来处理异步操作。同时，dva内置了react-router与fetch，因此可以看做是一个轻量的前端框架。
    - > 在接触redux-saga/dva之前，我使用的是[redux-thunk](https://github.com/reduxjs/redux-thunk)，它作为Redux的中间件，通过让`action`支持返回函数，使得Redux支持`dispatch`函数，来支持异步操作。redux-thunk足够用来解决简单的应用场景，但面对复杂的异步场景时，redux-thunk用起来相对有些繁琐。而采用generator思想的redux-saga与dva能够更加优雅的解决这一类问题。
* umi
  - [umi](https://umijs.org/)可以理解为React框架的**脚手架**，它整合了react/webpack/react-router/babel/dva等框架，让开发者采用插件与配置的方式来进行前端开发。
    - > umi十分类似vue中的next.js
    - > [create-react-app](https://github.com/facebook/create-react-app)同样是一款优秀的react脚手架，之所以选用umi，主要是因为它整合了dva，路由等框架，用起来更方便。
    - > emmmmm....，好吧，主要是因为任性。

## web server
* [nginx](https://www.nginx.com/resources/wiki/)
  - web服务器的主要作用是用来提供静态资源服务访问。前端框架将代码构建(build)成静态资源后，需要在用户访问页面时，将这些资源返回给用户浏览器。这些工作既可以在后台框架(如spring boot / express / koa)中做，又可以采用独立的web服务器来解决。
  - 我倾向采用专门的web服务器解决，首先，这样能够使得后台服务器更加专注于业务本身(api调用)，其次，部署独立的web server更适合进行对后台api进行水平扩展。
  - 在这里，我们采用了nginx作为web服务器，nginx是一款十分优秀的web server，能够提供反向代理（域名转发）、负载均衡、静态资源服务、数据缓存等诸多功能，能够很好地完成web服务器的工作。
    - > nginx现在分为[开源版](http://nginx.org/)和[商业版](https://www.nginx.com)(plus等)，他们的核心用法都相同，并且开源版的功能以及足够满足我们的需求。

## api server
* JavaScript
  * 我决定采用js作为web后端的语言，主要出于以下几个原因：
    1. web开发本身面对的场景并不复杂，js语言的特性足够解决开发过程中的问题。
    2. 由于前段代码已经采用js编写，后端代码如果也能使用js，可以很大程度降低学习成本，在小团队中，适合全栈开发。
    3. 相对于java来说，js省去了很多模板代码，再加之es6语法辅助，用起来比较爽XD。
* [egg.js](https://eggjs.org/)
  * egg基于[koa](https://koajs.com)，采用洋葱模型中间件，对request/response进行处理。我认为egg给我带来的最大好处是**约定优于配置**的思想，按照官方的话来说，就是：
    * > 采用一套统一的约定进行应用开发，团队内部采用这种方式可以减少开发人员的学习成本，开发人员不再是『钉子』，可以流动起来。没有约定的团队，沟通成本是非常高的，比如有人会按目录分栈而其他人按目录分功能，开发者认知不一致很容易犯错。但约定不等于扩展性差，相反 Egg 有很高的扩展性，可以按照团队的约定定制框架。
  * [sail](http://sailsjs.com/)也是一个优秀的后端框架，它的思想类似于egg，采用约定优于配置，但是其依赖过于耦合，而egg可以采用插件的方式灵活拔插，因此还是选择egg。
* sequnlize
  
### 容器化
容器的理念是将应用放在具有独立资源，相互隔离的轻量级环境中运行，这样做可以带来诸多好处，我不在这里一一阐述。可以将容器看做是一个非常轻量的虚拟机，其中[docker](https://www.docker.com/)作为容器技术的优秀实现，非常适合作为api服务器的容器。通过使用docker，使得api-server能够相互独立运行，方便扩展，易于部署运行，降低运维难度等，这也是DevOps的发展趋势之一。

### 无服务化
在亚马逊推出[lambda](https://aws.amazon.com/lambda/)服务之后，[无服务技术](https://en.wikipedia.org/wiki/Serverless_computing)逐渐进入了人们的视野。web服务同样很适合采用无服务化的方式进行开发。本实践中并未采用serverless技术的主要原因是该技术还不够大众，但是我认为，serverless一定是未来趋势，也许不久之后，这篇最佳实践就会用无服务改写。


