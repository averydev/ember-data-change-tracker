# ember-data-change-tracking

[![Build Status](https://secure.travis-ci.org/danielspaniel/ember-data-change-tracker.png?branch=master)](http://travis-ci.org/danielspaniel/ember-data-change-tracker) [![Ember Observer Score](http://emberobserver.com/badges/ember-data-change-tracker.svg)](http://emberobserver.com/addons/ember-data-change-tracker) [![npm version](https://badge.fury.io/js/ember-data-change-tracker.svg)](http://badge.fury.io/js/ember-data-change-tracker)

This addon aims to fill in the gaps in the change tracking that ember data does now. 
 - Currently ember-data tracks changes for numbers/strings/date/boolean attribute,
  and has a changeAttributes method to see what changed 
 
 - This addon tracks:
    - modifications in attributes that are object/json
    - replacement of belongsTo association models 

## Installation

* `ember install ember-data-change-tracking`

## Why?
    
    Say there is a user model like this:

```javascript
  export default Model.extend({
       name: attr('string'),
       info: attr('object'),
       json: attr(),
       company: belongsTo('company', { async: false, polymorphic: true }),
       profile: belongsTo('profile', { async: true }),
   });
```

   And you have a user with an object attribute this:

```javascript
  let company = //=> company model 
  let company2 = //=> differnt company model 
  let info = {foo: 1) // some object
  let user = //=> user model where info => info  and company => company
```

#### changed() method added to models
  -  Shows you any changes in the info object 
    - whether modified or replacing type attribute  
  - Shows when you replace a belongsTo association
   
Example: ( modify attribute ) 
```javascript
  info.foo = 1                // or
  user.set('info.foo', 1);   // same idea
                      //    old value, new value      
  user.changed().info //=> [{foo: 1),  {foo: 2)] 
```

Example: ( replace attribute ) 
```javascript
  user.set('info', {foo: 2}); 
                      //    old value, new value      
  user.changed().info //=> [{foo: 1),  {foo: 2)] 
```

Example: ( replace belongTo ) 
```javascript
  user.set('company', company2);  
                        //    old value, new value      
  user.changed().company //=> [company,  company2] 
```
   
## Serializer extras
  - Mixin is provided that will allow you to remove any attributes/associations
   that did not change from the serialized json
  - Useful when you want to reduce the size of a json payload 
   - removing unchanged values can be big reduction at times
   
 Example:  
 
  Let's say you set up the user model's serializer with keep-only-changed mixin  
  
 ```javascript
  // file: app/serializers/user.js
  import DS from 'ember-data';
  import keepOnlyChanged from 'ember-data-change-tracker/mixins/keep-only-changed';
 
  export default DS.RESTSerializer.extend(keepOnlyChanged);
 ```
   
  Then when you are updating the user model 
 ```javascript
  user.set('info.foo', 1);
  user.serialize(); //=> '{ info: {"foo:1"}'
  
  // without this mixin enabled the json would look like:
  // '{ name: 'dude', info: {"foo:1"}, company: "1" companyType: "company"', profile: "1"}'  
 ```
