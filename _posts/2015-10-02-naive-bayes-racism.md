---
layout: post
title: Naive Bayes and Unknowing Racism
date: 2015-10-03
author: Aaron Norby
categories: machine-learning racism
---

__What this is about__: analyzing text using machine learning to combat our own implicit
biases. What's implicit bias? It's bias (eg, racism) that you do not consciously
endorse but that manifests itself in the way you treat others, think, talk, and
behave, sometimes very subtly and sometimes in glaring ways. 

__The tl;dr__: maybe we can categorize text that we write as being about particular
privileged groups (eg, white male) and non-privileged groups (eg, black, female)
and by what kind of language it uses, in order make statistical comparisons and
thus discover bias in our own writing. After describing the problem of implicit
bias, I present my current attempt at writing software to do just this, in JavaScript.

-----

We humans are pretty good at thinking, but sometimes we think
too fast - relying on assumptions and jumping to conclusions that, in some
contexts, we'd be better off taking a second look at. But to do
that we need to slow down. And to do that (to do it smartly, anyway) we need to
know *when* to slow down and think twice. Computers think fast, but not in the way
that we do. Computers think fast by thinking a large number of things very very quickly. Humans
think fast by taking shortcuts that allow us to reduce the *number* of things that
need to be thought (we may also think a large number of things, maybe in parallel,
but that's another post).  Humans skip steps, computers don't. In this sense,
computers always think slowly. So maybe we can use this fact to develop software
that will help tell us when we're thinking too fast - when we're relying on
assumptions that we shouldn't be. (Some people have a greater tendency to slow
down and not rely on intuitive judgments than others: see [here][slow].)

[slow]: http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2644392

Human cognition works as well as it does (which is pretty well) in no small part
because of the strategic shortcuts that the mind has evolved to employ. We don't
stop to collect every last piece of information before making a decision because
there isn't time and collecting information can be costly. So we make guesses. And
some of these guesses are hard-wired into us. But many of these guesses involve
other people: we use stereotypes to navigate the social world because we simply
cannot learn everything there is to know about a person before deciding how to
interact with them. 

