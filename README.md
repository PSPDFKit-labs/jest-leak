## Jest Leak

This repo is used to reproduce a memory leak in Jest that was introduced between [v22.2.2 and v22.3.0](https://github.com/facebook/jest/compare/v22.2.2...v22.3.0).

The test case inside `__tests__` will require a very large JavaScript file which will only work when babel transpiling is turned off for this file (in this case, the JavaScript file is an auto-generated asm.js file from the latest [PSPDFKit](https://pspdfkit.com/web) version). 

You can see that the `transformIgnorePatterns` rule proplery works by running:

```
yarn jest
```

The leak occurs when using the `--coverage` option. It seems like the `transformIgnorePatterns` property is ignored in this case.

```
yarn jest --coverage
```

The resulting output will look like this:

```
<--- Last few GCs --->

[20556:0x102801e00]    17087 ms: Mark-sweep 1407.2 (1467.9) -> 1407.1 (1471.4) MB, 954.8 / 0.0 ms  allocation failure GC in old space requested
[20556:0x102801e00]    18088 ms: Mark-sweep 1407.1 (1471.4) -> 1407.0 (1440.4) MB, 1000.9 / 0.0 ms  last resort GC in old space requested
[20556:0x102801e00]    19084 ms: Mark-sweep 1407.0 (1440.4) -> 1407.0 (1440.4) MB, 996.6 / 0.0 ms  last resort GC in old space requested


<--- JS stacktrace --->

==== JS stack trace =========================================

Security context: 0x2de3696a5501 <JSObject>
    1: /* anonymous */ [/Users/philipp/dev/playground/jest-leak/node_modules/babylon/lib/index.js:~3555] [pc=0x1e01706df560](this=0x2de3f97b8849 <Parser map = 0x2de300c69719>,close=0x2de354570589 <TokenType map = 0x2de300c29f51>,possibleAsyncArrow=0x2de3212023e1 <false>)
    2: /* anonymous */ [/Users/philipp/dev/playground/jest-leak/node_modules/babylon/lib/index.js:~3507] [pc=0x1e01706cdd08](t...

FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
 1: node::Abort() [/Users/philipp/.asdf/installs/nodejs/9.7.1/bin/node]
 2: node::FatalTryCatch::~FatalTryCatch() [/Users/philipp/.asdf/installs/nodejs/9.7.1/bin/node]
 3: v8::internal::V8::FatalProcessOutOfMemory(char const*, bool) [/Users/philipp/.asdf/installs/nodejs/9.7.1/bin/node]
 4: v8::internal::Factory::NewUninitializedFixedArray(int) [/Users/philipp/.asdf/installs/nodejs/9.7.1/bin/node]
 5: v8::internal::(anonymous namespace)::ElementsAccessorBase<v8::internal::(anonymous namespace)::FastPackedObjectElementsAccessor, v8::internal::(anonymous namespace)::ElementsKindTraits<(v8::internal::ElementsKind)2> >::GrowCapacity(v8::internal::Handle<v8::internal::JSObject>, unsigned int) [/Users/philipp/.asdf/installs/nodejs/9.7.1/bin/node]
 6: v8::internal::Runtime_GrowArrayElements(int, v8::internal::Object**, v8::internal::Isolate*) [/Users/philipp/.asdf/installs/nodejs/9.7.1/bin/node]
 7: 0x1e01705842fd
error Command failed with signal "SIGABRT".
```

You can see that the stack trace is referencing `babylon` which should not be running for the large JavaScript file.

