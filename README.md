gulp-replay
===========

gulp-replay is a caching layer for streams.


# Example

```javascript
var gulp = require('gulp')
  , tasks = require('gulp-load-tasks')()
  , rework = require('gulp-rework')
  , es = require('event-stream')
  , File = require('vinyl')
  , dev = true;
  
function maybeWatch(pattern, fn) {
  dev && gulp.watch(pattern, function(ev) {
    if (ev.type !== 'deleted')
      fn(gulp.src(ev.path));
    else {
      // Necessary to make remove's work properly
      // with gulp-replay
      var stream = es.pause();
      fn(stream);
      stream.write(new File({
        contents: new Buffer(''),
        path: ev.path
      }));
      stream.end();
    }
  });
  
  fn(gulp.src(pattern));
}

gulp.task('styl', function() {
  var pattern = 'lib/**/*.styl'
    , replayCache = replay();

  maybeWatch(pattern, function(stream) {
    stream
      .pipe(tasks.rework(
        rework.mixin(require('rework-mixins')),
        rework.ease(),
        rework.colors(),
        rework.references(),
        rework.at2x(),
        rework.extend(), 
        {sourcemap: true}))
      .pipe(tasks.autoprefixer('last 2 versions'))
      .pipe(replayCache())
      .pipe(tasks.concat('build.css'))
      .pipe(gulp.dest('public'))
  });
});
```

Recompiles only the styl file that changed.  The compiled versions of the others are simply re-emitted by replayCache.