Our stereotypes, however, come in no small part from our culture. And in a culture
where there are oppressed minorities who are subject to negative stereotyping, we
are going to tend to absorb those negative stereotypes. It does not matter if you
don't endorse them. Your mind absorbs them just by seeing them used by others.
So, when you see tv show after tv show, news story after news story, depicting
black men as violent criminals, there is going to be a part of your mind that turns
on whenever you encounter a black man and whispers in your ear 'that person is
dangerous'. You might *know* that that's a biased stereotype, and you might not
*want* to have any part of your mind whispering things like this to you. But the
reality is that you have only so much direct control over your own mind. (Although
that's not to say that we aren't responsible for our implicit biases; I think
the question of responsibility here is incredibly difficult - but I think it *is*
our responsibility to do what we can to correct these biases, both because they're
harmful and because they're mutually-reinforcing.)

Implicit biases are a lot like habits. You might not want to have a habit, but once
you've got it you can't get rid of it just by deciding to. That's why you keep
looking for your playlists where iTunes *used* to put them and not where iTunes
puts them now (and is also why it's so annoying when these things get moved around
and is maybe kind of a mistake for Apple to keep doing that). 

## Software to tell us when we're letting our implicit biases talk for us

If implicit biases are like habits, it would be good to have something to help us
change our habits and to let us know when our biases are driving our behaviors. 

One place where bias shows itself is in the way we write. My inspiration here is
[this article](http://www.huffingtonpost.com/john-metta/i-racist_b_7770652.html) by
John Metta. It contains what I think is probably the best example I've seen of how
implicit bias can influence one's decisions in spite of any intentions one has to
the contrary.  He points to subtle differences in language, to how "a white person
smoking pot is a 'Hippie' and a Black person doing it is a 'criminal'", and then
points out just the most perfect example you could imagine.  Here's the example,
quoting from Metta:

> There's a headline from *The Independent* that sums this up quite nicely:
"Charleston shooting: Black and Muslim killers are 'terrorists' and 'thugs'. Why
are white shooters called 'mentally ill'?"

Metta continues:

> Did you catch that? It's beautifully subtle. This is an article talking
> specifically about the different way we treat people of color in this nation and
> even in this article's headline, the white people are "shooters" and the Black
> and Muslim people are "killers."

There's an awful lot that one could say about the psychology of implicit bias (see
[here][harvard] or [here][project]), but this example pretty well captures what
makes it so hard to notice in oneself.  Implicit biases are a lot like habits. You
might not want to have a habit, but once you've got it you can't get rid of it just
by deciding to. This is why you reach for your keys where you used to leave them
even though you know you put them in a new place now. 

[harvard]: https://implicit.harvard.edu/implicit/research/

[project]: http://www.biasproject.org/

Your explicit beliefs might be entirely egalitarian, but if you don't want to use
biased language, you need to be able to catch yourself in the act so that you can
correct yourself. This is particularly important if you're writing for a public
audience, because the use of biased language results in reinforcing those
stereotypes in your audience when they read your work. 

To get to the point, then, I want to build something that will take a text, figure
out which sentences are about white people and which sentences are about oppressed
or disadvantaged groups, and tell you if you use nicer language to describe white
people than you do others. 

So, the problem has three subproblems: 

1. Classify text by racial content.

2. Classify text as using positive/negative language.

3. Look for differences in (2) relative to (1). 

I should say at the outset that I have not entirely figured this out yet. What I
describe below is a work in progress, using a Naive Bayes text classifier,
sentiment analysis, and a simple check for differences in means. So, if you like
the basic project but have some ideas about how to do it better, I very much would
like to hear about it. 

In this post, I'm going to describe the work I've done on (1). In later posts I
plan on talking about (2) and (3) as well as how I get my training data using the
New York Times API, and what training data might be better suited to my purposes.
But, basically, (2) involves using sentiment analysis to look at the groups of
sentences categorized by (1), and (3) involves simply comparing means. 


## Classifying text


So, here's the problem. How can a computer know whether a sentence containing the
word 'white' is about white people or the color white or the band The White
Stripes, or whatever else? If it can't make those distinctions, later analyses that
are based on the assumption that those sentences classified as being about white
people will be meaningless because they'll be completely dominated by noise.  

There are many approaches that one can take to text classification using machine
learning, and the options in JavaScript are somewhat limited (other languages like
Python and R have much bigger communities dedicated to this stuff). But JavaScript
does have some good options.  The one I'm using is a Naive Bayes classifier
provided by the [Natural][natural] package. 

[natural]:https://github.com/NaturalNode/natural

If you've never heard of Naive Bayes, what makes it 'naive' is that it makes a
strong independence assumption about the structure of your data (what Bayesian
prediction in general is, is a topic for another post). A Bayesian classifier is
trying to make predictions about data its never seen before by using attributes of
that data, on the basis of the relationship its already seen between those same
kinds of attributes and past data. For example, maybe a classifier is trying to
predict the voter registration status of people on the basis of what state they
live in and their income (after having already been given a large dataset with
other people's voter status, income, and state). Naive Bayes assumes that income
level and state of residence are independent from each other - that you get no
predictive power for one by knowing the other. Of course, they aren't actually
independent. But making this assumption greatly reduces the complexity of the
computations that need to be done. Indeed, it renders otherwise intractable
problems computationally feasible. And in many cases, including that of text
classification, the assumption, though false, allows for high predictive accuracy.
(See, for example, the explanations offered [here][stanford].)

[stanford]:https://web.stanford.edu/class/cs124/lec/naivebayes.pdf

Basically, it works like this. I get a lot of sentences from news sources (in this
case, the New York Times) and then label them by topic (more later on ways to do
this). This is the training data. Because I want my classifier to be able to
classify sentences as being about white people, black people, or neither, the
sentences are given these labels (I'm right now just doing white vs black in order
to contain the scope of the project.) Again, there should be questions here: the
subject of a sentence is almost never a clear-cut matter; the hope is that with
enough training data, ambiguity in classification will be minimized.  

Labeled training data look like this (in a file called 'data.json'): 

~~~javascript
{
  'This is a sentence about white people': 'w',

  'This is a sentence about black people': 'b',

  'This is a race-neutral sentence': 'n'
}
~~~

The classifier is then given this training data to train on, like so: 

~~~
var natural = require('natural');

var classifier = new natural.BayesClassifier();
var data = require('./data.json');

var trainingData = JSON.parse(data);
for (var sentence in trainingData) {
  classifier.addDocument(sentence, trainingData[sentence]);
}

classifier.train();
~~~

Each sentence counts as a 'document' for the classifier, so you add all of your
documents/sentences through the `addDocument` method and then call `train` to
actually train the classifier on that training data. You can then save the trained
classifier by calling `classifier.save('classifier.json', callback)`.
'`classifier.json`' is what the output file is going to be called, and the callback
is given two arguments, the first any error in saving and the second is the
classifier itself. 

Later on, you can bring that already-trained classifier back by calling
`natural.BayesClassifier.load('classifier.json', null, callback)`. Again the
callback takes any error encountered as the first argument and the classifier as
the second.  (Where `null` has been passed in it can instead accept the name of the
word stemmer that one originally specified, if one doesn't use the default). 

Of course, there's no point in saving a classifier that doesn't work, so you have
to test it on some data that it's never seen before. Unfortunately, Natural doesn't
provide testing features, so you have to do it yourself. You can automate it, or
just do it by hand.

In either case, you get the classifier to classify by using

~~~
classifier.classify('example sentence');
~~~

If you have a load of labeled testing data in an object (not the data you trained
on), you could use this to find out how good your classifier is just by counting
the percentage of labels it gets right.

You can also use 

~~~
classifier.getClassifications('example sentence');
~~~

and it will return numerical weights associated with each label, telling you how
confident it is about different classifications. 

Here we come to the problem. For a text classification task like the one I'm
trying, you need *a lot* of training data. A lot means that just two- or
three-hundred sentences is not enough.  Most recently, I labeled the first sentence
of every article in the New York Times in the last year that comes up in a search
with the keyword "Ferguson", and it was not enough data at only about 260
sentences. By 'not enough' I mean it thought both those example sentences from
Metta's article were about white people. So, you need a lot of data. 

This is probably a good lesson. Right now, despite all this fancy machine learning
junk, I would do better to simply classify by whether a sentence simply contains
the word "white" or the word "black". 

Of course, how well a classifier classifies depends entirely on the training data.
And here things get hard. This is a point at which I'd be more than happy to hear
suggestions.  I've been training on sentences from the New York Times because new
sources are an important point at which biases can propagate and be reinforced and
'journalistic' writing is common to both newspapers and many blogs. But there are a
lot of obvious problems. Is a sentence about a white police officer shooting a
black teenager about white people or about black people? I've taken the route of
giving these sorts of sentences both labels. But what about sentences that are
clearly about a black group or black issues - like a sentence about the Emanuel
African Methodist Episcopal Church - but that don't explicitly talk their subjects'
race? On the one hand, such sentences clearly have racial content; but from the
perspective of machine learning, it may not be helpful to key the label 'black' to
particular proper names. Of course, one of the points of machine learning is that
we humans are supposed to be able (to some degree) to ignore these things and let
the computers extract whatever patterns best fit the data, and any overfitting that
might result from keying to proper names should be taken care of with a large
enough data set.  That, anyway, is how it's supposed to go. In theory.


