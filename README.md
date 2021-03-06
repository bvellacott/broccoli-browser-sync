# broccoli-browser-sync - DEPRECATED - go to [Globegitter/broccoli-browser-sync](https://github.com/Globegitter/broccoli-browser-sync)
BrowserSync support for Broccolijs

## Usage

Right now usage is still a bit suboptimal, but you already get all the beauty of BrowserSync including live css injection as well as livereload.

```js
var browserSync = new BrowserSync(inputTrees, options);
```

to pass in any browsersync options:
```js
var options = {
  browserSync: {
    browser: 'Firefox'
  }
}
```

A fairly complex example:
```js
var fastBrowserify = require('broccoli-fast-browserify');
var babelify = require('babelify');
var mergeTrees = require('broccoli-merge-trees');
var compileSass = require('broccoli-sass-source-maps');
var funnel = require('broccoli-funnel');

var BrowserSync = require('broccoli-browser-sync');
var proxy = require('http-proxy-middleware');

var optionalTransforms = [
  'regenerator'
  // 'minification.deadCodeElimination',
  // 'minification.inlineExpressions'
];

var babelOptions = {stage: 0, optional: optionalTransforms, compact: true};

// var browserifyOpts = {deps: true, entries: files, noParse: noParse, ignoreMissing: true};
var transformedBabelify = fastBrowserify('app', {
  browserify: {
    extensions: [".js"]
  },
  bundles: {
    'js/app.js': {
      entryPoints: ['app.js'],
      transform: {
        tr: babelify,
        options: {
          stage: 0
        }
      }
    }
  }
});

var appCss = compileSass(['piggy/frontend/app'], 'main.scss', 'css/app.css');

var staticFiles = funnel('frontend', {
  srcDir: 'static'
});

// browsersync options
var bsOptions = {
  browserSync: {
    open: false,
    middleware: [
      proxy('/api/**', {
        target: 'http://localhost:8080/',
        pathRewrite: {
          '^/api': ''
        }
      }),
      proxy('/live', {
        target: 'http://localhost:8080/',
        pathRewrite: {
          '^/live': ''
        },
        ws: true})
    ]
  }
};
var browserSync = new BrowserSync([staticFiles, transformedBabelify, appCss], bsOptions);

module.exports = mergeTrees([staticFiles, transformedBabelify, appCss, browserSync]);
```
