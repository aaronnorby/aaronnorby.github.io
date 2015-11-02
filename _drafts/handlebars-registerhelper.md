---
layout: post
title: Use Handlebars registerHelper to reformat template dates
date: 2015-10-20
author: Aaron Norby
categories: templates handlebars
---

Handlebars is a great, straightforward and easy-to-use templating language. It's
*very* easy to use when the data you're feeding into it is already formatted in the
way you want to display it. For example, if you have dates that are in the
YYYY-MM-DD order you want to show them in. But it's a bit more complicated if you
want to move things around inside the template before rendering them. Say you
have an object like this: 

~~~javascript

var data = {
  name: 'Aaron',
  species: 'human',
  date: 'Tue, Oct 10 2015 13:01:34 -300'
}
~~~

If you don't want to move anything around you can use Handlebars like this to
display that info in your html: 

~~~javascript
var template = Handlbars.compile(
  {% raw %}
  '<h2>{{name}}</h2>' +
  '<h2>{{species}}</h2>' +
  '<h2>{{date}}</h2>');
  {% endraw %}
var html = template(data);
~~~

That's pretty great. But maybe you don't want the date to be in such a long,
esoteric form. Like maybe you want the public to see this and have them not think your
website is only for tech weirdos.  

To format as you template, Handlebars provides the `registerHelper` method. And to
specifically formate dates, we can use the versatile [Moment.js][momentlink] library in
conjunction with `registerHelper`.  Here's how. 

[momentlink]: http://momentjs.com/

The basic way you use `registerHelper` is you create a keyword that you put inside
the template strings you want to encode, and this keyword points to the function
that does the reformatting. 

Here's an example, which we'll continue filling out when we add Moment.js: 

~~~javascript
<script>
  Handlebars.registerHelper('dateformat', function(date) {
    return date + " I added to the date!";
  });
</script>
~~~

This won't really reformat your date so much as add a pointless string to the end
of it, but it demonstrates the `registerHelper` pattern and makes it pretty clear
the open-endedness of the kinds of transformations that are possible.  

Here's how we'd use it in our template from before: 

~~~javascript
var template = Handlbars.compile(
  {% raw %}
  '<h2>{{name}}</h2>' +
  '<h2>{{species}}</h2>' +
  '<h2>{{dateformat date}}</h2>');
  {% endraw %}
var html = template(data);
~~~

That's all there is to it: just put the pointer to the function in the template
string. Then, if your date was `Dec 5, 2015`, this would output `Dec 5, 2015 I
added to the date!`. 

Let's add in Moment.js to actually make the date look better. Moment.js has a ton
of date-reformatting options (and lots of other time-related methods). Dealing with
times is not exactly the most enthralling part of programming, but it's pretty much
inescapable. And sometimes they even have to be human-readable. So here's an
example of how to do that with Moment.js and Handlebars. The format that the date
comes in from the server is `Sun, 1 Nov 2015 04:41:14 -6:00`. We want to display it
as `November 1st, 2015 4:41 pm`. So we put the following in our `registerHelper`
(after including Moment.js in our loaded scripts, of course):

~~~javascript
<script>
Handlebars.registerHelper('dateformat', function(date) {
  return moment(date, "ddd, D MMM YYYY HH:mm:ss Z").format("MMMM Do, YYYY h:mm a"); 
});
</script>
~~~

And that's all there is to it. Times formatted on the fly in templates. Super
useful. 

