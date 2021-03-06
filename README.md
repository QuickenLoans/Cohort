# Cohort

Compiling files is tedious. Humans are bad at doing tedious things because we get bored and make mistakes. Computers are good at doing tedious things because they don't get bored and do everything they are told to do. Building source files into a distributable set of files is something that is tedious and needs to be done consistently between developers of a project so why not let the computer do that work for you?

Cohort is an attempt to alleviate most of this work for the developer. Simply define the procedures to perform to achieve a complete build of a project. The focus of Cohort is the simplicity of the setup; no programming should be necessary by the developer, beyond what they would have to do without a tool such as Cohort. The goal is for Cohort to be a drop-in helper to most projects; if you are using a Cakefile for compiling your Coffeescript you wont need to duplicate any of that in Cohort because the Cakefile will just be executed by Cohort instead of you.

```javascript
/*
  Function:
    cohort

  Description:
    Invoking `cohort` with arguments, builds a function and returns that function for later execution.

  Arguments:
    @config   - all build information to be executed when creating a new build
    @init     - only necessary to execute when starting development on the project new
    @callback - function executed immediately before execution is complete
*/

// function returned, nothing is executed
cohort(config, init, callback);

// function returned and immediately executed
cohort(config, init, callback)();
```

## Features

* (optional) Perform initial project setup
  * git submodules download and build
  * npm dependencies
* Execute commands before the build starts
* Lint files
* Concatenate and Minify files
* Run tests on compiled files
* Cleanup any temp files produced by the build

## Examples

Here is a basic example of Cohort doing lint*/concat/min on some css and js. 

*Linting is only done on js files currently.

```javascript
var cohort = require("cohort");

cohort.extend("less");
cohort.extend("scss");
cohort.extend("styl");

cohort([
  [ // files
    ["dist/css/app.css", [
        "src/less/libs.less"
      , "src/sass/example.scss"
      , "src/css/sample.css"
      , "src/css/another.css"
      ]]

    , ["dist/css/sass_libs.css", [
        "src/sass/example.scss"
      ]]

    , ["dist/css/stylus_libs.css", [
        "src/stylus/lib.styl"
      ]]

    , ["dist/js/app.js", [
        "src/js/app.js"
      , "dist/js/coffee_libs.js" // from Cake built file
      , "src/js/lib/important.coffee" // inline compile of coffee file
      ]]
  ]
])(); // notice the trailing parens to invoke the build,
      // cohort returns a function by default to allow for chaining of proceedures
```

The output of running this Cohort file will be the two files within the `dist` directory (`css/app.css` and `js/app.js`).

Notice that Cohort doesn't care about the type of files that will be concat'ed together (e.g. the css file accepts: css, less, scss[, more coming]). *Cohort does not support sass syntax as `libsass` doesn't support it, since it is deprecated.* Cohort will also concat coffee files inline although this is an anti-pattern since coffee files should be compiled together for optimizations that the coffescript compiler will do; Cohort suggests using a `Cakefile` to compile before concatenating js files together.

To execute commands before and/or after the files have been compiled Cohort offers a few options.

```javascript
cohort([
  banner // [string], will be prepended to all compiled files (optional)
  , [ // pre-build
      "cake -d dist/js -f coffee_libs bake"
    ]

  , [/*...*/] // files

  , [ // testing
      "mocha"
    ]

  , [ // cleanup
      "rm -rf dist/js/coffee_libs.js"
    ]
  ]
  , [ // init
      "git submodule update --init --recursive"
    , "cd git_modules/jquery && grunt build" // example jquery build
    ])();
```

Strings within the arrays are just commands that will be executed in the shell so will have access to installed libraries.

## Use

The expected use case for Cohort is to help with development by easing the setup for new developers, and maintain consistency across all developers. Starting from scratch the steps should be:

1. Clone the repository
2. Install npm dependencies - this is an unfortunate 'catch 22' fix since Cohort is an npm package that can't be used till it is downloaded.
3. Run `./cohort.js init` (#winning)
4. Start developing

## Advanced Configuration

By making use of the callback argument some tasks can be parallelized for speed; CSS and JS are not at all interdependent and should not ever be blocking for each other.

```javascript
cohort([["echo 'Build started'"]], function () {
  cohort([[css_files_config]])();
  cohort([[javascript_files_config]])();
  })();
```

# Libraries Being Used

* Chai - test assertion library
* CoffeeScript (via Cake)
* JSHint - static analysis and cyclomatic complexity
* LibSass - CSS preprocessor
* Less - CSS preprocessor
* Mocha - unit testing
* Stylus - CSS preprocessor
* Uglify - compression