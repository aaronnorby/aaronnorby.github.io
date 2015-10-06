---
layout: post
title: How to use D3 with Angular
date: 2015-10-06
author: Aaron Norby
categories: Angular D3
---

How to use D3 with Angular

You can make great-looking stuff with Angular, and you can make great-looking stuff
with D3. You can also make great-looking stuff with Angular and D3 together. Here's
a relatively painless way to make them work with each other. 

The simplest way to put D3 visualizations into an Angular web application is by
creating a custom Angular directive. For those who don't know already, directives
in angular are (at least in the way we'll be using them) effectively custom-made
html elements. So, instead of putting your D3 inside of a `div`, you'll put it
inside of something that you create, like: 

~~~javascript
<awesome-d3></awesome-d3>
~~~

How do you create custom directive? Suppose you've got your Angular app defined,
like so: 

~~~
var greatApp = angular.module('greatApp', []);
~~~

We're assigning our app to a variable so that we add our directive on to it in
another file if we so choose, to keep our code clean and modular. Here we define
our directive: 

~~~
greatApp.directive('awesomeD3', [function () {
  ...
}]);
~~~

Where the dots go our directive-making code will go. Note that the string naming
our directive is `awesomeD3`, in camelcase, while our tag-name we used earlier was
`awesome-d3` with a dash. Your camelcase directives will be converted to dash- (or
snake-) when Angular runs through the html it controls. 

Now we actually put some filling in our directive: 

~~~
greatApp.directive('awesomeD3', [function () {
  restrict: E
}]);
~~~

This `restrict` makes it so you can only use your directive as an element, as we're
doing. You can loosen or change these restrictions and allow attributes, classes, etc.
Continuing: 

~~~javascript
greatApp.directive('awesomeD3', [function () {
  restrict: E,

  link: function(scope, element, attributes) {
    var svg = d3.select(element[0])
          .append('svg')
          .attr('height', 500)
          .attr('width', 500);
    
    svg.selectAll('circle')
      .data(scope.data)
          .enter()
          .append('svg:circle')
          .('r', 9);
  }
}]);
~~~

The `link` function is really where you do everything. This is how your directive
is defined - its appearance, its behavior, and so on. Here we're just doing
some standard, boring svg circles with basic D3 methods.  

The real questions here are: what are those parameters to the `link` function, 
where is that data coming from, and why are we doing `element[0]` instead of just
`element` to append our svg? 

First, the `scope` argument. `scope` is the, well, scope object associated with your directive's
element. You can use it in the way you associate models with the `$scope` object in
controllers. To do so, you need to add something to your html and you need to add
something to your directive definition. First, the html: 

~~~
<awesome-d3 data="dataset"></awesome-d3>
~~~

(Note that the 'data' attribute doesn't have to be called 'data'. Call it whatever
you want.) You use it by defining your scope in your directive, like this: 

~~~
greatApp.directive('awesomeD3', [function () {
  restrict: E,

  link: function(scope, element, attributes) {
    var svg = d3.select(element[0])
          .append('svg')
          .attr('height', 500)
          .attr('width', 500);
    
    svg.selectAll('circle')
      .data(scope.data)
          .enter()
          .append('svg:circle')
          .('r', 9);
  },

  scope: {
    data: '='
  }
}]);
~~~

And suppose that in your controller (the controller whose scope encloses your
custom directive in the html document) you have this: 

~~~
greatApp.controller('greatController', ['$scope', function($scope) {
  $scope.dataset = {'heresData': 123};
}]);
~~~

This makes the data referred to by `dataset` available to your directive, via
`data`. So, the scope object can function as a conduit through which information
can be passed from your controller's models to to directive. By including `data:
'='`, your directive gains access to the `dataset` model because of the equation we
put in our html. 

Then, you can use it in your directive: 

~~~
greatApp.directive('awesomeD3', [function () {
  restrict: E,

  link: function(scope, element, attributes) {
    var svg = d3.select(element[0])
          .append('svg')
          .attr('height', 500)
          .attr('width', 500);
    
    svg.selectAll('circle')
      .data(scope.data)
          .enter()
          .append('svg:circle')
          .('r', d.heresData);
  },

  scope: {
    data: '='
  }
}]);
~~~

Now, you're setting the radius of your svg circle to whatever `dataset.heresData` is.
This is a pretty mundane use, but you can imagine the possibilities. 

Now what about `element[0]`? Not surprisingly, `element` in the link function
refers to your directive's element. But it's wrapped in a jQuery-lite (Angular's
version of jQuery) object, so to get the actual element you have to get it from a
jQuery-like array. Not too complicated, but an easy thing to forget to do. 

The good thing about having the element jQuery(-lite)-wrapped is that you can use
jQuery-lite methods to manipulate it. For example, you can add styles like so:

~~~
element.css({'margin': '20px'});
~~~

This code would be inside the link function so that you have access to `element`.
Note that in this case you *don't* try to grab it out of an array. 

And that, as far as the basics, is it. D3 loves to display data, and Angular loves to share
data with things that love to display data. 

If you'd like to see a working example, you can check out my code for a project
[here](https://github.com/aaronnorby/riskField) on GitHub, which produces something
that looks like this: 

![riskField](/assets/riskFieldScreenshot.jpg)




