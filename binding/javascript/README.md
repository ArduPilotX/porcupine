# JavaScript Binding for Porcupine

## Compatibility

The binding uses [WebAssembly](https://webassembly.org/) which is supported on
[almost all](https://caniuse.com/#feat=wasm) modern browsers.

## Initialization

Typically, the Porcupine WASM module is loaded asynchronously, and is therefore not guaranteed to be ready the first time you wish use `Porcupine.create()`. There are two options for handling this:

1. Poll the Porcupine `isLoaded()` method until it is true
2. Supply a callback to `Porcupine` using the `PorcupineOptions` method with your function assigned to the `callback` key; it will be invoked when `isLoaded()` becomes true

```javascript
let callback = function callback() {
  console.log("Porcupine.create() is ready to be called.");
};
// n.b.: PorcupineOptions must be declared before the Porcupine binding is loaded, or it will be ignored
let PorcupineOptions = { callback: callback };
```

## Usage

Create a new instance of the engine using `Porcupine.create()`:

```javascript
let keywordModels = [new Uint8Array([...]), ...];
let sensitivities = new Float32Array([0.5, ...]);

let handle = Porcupine.create(keywordModels, sensitivities)
```

`keywordModels` is an array of keyword signatures to be detected. Each model is stored as an array of 8-bit unsigned
integers (i.e. `UInt8Array`). Several models are available for free under [resource folder](/resources/keyword_files/wasm).

`sensitivities` is an array of 32-bit floating-point numbers (i.e. `Float32Array`). A higher sensitivity results in a
lower miss rate at the cost of a potentially higher false alarm rate. Sensitivity values should be within `[0, 1]`.

When instantiated `handle` can process audio via its `.process` method.

```javascript
    let getNextAudioFrame = function() {
        ...
    };

    while (true) {
        let keywordIndex = handle.process(getNextAudioFrame());
        if (keywordIndex !== -1) {
            // detection event callback
        }
    }
```

`getNextAudioFrame()` should return an array of audio samples in 16-bit format (i.e. `Int16Array`). The length of the
array can be retrieved using `handle.frameLength` and the required sample rate using `handle.sampleRate`. When done be
sure to release resources acquired by WebAssembly using `.release`.

```javascript
handle.release();
```
