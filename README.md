yt-gulp-rev-add-urlhash
================

This Plugin is used to purge the caches of html-imports by adding a hash with a version. To alter the gulp-rev-replace plugin was necessary because the replacement of urls by urls with version-queries ( f.e. "../tui-application.html?v=wl234l" ) had siteeffects simply using `indexOf`. Now line by line is processed and regular Expressions looking for `<link>` and `<script>` - Tags are used to make deadly sure only the indended urls are replaced with the same url extended by the url-hash-query. Also manually with a fixed query-string ( tui-application.html?v=####CACHE#### ) marked dependencies are processed.



A copy of gulp-rev-replace, add relative path replacement support. Due to side-effects with relative urls and addition of url-hashes the gulp-assetpaths plugin was added partially to step line by line through the files and use regex to only replace occurences in specific templates. ( <link rel="import"> - Tag. )

add `options.base` for relative path caculation.

add `options.suffix`

add `options.noRev` to replace relative path to absolute path without adding rev

add `options.relative` not to change path to absolute

## Install

```bash
$ npm install --save-dev gulp-rev-replace-relative
```


## Usage

Pipe through a stream which has both the files you want to be updated, as well as the files which have been renamed.

For example, we can use [gulp-useref](https://github.com/jonkemp/gulp-useref) to concatenate assets in an index.html,
and then use [gulp-rev](https://github.com/sindresorhus/gulp-rev) and gulp-rev-replace to cache-bust them.

```js
var gulp = require('gulp');
var rev = require('gulp-rev');
var revReplace = require('gulp-rev-replace');
var useref = require('gulp-useref');
var filter = require('gulp-filter');
var uglify = require('gulp-uglify');
var csso = require('gulp-csso');

gulp.task("index", function() {
  var jsFilter = filter("**/*.js", { restore: true });
  var cssFilter = filter("**/*.css", { restore: true });
  var indexHtmlFilter = filter(['**/*', '!**/index.html'], { restore: true });

  return gulp.src("src/index.html")
    .pipe(useref())      // Concatenate with gulp-useref
    .pipe(jsFilter)
    .pipe(uglify())             // Minify any javascript sources
    .pipe(jsFilter.restore)
    .pipe(cssFilter)
    .pipe(csso())               // Minify any CSS sources
    .pipe(cssFilter.restore)
    .pipe(indexHtmlFilter)
    .pipe(rev())                // Rename the concatenated files (but not index.html)
    .pipe(indexHtmlFilter.restore)
    .pipe(revReplace())         // Substitute in new filenames
    .pipe(gulp.dest('public'));
});
```

It is also possible to use gulp-rev-replace without gulp-useref:

```js
var rev = require("gulp-rev");
var revReplace = require("gulp-rev-replace");
gulp.task("revision", ["dist:css", "dist:js"], function(){
  return gulp.src(["dist/**/*.css", "dist/**/*.js"])
    .pipe(rev())
    .pipe(gulp.dest(opt.distFolder))
    .pipe(rev.manifest())
    .pipe(gulp.dest(opt.distFolder))
})

gulp.task("revreplace", ["revision"], function(){
  var manifest = gulp.src("./" + opt.distFolder + "/rev-manifest.json");

  return gulp.src(opt.srcFolder + "/index.html")
    .pipe(revReplace({manifest: manifest}))
    .pipe(gulp.dest(opt.distFolder));
});
```


## API

### revReplace(options)

#### options.canonicalUris
Type: `boolean`

Default: `true`

Use canonical Uris when replacing filePaths, i.e. when working with filepaths
with non forward slash (`/`) path separators we replace them with forward slash.

#### options.replaceInExtensions
Type: `Array`

Default: `['.js', '.css', '.html', '.hbs']`

Only substitute in new filenames in files of these types.

#### options.prefix
Type: `string`

Default: ``

Add the prefix string to each replacement.

#### options.manifest
Type: `Stream` (e.g., `gulp.src()`)

Read JSON manifests written out by `rev`. Allows replacing filenames that were
`rev`ed prior to the current task.

#### options.modifyUnreved, options.modifyReved
Type: `Function`

Modify the name of the unreved/reved files before using them. The filename is
passed to the function as the first argument.

For example, if in your manifest you have:

```js
{"js/app.js.map": "js/app-98adc164.js.map"}
```

If you wanted to get rid of the `js/` path just for `.map` files (because they
are sourcemaps and the references to them are relative, not absolute) you could
do the following:

```js
function replaceJsIfMap(filename) {
    if (filename.indexOf('.map') > -1) {
        return filename.replace('js/', '');
    }
    return filename;
}

return gulp.src(opt.distFolder + '**/*.js')
    .pipe(revReplace({
        manifest: manifest,
        modifyUnreved: replaceJsIfMap,
        modifyReved: replaceJsIfMap
    }))
    .pipe(gulp.dest(opt.distFolder));
```

## Contributors

- Chad Jablonski
- Denis Parchenko
- Evgeniy Vasilev
- George Song
- Håkon K. Eide
- Juan Lasheras
- Majid Burney
- Simon Ihmig
- Vincent Voyer
- Bradley Abrahams


## License

[MIT](http://opensource.org/licenses/MIT) © [James K Nelson](http://jamesknelson.com)
