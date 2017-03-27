# gulpfiledemo
var del = require('del');
var gulp = require('gulp');
var inlineNg2Template = require('gulp-inline-ng2-template');
var ts = require('gulp-typescript');
var runsequence = require('run-sequence');
var uglify = require('gulp-uglify');
var rimraf = require('rimraf');
var template = require('gulp-template');
var inject = require('gulp-inject');
var watch = require('gulp-watch');

/**
 * verson control
 */ 
var bump = require('gulp-bump');
var semver = require('semver');
var fs = require('fs');

var paths = {
    build : 'dist/build',
    base : 'src/app/reporting/',
    ts : ['src/app/**/*.ts','!src/app/**/*.spec.ts'],
    test_ts : ['src/app/**/*.spec.ts','!src/app/**/*.ts']
};

gulp.task('hello',()=>{
    console.log("Hello World");
})

gulp.task('clean',() => {
    del(paths.build);
});

gulp.task('clean2',(done)=>{
    rimraf(paths.build,()=>{
        console.log("removed");
    });
});

gulp.task('template',(done)=>{
    gulp.src('src/app/app.component.html')
    .pipe(template({name:'Riveryyyy'}))
    .pipe(gulp.dest('dist/tmp'));
});


/**
 * 可以 用来引入一些静态资源，以及第三方库
 */
gulp.task('inject', ()=>{
    var target = gulp.src('src/index.html');
    var sources = gulp.src(['./src/**/*.js', './src/**/*.css'], {read: false});
    target.pipe(inject(sources))
           .pipe(gulp.dest(paths.build));
});

gulp.task('inline',function(done){
    gulp.src(paths.ts)
    .pipe(inlineNg2Template({useRelativePaths:true}))
    .pipe(ts())
    .pipe(uglify())
    .pipe(gulp.dest(paths.build));
    done;
});

gulp.task('compile-typescript',()=>{
     return gulp.src(paths.ts)
    .pipe(inlineNg2Template({useRelativePaths:true}))
    .pipe(ts())
    .pipe(uglify())
    .pipe(gulp.dest(paths.build));
})

gulp.task('watch',function(done){
    return gulp.watch(paths.ts,['compile-typescript'],{ ignoreInitial: false },()=>{
        console.log('watched');
    })
});

gulp.task('compile',(cb)=>{
    runsequence(
        'hello',
        'clean',
        'inline',
        cb
    );
});

var getPackageJson = function () {
  return JSON.parse(fs.readFileSync('./package.json', 'utf8'));
};
 
// bump versions on package/bower/manifest 
gulp.task('bump', function () {
  // reget package 
  var pkg = getPackageJson();
  // increment version 
  var newVer = semver.inc(pkg.version, 'patch');
 
  return gulp.src('./package.json')
    .pipe(bump({
      version: newVer
    }))
    .pipe(gulp.dest('./'));
});
