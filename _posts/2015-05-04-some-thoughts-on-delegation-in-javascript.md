---
layout: post
title:  "Some thoughts on delegation in JavaScript"
date:   2015-05-04 16:16
categories: update
---

Recently I've been thinking that not only is [Platonic inheritance](https://twitter.com/debrisapron/status/595734720102694915) generally not that helpful for expressing the ideas I want to express, pretty much _all_ the various ways of implicitly copying unnamed methods into an object - prototypal inheritance, Ruby mixins, `_.extend` in lodash/Underscore etc - obfuscate my code and don't give me a lot in return.

In my own recent JavaScript practice I have ditched prototypal inheritance almost entirely. I can see myself using it as a performance optimisation where I feel the overheard of creating large numbers of unlinked objects is unacceptable, but that's it. Instead I use the (to me) conceptually much simpler toolkit of factory methods, closures, object literals and delegation.

Here's some pretty conventional code to construct an object derived from Node's [EventEmitter](https://nodejs.org/api/events.html):

{% highlight js %}
var util = require('util');
var events = require('events');

function PingEmitter() {}

util.inherits(PingEmitter, events.EventEmitter);

PingEmitter.prototype.ping = function () {
  this.emit('ping');
};

PingEmitter.prototype.pingTwice = function () {
  this.ping();
  this.ping();
};
{% endhighlight %}

We can use this constructor very easily by calling `new PingEmitter()` and the returned object works fine, we can call `ping()` and `pingTwice()` and it emits the events. However there are a number of problems here.

Firstly, exposing an object constructor seems to me to break encapsulation: how we generate our PingEmitter objects is surely an implementation detail, not an inherent property of our API. When I'm consuming an API I just want a function to give me an object, I don't care how it's being created internally (and maybe it's not being created, maybe it's being fetched from a pool or something).

Secondly, you force consumers of MyEmitter to use `new` or `Object.create`. More typing, more line noise, more cognitive overhead.

Thirdly, you are blindly copying all the methods of EventEmitter into your new object. That means that, for example, consumers can use your `PingEmitter` to emit pongs by calling `pingEmitter.emit('pong')`. If I wanted my PingEmitter emitting pongs, I'd have given it a pong method.

Fourthly, methods of your object that call each other have to use `this` to specify they are calling the message on themselves. The semantics of `this` are [famously confusing](https://www.google.com/search?q=how+does+this+work+in+javascript) and add one more cognitive overhead to the already difficult process of reading JavaScript.

Fifthly, it is well minging.

## A better way maybe

A solution which fixes all these problems and IMHO produces much more readable code is to use a factory function and expose only the methods of the EventEmitter object that you actually want your consumers to use, like so:

{% highlight js %}
var events = require('events');

function pingEmitter() {
  var eventEmitter = new events.EventEmitter();

  function ping() {
    eventEmitter.emit('ping');
  }

  function pingTwice() {
    ping();
    ping();
  }
  
  return {
    ping: ping,
    pingTwice: pingTwice,
    on: eventEmitter.on
  };
}
{% endhighlight %}

Now we can just call `pingEmitter().ping()` and... nothing happens. Or maybe you get an error message. This is because EventEmitter internally is using `this`, but now its `this` is referring to our newly created object, which is obviously missing a lot of stuff EventEmitter needs.

To fix this problem, I use the `bindAll` function from the marvellous [lodash](https://lodash.com/) library. This function binds all an object's methods to itself, ensuring you can use them from outside the object without errors. So:

{% highlight js %}
var events = require('events');
var _ = require('lodash');

function pingEmitter() {
  var eventEmitter = _.bindAll(new events.EventEmitter());
  
  function ping() {
    eventEmitter.emit('ping');
  }
  
  function pingTwice() {
    ping();
    ping();
  }
  
  return {
    ping: ping,
    pingTwice: pingTwice,
    on: eventEmitter.on
  };
}
{% endhighlight %}

And it works fine. This for me not only solves all the problems I listed above, it's very readable and makes it painfully explicit which methods are being delegated where.

A couple of downsides that I can see: It's a little bit more verbose (although some of that verbosity is reduced by [the new object literal notation in ES6](http://ariya.ofilabs.com/2013/02/es6-and-object-literal-property-value-shorthand.html)) and you can't call your pingEmitter objects `pingEmitter` because that's the name of the factory function. Would be nice to start my factory method names with a capital letter to avoid this, but every JS linter (and programmer) in the world expects capitals to denote constructors.

None of this seems particularly controversial to me but I'm kind of an eternal n00b when it comes to this stuff, so if any clever JavaScript types want to tell me where I'm going wrong I'd be very interested to hear.