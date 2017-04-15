# api documentation for  [gulp-webserver (v0.9.1)](https://github.com/schickling/gulp-webserver)  [![npm package](https://img.shields.io/npm/v/npmdoc-gulp-webserver.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-gulp-webserver) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-gulp-webserver.svg)](https://travis-ci.org/npmdoc/node-npmdoc-gulp-webserver)
#### Gulp plugin to run a local webserver with LiveReload

[![NPM](https://nodei.co/npm/gulp-webserver.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/gulp-webserver)

[![apidoc](https://npmdoc.github.io/node-npmdoc-gulp-webserver/build/screenCapture.buildCi.browser.apidoc.html.png)](https://npmdoc.github.io/node-npmdoc-gulp-webserver/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-gulp-webserver/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-gulp-webserver/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Johannes Schickling",
        "url": "https://github.com/schickling"
    },
    "bugs": {
        "url": "https://github.com/schickling/gulp-webserver/issues"
    },
    "dependencies": {
        "connect": "^3.0.1",
        "connect-livereload": "^0.4.0",
        "gulp-util": "^2.2.19",
        "isarray": "0.0.1",
        "node.extend": "^1.0.10",
        "open": "^0.0.5",
        "proxy-middleware": "^0.5.0",
        "serve-index": "^1.1.4",
        "serve-static": "^1.3.0",
        "through2": "^0.5.1",
        "tiny-lr": "0.1.4",
        "watch": "^0.11.0"
    },
    "description": "Gulp plugin to run a local webserver with LiveReload",
    "devDependencies": {
        "mocha": "^1.20.1",
        "supertest": "^0.13.0"
    },
    "directories": {},
    "dist": {
        "shasum": "e09992165d97c5865616d642a1601529b0367064",
        "tarball": "https://registry.npmjs.org/gulp-webserver/-/gulp-webserver-0.9.1.tgz"
    },
    "gitHead": "ff363747cb06f6bcda9d3cec98698615c2cfbea1",
    "homepage": "https://github.com/schickling/gulp-webserver",
    "keywords": [
        "gulpplugin",
        "webserver",
        "connect",
        "livereload"
    ],
    "license": "MIT",
    "main": "src/index.js",
    "maintainers": [
        {
            "name": "schickling"
        }
    ],
    "name": "gulp-webserver",
    "optionalDependencies": {},
    "repository": {
        "type": "git",
        "url": "git+https://github.com/schickling/gulp-webserver.git"
    },
    "scripts": {
        "test": "mocha"
    },
    "version": "0.9.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module gulp-webserver](#apidoc.module.gulp-webserver)
1.  [function <span class="apidocSignatureSpan"></span>gulp-webserver (options)](#apidoc.element.gulp-webserver.gulp-webserver)
1.  [function <span class="apidocSignatureSpan">gulp-webserver.</span>toString ()](#apidoc.element.gulp-webserver.toString)



# <a name="apidoc.module.gulp-webserver"></a>[module gulp-webserver](#apidoc.module.gulp-webserver)

#### <a name="apidoc.element.gulp-webserver.gulp-webserver"></a>[function <span class="apidocSignatureSpan"></span>gulp-webserver (options)](#apidoc.element.gulp-webserver.gulp-webserver)
- description and source-code
```javascript
gulp-webserver = function (options) {

  var defaults = {

<span class="apidocCodeCommentSpan">    /**
     *
     * BASIC DEFAULTS
     *
     **/
</span>
    host: 'localhost',
    port: 8000,
    path: '/',
    fallback: false,
    https: false,
    open: false,

    /**
     *
     * MIDDLEWARE DEFAULTS
     *
     * NOTE:
     *  All middleware should defaults should have the 'enable'
     *  property if you want to support shorthand syntax like:
     *
     *    webserver({
     *      livereload: true
     *    });
     *
     */

    // Middleware: Livereload
    livereload: {
      enable: false,
      port: 35729,
      filter: function (filename) {
        if (filename.match(/node_modules/)) {
          return false;
        } else { return true; }
      }
    },

    // Middleware: Directory listing
    // For possible options, see:
    //  https://github.com/expressjs/serve-index
    directoryListing: {
      enable: false,
      path: './',
      options: undefined
    },

    // Middleware: Proxy
    // For possible options, see:
    //  https://github.com/andrewrk/connect-proxy
    proxies: []

  };

  // Deep extend user provided options over the all of the defaults
  // Allow shorthand syntax, using the enable property as a flag
  var config = enableMiddlewareShorthand(defaults, options, [
    'directoryListing',
    'livereload'
  ]);

  if (typeof config.open === 'string' && config.open.length > 0 && config.open.indexOf('http') !== 0) {
    // ensure leading slash if this is NOT a complete url form
    config.open = (config.open.indexOf('/') !== 0 ? '/' : '') + config.open;
  }

  var app = connect();

  var openInBrowser = function() {
    if (config.open === false) return;
    if (typeof config.open === 'string' && config.open.indexOf('http') === 0) {
      // if this is a complete url form
      open(config.open);
      return;
    }
    open('http' + (config.https ? 's' : '') + '://' + config.host + ':' + config.port + (typeof config.open === 'string' ? config
.open : ''));
  };

  var lrServer;

  if (config.livereload.enable) {

    app.use(connectLivereload({
      port: config.livereload.port
    }));

    if (config.https) {
      if (config.https.pfx) {
        lrServer = tinyLr({
          pfx: fs.readFileSync(config.https.pfx),
          passphrase: config.https.passphrase
        });
      }
      else {
        lrServer = tinyLr({
          key: fs.readFileSync(config.https.key || __dirname + '/../ssl/dev-key.pem'),
          cert: fs.readFileSync(config.https.cert || __dirname + '/../ssl/dev-cert.pem')
        });
      }
    } else {
      lrServer = tinyLr();
    }

    lrServer.listen(config.livereload.port, config.host);

  }

  // middlewares
  if (typeof config.middleware === 'function') {
    app.use(config.middleware);
  } else if (isarray(config.middleware)) {
    config.middleware
      .filter(function(m) { return typeof m === 'function'; })
      .forEach(function(m) {
        app.use(m);
      });
  }

  // Proxy requests
  for (var i = 0, len = config.proxies.length; i < len; i++) {
    var proxyoptions = url.parse(config.proxies[i].target);
    if (config.proxies[i].hasOwnProperty('options')) {
      extend(proxyoptions, config.proxies[i].options);
    }
    app.use(config.proxies[i].source, proxy(proxyoptions));
  }

  if (config.directoryListing.enable) {
    app.use(config.path, serveIndex(path.resolve(config.directoryListing.path), config.directoryListing.options));
  }


  var files = [];

  // Create server
  var stream = through.obj(function(file, enc, callback) {

    app.use(config.path, serveStatic(file.path));

    if (config.livereload.enable) {
      var watchOptions = {
        ignoreDotFiles: true,
        filter: config.livereload.filter
      };
      watch.watchTree(file.path, watchOptions, function (filename) {
        lrServer.changed({
          body: {
            files: filename
          }
        });

      });
    }

    this.push(file);
    callback();
  })
  .on('data', function(f){files.push(f);})
  .on('end', function(){
    if (config.fallback) {
      files.forEach(function(file){
        var fallba ...
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.gulp-webserver.toString"></a>[function <span class="apidocSignatureSpan">gulp-webserver.</span>toString ()](#apidoc.element.gulp-webserver.toString)
- description and source-code
```javascript
toString = function () {
    return toString;
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
