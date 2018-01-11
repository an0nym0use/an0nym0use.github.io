---
date: 2015-09-18T00:00:00Z
title: Meteor Tips - UI Hooks woopsies
url: meteor-tips-uihooks-woopsies
---

Meteor's undocumented `_uihooks` feature is pretty awesome. It gives us the ability to hook into the DOM manipulation functions that we normally get for free with reactive templating. Sometimes we need to control how things get rendered, for example if elements need entry or exit animations. [Here is a great article][discovermeteor-post] that gives an overview on `_uihooks`.

Unfortunately, as happens with life on the bleeding edge, undocumented features change, and there is no authoritative place to check out why your working code is now breaking. This happened recently when the API for `_uihooks` changed.

Previously you could use `_uihooks` like so:

```javascript
Template.container.onRendered(function(){
  this.find('.list-wrapper')._uihooks = {
    insertElement: function(node, next){
      // Do things before inserting
      $(node).insertBefore(next);
      // Do things after inserting
    }
  };
});
```

Where `node` was the new node to be inserted, and `next` was the element that the new node needed to be inserted before. Meteor used to render empty text nodes in all templates, thereby always having an element to insert our new node before. When this changed, upon inserting a node as the last child in the container, `next` would be `null`. A possible way to change this is like this:

```javascript
Template.container.onRendered(function(){
  var container = '.list-wrapper';
  this.find(container)._uihooks = {
    insertElement: function(node, next){
      // Do things before inserting
      if(_.isNull(next)){
        $(container).append(node);
      } else {
        $(node).insertBefore(next);
      }
      // Do things after inserting
    }
  };
});
```

Alternatively, there is this great package: [https://github.com/gwendall/meteor-ui-hooks][uihooks-package].

Happy rendering!


[discovermeteor-post]: https://book.discovermeteor.com/chapter/animations
[uihooks-package]: https://github.com/gwendall/meteor-ui-hooks
