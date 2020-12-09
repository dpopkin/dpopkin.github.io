---
layout: post
title:  "Prototype Pollution"
date:   2020-12-07 17:32:00 -0800
categories: JavaScript Vue.js
---

And now for something completely different (JavaScript objects!). Copying some text from the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object):
> "Nearly all objects in JavaScript are instances of Object; a typical object inherits properties (including methods) from Object.prototype, **although these properties may be shadowed (a.k.a. overridden)**."

Now consider this code snippit:
{% highlight javascript %}
let object = {};
// Not sorry.
Object.setPrototypeOf(object, {"generalKenobi":"Hello There."});
console.log(object.generalKenobi); // Output = "Hello There"
console.log({}.generalKenobi); // Output = undefined
{% endhighlight %}

Put simply, this creates an empty JS object and then sets a prototype value called `generalKenobi` which is inherited by the local object for easy access. While it might seem like a great way to simplify code, Something to note is that it's generally not a good idea to add or overwrite prototypes for two reasons. First, because it might incur a performance penalty and two, also from the [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf):

> [...] In addition, the effects of altering inheritance are subtle and far-flung, and are not limited to simply the time spent in the Object.setPrototypeOf(...) statement, but may extend to **any** code that has access to any object whose [[Prototype]] has been altered.

Heres another example with a slighty different syntax: 
{% highlight javascript %}
let object = {}
object.__proto__.generalKenobi = "Hello There.";
console.log(object.generalKenobi); //Output = "Hello There."
console.log({}.generalKenobi); // Output = "Hello There."?!
{% endhighlight %}

Uh, what gives?! Well it turns out using the `__proto__` method overrides the global JavaScript object in this context. Meaning this method is accessable to **all** objects and even some other primitives (try it with a number! It works!). Can use use this to override functions? Yes.

{% highlight javascript %}
let object = {}
object.__proto__.toString = ()=>{alert("Hello There.")};
console.log(object.toString); //Output = "Hello There." in a alert box.
console.log({}.toString); // Output = "Hello There." also in a alert box?!
{% endhighlight %}

This is known as prototype pollution, which can best be defined as a prototype method or value being added or overwritten and can spell major issues for application logic.

Consider the following merge operation for objects in JavaScript (taken from [here](https://medium.com/node-modules/what-is-prototype-pollution-and-why-is-it-such-a-big-deal-2dd8d89a93c)): 
{% highlight javascript %}
merge(currentData, incomingData) {
    for(var attr in incomingData) {
        if(typeof(currentData[attr]) === "object" && typeof(incomingData[attr]) === "object") {
            this.merge(currentData[attr], incomingData[attr]);
        } else {
            currentData[attr] = incomingData[attr];
        }
    }
        return currentData;
},
{% endhighlight %}

Now imagine this is what calls the method (assume taskData is just a generic object containing a list of tasks):
{% highlight javascript %}
task = {"Priority": "Pick up apples at grocery store."}
this.merge(this.taskData, task);
this.userData = { userNumber: this.currentAPIId};
{% endhighlight %}

Guess what. its vulnerable to prototype pollution because it'll accept `__proto__`. So lets craft a scenario in which we can exploit this. let's assume there's something in the front-end application logic that toggles hiding a delete task button to a `canDeleteTask` value in the `userData` object. Also assume that due to a bug, `canDeleteTask` is unassigned, but due to `undefined === false` equalling false, it seems fine to the dev. So with that in mind:

{% highlight javascript %}
task = {"__proto__": {"canDeleteTask":true}}
this.merge(this.taskData, task);
this.userData = { userNumber: this.currentAPIId};
{% endhighlight %}

And we have the ability to delete buttons. As always, I've implemented an example of this in [DVLA](https://github.com/dpopkin/DVLA).