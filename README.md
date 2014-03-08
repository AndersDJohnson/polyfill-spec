# Polyfill Spec

**An open standard formalizing patterns for polyfill identification and interoperability.**

Polyfills are great. But they do not always conform perfectly to the behavior of the features they emulate. This can mislead downstream code, whose feature detection might result in an assumption of full conformance. We need a standard so that polyfills can be identified as such.

This allows downstream code to check for a possibly polyfilled feature, whose implementation and level of conformance is unknown to it, and then in those cases, choose whether to opt instead to use a preferred implementation, or use fallback behavior.

## Use Case

In a recent project, we used [Respond.js](https://github.com/scottjehl/Respond) to support media queries for older browsers. It relies on the [matchMedia.js](https://github.com/paulirish/matchMedia.js) polyfill to determine whether media queries are supported in the browser. It always returned `false` in older browsers, which indicated to Respond that it should polyfill. But other code in the project wanted to use the more conformant  [media-match](https://github.com/weblinc/media-match) polyfill to keep responsive JavaScript behavior in sync with responsive CSS styles. It would have fooled Respond into thinking media queries were supported, so that Respond would not polyfill. So we modified matchMedia.js to set the `polyfill` property, and media-match to detect this, then override with its polyfill.


## Proposal

Polyfill implementations should expose a `polyfill` property with a value of `true`.

##  Pattern

For polyfill library authors, here is a code wrapper that can help your library conform to this spec.

```js
(function (root, factory) {
  var spec = factory(root);
  var feature = spec.get ? spec.get() : root[spec.global];
  if ( ! feature || ( feature.polyfill && spec.replace ) ) {
    var polyfill = spec.polyfill;
    polyfill.polyfill = true;
    spec.set ? spec.set(polyfill) : root[spec.global] = polyfill;
  }
})(this, function (root) {

  // return spec definition...

});
```

### spec definition examples

a. Global name
  ```js
  {
    global: 'matchMedia',
    polyfill: function () {
      // implementation...
    }
  };
  ```

b. Getter/setter
  ```js
  {
    get: function () {
      return root.navigator.geolocation;
    },
    set: function (poylfill) {
      root.navigator.geolocation = polyfill;
    },
    polyfill: function () {
      // implementation...
    }
  };
  ```

c. Replace an existing polyfill
  ```js
  {
    global: 'matchMedia',
    replace: true,
    polyfill: function () {
      // implementation...
    }
  };
  ```
