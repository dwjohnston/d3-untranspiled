This repo reproduces the problem with providing untranspiled code as a repo. 

## The Work around

The work around is to add this to your script test: 

```
    "test": "react-scripts test --transformIgnorePatterns 'node_modules/!(d3-selection|another-dependency-here)'",
```

Here, essentially we are saying 'don't attempt transpilation on any node_modules, _except_ these ones (ie. still transpile those).

## Info

In this case the offender is `d3-selection@3.0.0`

To see the error: 

```
yarn
yarn test
```

To see the workaround, see the `workaround` branch. 


```
 FAIL  src/App.test.js
  ● Test suite failed to run

    Jest encountered an unexpected token

    Jest failed to parse a file. This happens e.g. when your code or its dependencies use non-standard JavaScript syntax, or when Jest is not configured to support such syntax.

    Out of the box Jest supports Babel, which will be used to transform your files into valid JS based on your Babel configuration.

    By default "node_modules" folder is ignored by transformers.

    Here's what you can do:
     • If you are trying to use ECMAScript Modules, see https://jestjs.io/docs/ecmascript-modules for how to enable it.
     • If you are trying to use TypeScript, see https://jestjs.io/docs/getting-started#using-typescript
     • To have some of your "node_modules" files transformed, you can specify a custom "transformIgnorePatterns" in your config.
     • If you need a custom transformation specify a "transform" option in your config.
     • If you simply want to mock your non-JS modules (e.g. binary assets) you can stub them out with the "moduleNameMapper" config option.

    You'll find more details and examples of these config options in the docs:
    https://jestjs.io/docs/configuration
    For information about custom transformations, see:
    https://jestjs.io/docs/code-transformation

    Details:

    /home/djohnston/git/repros/d3-selection-untranspiled/node_modules/d3-selection/src/index.js:1
    ({"Object.<anonymous>":function(module,exports,require,__dirname,__filename,jest){export {default as create} from "./create.js";
                                                                                      ^^^^^^

    SyntaxError: Unexpected token 'export'

      2 | import './App.css';
      3 |
    > 4 | import * as d3 from 'd3-selection';
        | ^
      5 |
      6 | function App() {
      7 |   return (

      at Runtime.createScriptFromCode (node_modules/jest-runtime/build/index.js:1728:14)
      at Object.<anonymous> (src/App.js:4:1)

Test Suites: 1 failed, 1 total
Tests:       0 total
Snapshots:   0 total
Time:        0.836 s
Ran all test suites related to changed files.
``` 


## More details: 


The d3-selection package looks like this: 


```
{
  "name": "d3-selection",
..snip
  "type": "module",
  "module": "src/index.js",
  "main": "src/index.js",
```


I believe the problem here is that `main` also points to `src/index` - meaning the untranspiled code will be used, and then jest doesn't like it. 

I thought that perhaps deleting the `main` property might force the module resolution into using the module, but apparently not. 
 
I'm not sure where it is configured, if anyone wants to shine some light, but it seems like jest is doing a plain cjs module resolution strategy for the node_modules. 


