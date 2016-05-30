# dynamic-config

Read config files depending on your environment. 

- load different configurations depending on your _env_ 
  - via argv: `node app.js --env production`
  - via env: `export ENV=production; node app.js` 
- expects a JavaScript file as config (so you can add dynamic content) 

## Options 

## defaultEnv: String, (default: develop)

Define which env should be set as default. 

## log: Boolean (default: false) 

Enable logging of path/env resolution. 

## envName: String (default: env) 

The argument/env variable name we expect. 

## Example

```javascript
//config/index.js

var dynamicConfig = require("dynamic-config");

//optional options
dynamicConfig.options.defaultEnv = "develop";
dynamicConfig.options.log = true;

var config = dynamicConfig(__dirname, "config.js");

//log the resolved config path (e.g. /etc/myapp/develop/config.js)
console.log(dynamicConfig.resolvedConfigPath);

module.exports = config;
```

```javascript 
//config/develop/config.js

module.exports = {
  whereami: "develop"
}
```

```javascript
//app.js
var config = require("./config");

console.log(config);
```

```javascript 
node app.js

{ whereami: 'develop' }

//# Set environment via args
node app.js --env prod

returns { whereami: 'prod' }


//# Set environment via env
export env=stage; node app.js

{ whereami: 'stage' }
```





## Plugins

### extend()

```javascript

"use strict";

var dynamicConfig = require("dynamic-config");

//# extend from env
dynamicConfig.use(require("dynamic-config/plugins/extend/env"));

//# extend from argv
dynamicConfig.use(require("dynamic-config/plugins/extend/argv"));

//# extend from file
dynamicConfig.use(require("dynamic-config/plugins/extend/file"));

module.exports = dynamicConfig(__dirname, "config.js");
```

```javascript
node app.js

{ name: 'superApp', port: 9000 }

//# Overwrite via argv
node app.js --port 80 //or node app.js --port=80

{ name: 'superApp', port: 80 }

//# Overwrite via env
export port=90 node app.js

{ name: 'superApp', port: 90 }

//# Order matters...
export port=90; node app.js --port 80

{ name: 'superApp', port: 80 }
```

#### Overwrite via file
Create a file named the same as your config, but contains `.local` in front of the extension.
Example: `config.js` -> `config.local.js`

In the overwrite file you can define any subset of the object, that is defined in the main config and it would overwrite
the corresponding value.

```javascript
//# config.js
module.exports = {
    a: 1,
    b: {
        c: "c",
        d: 2
    },
    e: 3
}

//# config.local.js
module.exports = {
    e: "e",
    b: {
        d: "d"
    }
}

//# result
{
    a: 1,
    b: {
        c: "c",
        d: "d"
    },
    e: "e"
}
```