# gulp-replace-task [![Build Status](https://secure.travis-ci.org/outaTiME/gulp-replace-task.png?branch=master)](http://travis-ci.org/outaTiME/gulp-replace-task)

> Replace text patterns using [pattern-replace](https://github.com/outaTiME/pattern-replace).



## Install

From NPM:

```shell
npm install gulp-replace-task --save-dev
```

## Replace Task

Assuming installation via NPM, you can use `gulp-replace-task` in your gulpfile like this:

```javascript
var gulp = require('gulp');
var replace = require('gulp-replace-task');

gulp.task('default', function() {
  gulp.src('src/index.html')
    .pipe(replace({
      patterns: [
        {
          match: 'foo',
          replacement: 'bar'
        }
      ]
    }))
    .pipe(gulp.dest('build'));
});
```

### Options



#### patterns
Type: `Array`

Define patterns that will be used to replace the contents of source files.

#### patterns.match
Type: `String|RegExp`

Indicates the matching expression.

If matching type is `String` and `expression` attribute is `false` we use a simple variable lookup mechanism `@@string` (in any other case we use the default regexp replace logic):

```javascript
{
  patterns: [
    {
      match: 'foo',
      replacement: 'bar', // replaces "@@foo" to "bar"
      expression: false   // simple variable lookup
    }
  ]
}
```

#### patterns.replacement
Type: `String|Function|Object`

Indicates the replacement for match, for more information about replacement check out the [String.replace].

You can specify a function as replacement. In this case, the function will be invoked after the match has been performed. The function's result (return value) will be used as the replacement string.

```javascript
{
  patterns: [
    {
      match: /foo/g,
      replacement: function () {
        return 'bar'; // replaces "foo" to "bar"
      }
    }
  ]
}
```

Also supports object as replacement (we create string representation of object using [JSON.stringify]):

```javascript
{
  patterns: [
    {
      match: /foo/g,
      replacement: [1, 2, 3] // replaces "foo" with string representation of "array" object
    }
  ]
}
```

[String.replace]: http://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace
[JSON.stringify]: http://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

#### patterns.json
Type: `Object`

If an attribute `json` found in pattern definition we flatten the object using `delimiter` concatenation and each key–value pair will be used for the replacement (simple variable lookup mechanism and no regexp support).

```javascript
{
  patterns: [
    {
      json: {
        "key": "value" // replaces "@@key" to "value"
      }
    }
  ]
}
```

Also supports nested objects:

```javascript
{
  patterns: [
    {
      json: {
        "key": "value",   // replaces "@@key" to "value"
        "inner": {        // replaces "@@inner" with string representation of "inner" object
          "key": "value"  // replaces "@@inner.key" to "value"
        }
      }
    }
  ]
}
```

#### patterns.yaml
Type: `String`

If an attribute `yaml` found in pattern definition we flatten the object using `delimiter` concatenation and each key–value pair will be used for the replacement (simple variable lookup mechanism and no regexp support).

```javascript
{
  patterns: [
    {
      yaml: 'key: value'  // replaces "@@key" to "value"
    }
  ]
}
```

#### patterns.expression
Type: `Boolean`
Default: `false`

Indicates the type of matching.

If detects regexp instance in `match` attribute, we assume to works with expression matcher (in any other case should be forced).

#### variables
Type: `Object`

This is the old way to define patterns using plain object (simple variable lookup mechanism and no regexp support), you can still using but for more control you should use the new `patterns` way.

```javascript
{
  variables: {
    'key': 'value' // replaces "@@key" to "value"
  }
}
```

#### prefix
Type: `String`
Default: `@@`

The prefix added for matching (prevent bad replacements / easy way).

> This only applies for simple variable lookup mechanism.

#### usePrefix
Type: `Boolean`
Default: `true`

If set to `false`, we match the pattern without `prefix` concatenation (useful when you want to lookup an simple string).

> This only applies for simple variable lookup mechanism.

#### preservePrefix
Type: `Boolean`
Default: `false`

If set to `true`, we preserve the `prefix` in target.

> This only applies for simple variable lookup mechanism and `patterns.replacement` is an string.

#### delimiter
Type: `String`
Default: `.`

The delimiter used to flatten when using object as replacement.

#### preserveOrder
Type: `Boolean`
Default: `false`

If set to `true`, we preserve the patterns definition order, otherwise these will be sorted (in ascending order) to prevent replacement issues like `head` / `header` (typo regexps will be resolved at last).


### Usage Examples

#### Basic

File `src/manifest.appcache`:

```
CACHE MANIFEST
# @@timestamp

CACHE:

favicon.ico
index.html

NETWORK:
*
```

gulpfile:

```js
gulp.task('default', function() {
  gulp.src('src/manifest.appcache')
    .pipe(replace({
      patterns: [
        {
          match: 'timestamp',
          replacement: new Date().getTime()
        }
      ]
    }))
    .pipe(gulp.dest('build'));
});
```

#### Multiple matching

File `src/manifest.appcache`:

```
CACHE MANIFEST
# @@timestamp

CACHE:

favicon.ico
index.html

NETWORK:
*
```

File `src/humans.txt`:

```
              __     _
   _    _/__  /./|,//_`
  /_//_// /_|///  //_, outaTiME v.@@version

/* TEAM */
  Web Developer / Graphic Designer: Ariel Oscar Falduto
  Site: http://www.outa.im
  Twitter: @outa7iME
  Contact: afalduto at gmail dot com
  From: Buenos Aires, Argentina

/* SITE */
  Last update: @@timestamp
  Standards: HTML5, CSS3, robotstxt.org, humanstxt.org
  Components: H5BP, Modernizr, jQuery, Twitter Bootstrap, LESS, Jade, Grunt
  Software: Sublime Text 2, Photoshop, LiveReload

```

gulpfile:

```js
var pkg = require('./package.json');
gulp.task('default', function() {
  gulp.src(['src/manifest.appcache', 'src/humans.txt'])
    .pipe(replace({
      patterns: [
        {
          match: 'version',
          replacement: pkg.version
        },
        {
          match: 'timestamp',
          replacement: new Date().getTime()
        }
      ]
    }))
    .pipe(gulp.dest('build'));
});
```

#### Cache busting

File `src/index.html`:

```html
<head>
  <link rel="stylesheet" href="/css/style.css?rel=@@timestamp">
  <script src="/js/app.js?rel=@@timestamp"></script>
</head>
```

gulpfile:

```js
gulp.task('default', function() {
  gulp.src('src/index.html')
    .pipe(replace({
      patterns: [
        {
          match: 'timestamp',
          replacement: new Date().getTime()
        }
      ]
    }))
    .pipe(gulp.dest('build'));
});
```

#### Include file

File `src/index.html`:

```html
<body>
  @@include
</body>
```

gulpfile:

```js
var fs = require('fs');
gulp.task('default', function() {
  gulp.src('src/index.html')
    .pipe(replace({
      patterns: [
        {
          match: 'include',
          replacement: fs.readFileSync('./includes/content.html', 'utf8')
        }
      ]
    }))
    .pipe(gulp.dest('build'));
});
```

#### Regular expression

File `src/username.txt`:

```
John Smith
```

gulpfile:

```js
gulp.task('default', function() {
  gulp.src('src/username.txt')
    .pipe(replace({
      patterns: [
        {
          match: /(\w+)\s(\w+)/,
          replacement: '$2, $1' // replaces "John Smith" to "Smith, John"
        }
      ]
    }))
    .pipe(gulp.dest('build'));
});
```

#### Lookup for `foo` instead of `@@foo`

The `String` matching type or `expression` in `false` generates a simple variable lookup mechanism `@@string`, to skip this mode use one of the below rules ... make your choice:

gulpfile:

```js
gulp.task('default', function() {
  gulp.src('src/foo.txt')

    // option 1 (explicitly using an regexp)
    .pipe(replace({
      patterns: [
        {
          match: /foo/g,
          replacement: 'bar'
        }
      ]
    }))

    // option 2 (easy way)
    .pipe(replace({
      patterns: [
        {
          match: 'foo',
          replacement: 'bar'
        }
      ],
      usePrefix: false
    }))

    // option 3 (old way)
    .pipe(replace({
      patterns: [
        {
          match: 'foo',
          replacement: 'bar'
        }
      ],
      prefix: '' // remove prefix
    }))

    .pipe(gulp.dest('build'));
});
```

## Release History

 * 2014-03-20   v0.0.1   Initial version.

---

Task submitted by [Ariel Falduto](http://outa.im/)