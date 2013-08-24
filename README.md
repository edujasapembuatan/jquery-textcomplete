Autocomplete for Textarea
=========================

Introduces autocompleting power to textareas, like a GitHub comment form has.

How to use
----------

Note: The key words "MUST", "SHOULD", and "MAY" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

-----

jQuery MUST be loaded ahead.

```html
<script src="path/to/jquery.js"></script>
<script src="path/to/jquery.textcomplete.js"></script>
```

Then `jQuery.fn.textcomplete` is defined. The method MUST be called for textarea elements, and the receiver SHOULD include an element.

```js
$('textarea').textcomplete(strategies);
```

The `strategies` is an Object not an Array. Each value is called a strategy object and the corresponding key is the name of the strategy.

```js
var strategies = {
  name: strategy
};
```

The `strategy` is an Object which MUST have `match`, `search` and `replace` and MAY have `index`, `maxCount` and `template`.

```js
var strategy = {
  // Required
  match:    matchRegExp,
  search:   searchFunc,
  replace:  replaceFunc,

  // Optional
  index:    indexNumber,
  maxCount: maxCountNumber,
  template: templateFunc
}
```

The `matchRegExp` and `indexNumber` MUST be a RegExp and a Number respectively. `matchRegExp` MUST contain capturing groups and SHOULD end with `$`. `indexNumber` defaults to 2. The word captured by `indexNumber`-st group is going to be the `term` argument of `searchFunc`.

```js
// Detect the word starting with '@' as a query term.
var matchRegExp = /(^|\s)@(\w*)$/;
var indexNumber = 2;
```

The `searchFunc` MUST be a Function which gets two arguments, `term` and `callback`. It MUST invoke `callback` with an Array of String. It is guaranteed that the function will be invoked exclusively even though it contains async call.
The `maxCountNumber` MUST be a Number and default to 10. Even if `searchFunc` callbacks with large array, the array will be truncated into `maxCountNumber` elements.

```js
var searchFunc = function (term, callback) {
  $.getJSON('/search', { q: term })
    .done(function (resp) {
      // Resp must be an Array of String such as:
      //   ['hello', 'world']
      callback(resp);
    })
    .fail(function () {
      // Callback must be invoked even if something went wrong.
      callback([]);
    });
};
```

The `templateFunc` MUST be a Function which gets and returns a string. The function is going to be called as an iteretor for the array given to the `callback` of `searchFunc`.

```js
var templateFunc = function (value) {
  return '<b>' + value + '</b>';
};
```

The `replaceFunc` MUST be a Function which gets and returns a string. It is going to be invoked when a user will click and select an item of autocomplete dropdown.

```js
var replaceFunc = function (value) { return '$1@' + value + ' '; };
```

The result is going to be used to replace the textarea's text content using `String.prototype.replace` with `matchRegExp`:

```js
textarea.value = textarea.value.replace(matchRegExp, replaceFunc(value));
```

Example
-------

```js
var emojies = ['+1', '-1', 'dog', 'cat'];

$('textarea').textcomplete({
  // mention strategy
  mention: {
    match: /(^|\s)@(\w*)$/,
    search: function (term, callback) {
      $.getJSON('/search', { q: term })
        .done(function (resp) { callback(resp); })
        .fail(function ()     { callback([]);   });
    },
    replace: function (value) {
      return '$1@' + value + ' ';
    }
  },

  // emoji strategy
  emoji: {
    match: /(^|\s):(\w*)$/,
    search: function (term, callback) {
      var regexp = new RegExp('^' + term);
      callback($.map(emojies, function (emoji) {
        return regexp.test(emoji) ? emoji : nil;
      }));
    },
    replace: function (value) {
      return '$1:' + value + ': ';
    }
  }
});
```

License
-------

MIT with starring the repository at GitHub. OR GPL without starring.
