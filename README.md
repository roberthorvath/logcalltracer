# logcalltracer
A minimalistic library that shows the caller's location in log messages, with an optional timestamp, log level.

## Usage

You can use the following methods, the same way you would use console.log, .warn, and .error methods:
* logInfo
* logWarn
* logError

The default output is the caller's location, then the values of the parameters. Example:
```javascript
// Import module
const logger = require("logcalltracer");

logger.logInfo("Hello world!")
```

shows the following output:
```
/myApp/start.js:4: Hello world!
```

## Customizing output
The library can be configured to provide additional information, such as timestamp, log level, coloring, and so on. Use the following properties on the `options` object:
### **timeStampFormatter** : Function
A method to get a formatted timestamp. The default value is `undefined` so log output has no timestamp. You can use the built-in Date object, or any 3rd party library, to provide a method that returns your preferred timestamp.

Example:
```javascript
// Import module
const logger = require("logcalltracer");

logger.options.timeStampFormatter = () => new Date().toISOString();

logger.logInfo("Hello world!")
```

shows the following output:
```
2021-06-04T11:57:43.175Z  /myApp/start.js:4: Hello world!
```

### **showLogLevel**: boolean
When et to true, the output displays the log level.

### **inspectOptions**: object
This library uses `util.inspect` to format arguments. You can change the default formatting behavior using the inspectOptions: https://nodejs.org/dist/latest-v16.x/docs/api/util.html#util_util_inspect_object_options

For a development environment, I suggest the following to start with:
```javascript
const logger = require("logcalltracer");
logger.options.inspectOptions = {compact: false, depth: Infinity};
```
### **logInfoFn**: Function, and **logWarnFn**, **logErrorFn**
By default, the methods are mapped to the `console`'s logging methods:
| Method | Calls |
|---     |---    |
|logInfo | console.log(timestamp?, info?, callerLocation, ...arguments) |
|logWarn | console.warn(timestamp?, warn?, callerLocation, ...arguments) |
|logError| console.error(timestamp?, err?, callerLocation, ...arguments) |

You can plug in your own, or an external logging library's logger methods. Example:

```javascript
const logger = require("logcalltracer")
const util = require("util")
const fs = require("fs");

logger.options.logInfoFn = function(timestamp, logLevel, callerLocation, ...rest) {
    console.log("My custom logging method");
    fs.appendFile(
        "test.log",
        `${timestamp} ${logLevel} ${callerLocation} ${util.formatWithOptions.apply(null, [logger.options.inspectOptions, ...rest])}\n`,
        "utf8",
        err => {});
}

logger.options.timeStampFormatter = () => new Date().toISOString();
// logger.options.showLogLevel is not set, so it won't be passed to the logInfoFn method.
logger.logInfo("Application starting..", { a: 42, b: { c: 'Hi!'}});
```
will create a file with the following content:
```
2021-06-04T12:26:09.455Z  /myApp/start.js:16: Application starting.. { a: 42, b: { c: 'Hi!' } }
```
### **logLevelColored**: get/set(boolean)
By default, no coloring is performed on the output. When both `showLogLevel` and `logLevelColored` are set to true, the log level is colored as following:
| Method | Color |
|---     |---    |
|logInfo | None  |
|logWarn | Yellow|
|logError| Red   |

### An example with options
```javascript
// Import module
const logger = require("logcalltracer");

// Customize the behavior if you want to.
if (process.env.NODE_ENV === "production") {
  logger.options.timeStampFormatter = () => new Date().toISOString();
  logger.options.showLogLevel = true;
  logger.options.inspectOptions = {compact: false};
} else {
  logger.options.timeStampFormatter = () => new Date().toLocaleTimeString();
  logger.options.inspectOptions = {compact: false, depth: Infinity};
  logger.options.logLevelColored = true;
  logger.options.showLogLevel = true;
}

logger.logInfo("Application starting..", { a: 42, b: { c: 'Hi!'}});
```

shows the following output:
```
1:42:13 PM INFO  c:\myApp\start.js:16: Application starting.. {
  a: 42,
  b: {
    c: 'Hi!'
  }
}
```

## License

[MIT](LICENSE)