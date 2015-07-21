---
title: Refactoring Ember Properties
author: John McDowall
layout: post
permalink: /refactoring-ember-properties/
draft: true
dsq_thread_id:
categories:
  - ember-core
---

You might have run across the situation where you have lots of properties in a Controller or Component, and they have associated Computed Properties that calculate some presence value. 

Here's a way to refactor that proliferation of property accessors using Higher Order Functions. 

## Typical Ember Example

You may have seen, or written code like this in Ember:


{%highlight javascript linenos%}
import Ember from 'ember';

let LoginComponent = Ember.Component.extend({
  userName: '', 
  password: '', 

  hasUsername: Ember.computed('userName', function() {
    return Ember.computed.notEmpty('userName');
  }),

  hasPassword: Ember.computed('password', function(){
    return Ember.computed.notEmpty('password');
  }),

  ... etc ...
});

export default LoginComponent;
{% endhighlight %}

Typically these properties are the results of requirements from the UI to display different elements on particular states. The template will have some conditionals that lean on these properties to achieve this. Depending on how complex the UI is, you could end up having a few of these state properties. 

Here's one way you can clean them up.

## Higher Order Functions 
Any function that takes another function as a parameter, or returns a function as its result is a Higher Order Function. Given that functions are first class citizens in Javascript, it's pretty easy, even casual to use Higher Order Functions to clean up functions that differ only by name and the property referenced. 

## Using Higher Order Functions to reduce cruft

Note that this is a trivial example for the purposes of showing how Higher Order Functions _can_ be used, and the example is chosen to illustrate the technique. 

And with that in mind lets look at the solution:

{%highlight javascript linenos%}
import Ember from 'ember';

let computed = Ember.computed;

let LoginComponent = Ember.Component.extend({

  UI_FIELDS: ['userName', 'password'],

  // Builds a field property name from a field name.
  _fieldPropName(field) {
    return `has${field.capitalize()}`;
  },

  // Run once on component init, sets the default empty values for the field
  // and generated the hasPropertyName propertiese for each field.
  defineFieldProperties: Ember.on('init', function() {
    this.UI_FIELDS.forEach((field) => {
      this.set(field, '');
      let propName = this._fieldPropName(field);
      Ember.defineProperty(this, propName, this.has(field));
    });
  }),

  // This is our Higher Order Function, that we are passing in the field name
  // to so that it can be captured by the Computed Property and returned as the
  // function to be called.
  has(field) {
    return computed.notEmpty(field);
  },
});

export default LoginComponent;

{% endhighlight %}

## Summary

Higher Order Functions have many uses, and this is a great example of where you can quickly apply them to clean up some duplication. You could also extract the code above into a Mixin and share it between similarly affected sections of code. 

An interesting approach would be to build a simple form validation system on this technique with some pre-canned validators that are higher order functions along the lines of the `has` function, for example `minLength`, `maxLength` and so on. Then the `UI_FIELDS` value would be expanded to an object, and each field name would have associated keys describing the validation to use, defaults, etc. 
