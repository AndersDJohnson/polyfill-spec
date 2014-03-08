# Polyfill Spec

**An open standard formalizing patterns for polyfill identification and interoperability.**

Polyfills are great. But they do not always conform perfectly to the behavior of the features they emulate. This can be mislead downstream code, whose feature detection might result in the assumption of full conformance. We need a standard so polyfills can be identified as such.

This allows downstream code to check for a possibly polyfilled feature, whose implementation and level of conformance is unknown to it, and in those cases opt to use a preferred implementation.

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
