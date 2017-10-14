---
layout: post
title: Study Notes of React Tutorial
date: 2016-03-05
categories: Javascript
---

> The Link of React Tutorial: http://facebook.github.io/react/docs/tutorial.html

Above is the tutorial which told me that I need to setted up a node.js server

##  About installing the Node

npm install
node server.js
By the command above, I can start a Node.JS server on 127.0.0.1:3000

I have not used any node.js server anymore, so I searched some topic about what the "npm" it is.

The link of the usage of npm: http://www.tutorialspoint.com/nodejs/nodejs_npm.htm

For my MAC, I use brew to install node.js, the command is:

brew install nodejs
The GIT repo address of my learning is: https://git.coding.net/pipicold/react-test.git


## About the fundamental knowledge

To create a React component: `React.createClass()`

While creating a component, a important method is

`render: ()`

In this method, you can added some codes for when your component is firstly added.

You do not have to return basic HTML. You can return a tree of components that you (or someone else) built.

`ReactDOM.render()` instantiates the root component, starts the framework, and injects the markup into a raw DOM element, provided as the second argument.

![first render](/img/2016-03-05/1.png)


The class name of a component equals the name of the tag

example:

If I defined a class named "CommentBox" by the code:


```JavaScript
var CommentBox = React.createClass();
```
I should call this component by:

```JavaScript
ReactDOM.render(        <CommentBox />,        document.getElementById("content"))
```

## Composing Components



I found that maybe React cannot divide the code into separated files

Firstly, I wrote 3 files

1. tutorial2.js

```Javascript

var CommentList = React.createClass(   
    {           
        render: function(){         
            return(         
               <div className="commentList">     
                    Hello, I am a CommentList      
               </div>        
            );
        }
    }
);
var CommentForm = React.createClass(
    {            
        render:function(){                
            return(                        
                <div className = "commentForm">                        
                    Hey, I am a Comment Form                      
                </div>
            );            
        }        
    }        
);

```

2. tutorialall.js

```JavaScript
var CommentBox = React.createClass(
        {
render: function(){                
return (                        
        //'className' means 'class' in standard HTML
        <div className = "commentBox">                        
        Hello pipicold                        
        <h1>Comments</h1>
        <CommentList />                        
        <CommentForm />                        
        </div>                      
       );  
}        
}        
);
ReactDOM.render(
        <CommentBox />,      
        document.getElementById('content')        
        );

```

3. index.html

```HTML
<!DOCTYPE html>     <html>  

<head>    

<meta charset="utf-8">

<title>React Tutorial</title><br><!-- Not present in the tutorial. Just for basic styling. -->    

<link rel="stylesheet" href="css/base.css" />    
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.14.7/react.js">
</script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.14.7/react-dom.js"> </script>    

<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.6.15/browser.js"> </script>    

<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.2.0/jquery.min.js"> </script>    

<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.5/marked.min.js"> </script><br></head>  

<body>    

<div id="content"> </div>    

<script type="text/babel" src="scripts/tutorial2.js"> </script>    

<script type="text/babel" src="scripts/tutorialall.js"> </script><br></body> </html>

```


It will not work.

After I merged those two JS file into one, the page went to normal.

Using props

the braces seems to mean the call of properties

```HTML

<div className="comment">
<h2 className="commentAuthor">
{this.props.author}  
</h2>  
{this.props.children}       
</div>

```

what should be came up with is that the `{this.props.children}` means the elements between the tags: `<Comment> </Comment>`



## Component Properties

Pass the properties like original HTML


```HTML
render: function(){                
    return(                        
            <div className="commentList">                        
            <Comment author="Mrs. Wu"> I do not like baby</Comment>
            <Comment author="Mr. Wu"> I don't like baby</Comment>  
            </div>
          );            
}
```


By the way, I found that it is unimportant for the sequence of each Component, you can put a Component at any where but should before the line :

```JavaScript
ReactDOM.render();
```


Besides, you can use any characters between tags in JSX

The rendered page showed like follow:

![first render](/img/2016-03-05/3.png)

## Adding Markdown

http://facebook.github.io/react/docs/tutorial.html#adding-markdown



Marked is a external library wirtten in JS

offical usage in its github website: https://github.com/chjj/marked

```HTML
<!doctype html>
<html>
<head>
  <meta charset="utf-8"/>
<title>Marked in the browser</title>
  <script src="lib/marked.js"></script>
</head>
<body>
 <div id="content"></div>
  <script>    document.getElementById('content').innerHTML =      marked('# Marked in browser\n\nRendered by **marked**.');  </script>
</body>
</html>
```

Whlie applying it to React, just us the braces to run your JS code

the content in the braces means the JS code, outside of that is the HTML part

besides, you cannot use ";" in the return, reason:

http://blog.csdn.net/lihongxun945/article/details/45826851

>因为 React.createElement(“div”, null, var a = 1;) 是语法错误。

>那么你也可以理解为什么大括号中的js表达式不能有分号结尾了吧。


>需要注意的是，如果你在属性中输出JS变量，是不能加引号的，不然会被当做字符串而不被解析。

>应该是这样：


the renderred page is like this:


![first render](/img/2016-03-05/2.png)


according to the description of React's tutorial

That's React protecting you from an XSS attack. There's a way to get around it but the framework warns you not to use it:

two steps:

1. using html: tag

```JavaScript
rawMarkup: function() {
    var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return { __html: rawMarkup };
  },
```

2. using span tag

```HTML
<span dangerouslySetInnerHTML={this.rawMarkup()} />
```

caution! you must use comma to separate two functions

```JavaScript

var Comment = React.createClass(
        {
            rawMarkup: function()
            {
                var rawMarkup = marked( this.props.children.toString(), {sanitize:true});
                return {__html: rawMarkup};
            },//be careful of this comma!!
            render: function(){
                return(
                        <div className="comment">
                            <h2 className="commentAuthor">
                                {this.props.author}
                            </h2>
                            <span dangerouslySetInnerHTML={this.rawMarkup()}/>
                        </div>
                        );
            }
        }
        );

```


## Hook up the data model

It is so fun and free while using function in JavaScript:



http://www.jb51.net/article/22742.htm



再有一个值得说一下的，就是JS中的函数的参数不一定是严格匹配的，通常的编程经验，比如有这样一个函数 fun(aa,bb)，那么我们在调用这个函数的时候就应该给他传递两个参数。但是在JS中，我们可以给他传递任意个参数，1个，3个，等等，都可以。JS中的参数传递，不完全是按照函数声明时指定的那些参数，在每次调用函数的时候，都会有一个命名为arguments的数组，这个数组里面存储了函数调用时，传递进来的所有参数，有了它，我们甚至可以不再函数声明时指定形式参数

## Reactive state

 To implement interactions, we introduce mutable state to the component. this.state is private to the component and can be changed by calling this.setState(). When the state updates, the component re-renders itself.





## Adding new comments


it is so hard for me to debug in React, maybe I should find a way to debug it.

also, I was not so familir with the "function" of Javascript, I should learn about it



It is so tired for me, because it is already 1.30 am now...I must go to bed...

There is an extension for debuging React, on chrome.
