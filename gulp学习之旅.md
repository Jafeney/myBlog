<p style="text-indent: 1.5em">自nodeJS登上前端舞台，自动化构建变得越来越流行。目前最流行的当属grunt和gulp,这两个光看名字挺像，功能也差不多，不过gulp能在grunt这位大哥如日中天的境况下开辟出自己的一片天地，有着她独到的优点。
</p>
<ol>
    <li><strong>易用</strong> Gulp相比Grunt更简洁，而且遵循代码优于配置策略，维护Gulp更像是写代码。</li>
    <li><strong>高效</strong> Gulp相比Grunt更有设计感，核心设计基于Unix流的概念，通过管道连接，不需要写中间文件。</li>
    <li><strong>高质量</strong>
    Gulp的每个插件只完成一个功能，这也是Unix的设计原则之一，各个功能通过流进行整合并完成复杂的任务。例如：Grunt的imagemin插件不仅压缩图片，同时还包括缓存功能。他表示，在Gulp中，缓存是另一个插件，可以被别的插件使用，这样就促进了插件的可重用性。目前官方列出的有673个插件。
    </li>
    <li><strong>易学</strong>
    Gulp的核心API只有5个，掌握了5个API就学会了Gulp，之后便可以通过管道流组合自己想要的任务。
    </li>
    <li><strong>流</strong>
    使用Grunt的I/O过程中会产生一些中间态的临时文件，一些任务生成临时文件，其它任务可能会基于临时文件再做处理并生成最终的构建后文件。而使用Gulp的优势就是利用流的方式进行文件的处理，通过管道将多个任务和操作连接起来，因此只有一次I/O的过程，流程更清晰，更纯粹。
    </li>
    <li><strong>代码优于配置</strong> 维护Gulp更像是写代码，而且Gulp遵循CommonJS规范，因此跟写Node程序没有差别。</li>
</ol>

<strong>一个简单的Gulpfile.js配置格式</strong>
```JavaScript
    var gulp = require('gulp');
    var jshint = require('gulp-jshint');
    var concat = require('gulp-concat');
    var rename = require('gulp-rename');
    var uglify = require('gulp-uglify');

    // Lint JS
    gulp.task('lint', function() {
    return gulp.src('src/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'));
    });

    // Concat & Minify JS
    gulp.task('minify', function(){
        return gulp.src('src/*.js')
        .pipe(concat('all.js'))
        .pipe(gulp.dest('dist'))
        .pipe(rename('all.min.js'))
        .pipe(uglify())
        .pipe(gulp.dest('dist'));
    });

    // Watch Our Files
    gulp.task('watch', function() {
        gulp.watch('src/*.js', ['lint', 'minify']);
    });

    // Default
    gulp.task('default', ['lint', 'minify', 'watch']);
```













