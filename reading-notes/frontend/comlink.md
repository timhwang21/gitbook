# Comlink

## Usage Summary

* Exposes two main APIs, `wrap()` and `expose()`.
  * `expose()` takes a value defined in a worker. This value can be a static value, or it can be a function.
  * `wrap()` takes a `Worker` which is initialized with the path to a worker module. It returns a proxy of the `expose()`d value. This proxy is asynchronous -- if the wrapped value is a function, it becomes an async function, and if the wrapped value is an object, getters on the object are now async.
* The proxy returned by `wrap()` can receive values in three ways:
  * For primitives, no special handling is needed. However, it's worth noting that these values are deep-copied.
  * Certain values such as `ArrayBuffer`s can be `transfer()`ed instead of deep copied. This would be useful for cryptography and image/video processing, which 1. are resource-intensive, and 2. use buffers as the standard data transfer primitive already.
  * Nonserializable values can be `proxy()`ed, which exposes a proxy of the value. This is the required approach for functions \(for callbacks\), and other things like DOM events \(though this seems messy and like something you should avoid\).
* Warning: proxied worker functions cannot be directly used as event handlers, because events are not clonable or transferrable. You must construct a [transfer handler](https://github.com/GoogleChromeLabs/comlink#transfer-handlers-and-event-listeners) that marshals and unmarshals the event in a sane way.
* To garbage collect, `releaseProxy()` detaches the proxy.

## Helpers

* [Worker Plugin](https://github.com/GoogleChromeLabs/worker-plugin) is a Webpack plugin that automagically detects modules passed to `new Worker()` and processes them.
* [Workerize Loader](https://github.com/developit/workerize-loader) is a Webpack loader that automatically \(with less magic\) processes modules by name or via the `'workerize-loader!'` import helper.
* [Clooney](https://github.com/GoogleChromeLabs/clooney) is a helper library that abstracts away workers entirely. Exposes a function that wraps a class \(no functions allowed, unfortunately\) and makes all its functions async. Under the hood, the class is actually running in a worker. This paired with Worker Plugin seems like the most "batteries included" approach.
* A `comlink-loader` package exists, but is [unofficially deprecated](https://github.com/GoogleChromeLabs/comlink/issues/253). Worker Plugin is the official Google Chrome Labs recommendation.
* [Workly](https://github.com/pshihn/workly) looks like a dead simple library that's admittedly less flexible, but super easy to drop in. No bundler interaction at all needed.
* [Greenlet](https://github.com/developit/greenlet) is an _even simpler_ version of Workly that is _even less flexible_, and only accepts functions. If you're only ever workerizing functions, this seems like the way to go!

