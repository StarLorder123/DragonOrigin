# 1.  gulp构建工具
- ## 1.1.  Gulp工具介绍
- Gulp框架旨在将开发流程中让人痛苦或耗时的任务自动化，从而减少所浪费的时间、创造更大价值
- 中文官网：[https://www.gulpjs.com.cn/](https://www.gulpjs.com.cn/)
- ## 1.2.  Gulp基本概念
- ### 1.2.1.  Vinyl
- Vinyl 是描述文件的元数据对象。Vinyl 实例的主要属性是文件系统中文件核心的 path 和 contents 核心方面。Vinyl 对象可用于描述来自多个源的文件（本地文件系统或任何远程存储选项上）。
- Vinyl适配器
- Vinyl 提供了一种描述文件的方法，但是需要一种访问这些文件的方法。使用 Vinyl 适配器访问每个文件源。
- 适配器暴露了：
	- 一个签名为 src(globs, [options]) 的方法，返回一个生成 Vinyl 对象的流。
	- 一个带有签名为 dest(folder, [options]) 的方法，返回一个使用 Vinyl 对象的流。
	- 任何特定于其输入/输出媒体的额外方法-例如 symlink 方法 vinyl-fs 所提供的。它们应该总是返回产生和/或消耗 Vinyl 对象的流。
- ### 1.2.2.  Tasks（任务）
- 每个 gulp 任务都是一个异步 JavaScript 函数，它要么接受一个错误优先回调，要么返回一个流、promise、事件发射器、子进程或observable。由于一些平台限制，不支持同步任务。
- ### 1.2.3.  Globs
- glob 是一串文字和/或通配符，如 *, **, 或 !，用于匹配文件路径。Globbing 是使用一个或多个 globs 在文件系统上定位文件的操作。
- glob base (有时称为 glob parent)是 glob 字符串中任何特殊字符之前的路径段。因此，/src/js/**.js 的 blob base 是 /src/js/。所有匹配 glob 的路径都保证共享 glob base——该路径段不能是可变的。
- 由 src() 生成的 Vinyl 实例是用 glob base 集作为它们的 base 属性构造的。当使用 dest() 写入文件系统时，将从输出路径中删除 base ，以保留目录结构。
- ## 1.3.  关键API的解释
- ### 1.3.1.  src()
- 创建一个流，用于从文件系统读取 Vinyl 对象。
- **注：**BOMs(字节顺序标记)在 UTF-8 中没有任何作用，除非使用 removeBOM 选项禁用，否则 src() 将从读取的 UTF-8 文件中删除BOMs。
  
  ```
  const { src, dest } = require('gulp');
  
  function copy() {
  return src('input/*.js')
    .pipe(dest('output/'));
  }
  
  exports.copy = copy;
  ```
- ### 1.3.2.  dest()
- 创建一个用于将 Vinyl 对象写入到文件系统的流。
  
  ```
  const { src, dest } = require('gulp');
  
  function copy() {
  return src('input/*.js')
    .pipe(dest('output/'));
  }
  
  exports.copy = copy;
  ```
- ### 1.3.3.  series()
- 将任务函数和/或组合操作组合成更大的操作，这些操作将按顺序依次执行。对于使用 series() 和 parallel() 组合操作的嵌套深度没有强制限制。
  
  ```
  const { series } = require('gulp');
  
  function javascript(cb) {
  // body omitted
  cb();
  }
  
  function css(cb) {
  // body omitted
  cb();
  }
  
  exports.build = series(javascript, css);
  ```
- ### 1.3.4.  parallel()
- 将任务功能和/或组合操作组合成同时执行的较大操作。对于使用 series() 和 parallel() 进行嵌套组合的深度没有强制限制。
  
  ```
  const { parallel } = require('gulp');
  
  function javascript(cb) {
  // body omitted
  cb();
  }
  
  function css(cb) {
  // body omitted
  cb();
  }
  
  exports.build = parallel(javascript, css);
  ```
- ### 1.3.5.  task()
- 在任务系统中定义任务。然后可以从命令行和 series()、parallel() 和 lastRun() api 访问该任务。
  
  ```
  const { task } = require('gulp');
  
  task('build', function(cb) {
  // body omitted
  cb();
  });
  
  const build = task('build');
  ```
- ## 1.4.  VSCode中的gulp打包
- 在进行windows平台下的vscode打包时，会执行三条命令，分别是：
	- yarn gulp vscode-win32-x64
	- yarn gulp vscode-win32-x64-inno-updater
	- yarn gulp vscode-win32-x64-user-setup
- 可以根据上面的gulp的描述，看出来，vscode-win32-x64、vscode-win32-x64-inno-updater、vscode-win32-x64-user-setup是gulp暴露出来的三个任务。
- ### 1.4.1.  VSCode中gulp的顺序流程
- 在执行yarn gulp xxx的时候，实际执行的是：node --max_old_space_size=8192 ./node_modules/gulp/bin/gulp.js xxx
- 执行glup相关的内容的时候，首先会执行在项目根目录下的gulpfile.js。这个文件的内容：
  
  ```
  require('./build/gulpfile');
  ```
- 可以看到，这个文件内，引用了build/gulpfile.js这个文件。build/gulpfile.js文件的内容：
  
  ```
  'use strict';
  
  // Increase max listeners for event emitters
  require('events').EventEmitter.defaultMaxListeners = 100;
  
  const gulp = require('gulp');
  const util = require('./lib/util');
  const task = require('./lib/task');
  const { transpileClientSWC, transpileTask, compileTask, watchTask, compileApiProposalNamesTask, watchApiProposalNamesTask } = require('./lib/compilation');
  const { monacoTypecheckTask/* , monacoTypecheckWatchTask */ } = require('./gulpfile.editor');
  const { compileExtensionsTask, watchExtensionsTask, compileExtensionMediaTask } = require('./gulpfile.extensions');
  
  // API proposal names
  gulp.task(compileApiProposalNamesTask);
  gulp.task(watchApiProposalNamesTask);
  
  // SWC Client Transpile
  const transpileClientSWCTask = task.define('transpile-client-swc', task.series(util.rimraf('out'), util.buildWebNodePaths('out'), transpileTask('src', 'out', true)));
  gulp.task(transpileClientSWCTask);
  
  // Transpile only
  const transpileClientTask = task.define('transpile-client', task.series(util.rimraf('out'), util.buildWebNodePaths('out'), transpileTask('src', 'out')));
  gulp.task(transpileClientTask);
  
  // Fast compile for development time
  const compileClientTask = task.define('compile-client', task.series(util.rimraf('out'), util.buildWebNodePaths('out'), compileApiProposalNamesTask, compileTask('src', 'out', false)));
  gulp.task(compileClientTask);
  
  const watchClientTask = task.define('watch-client', task.series(util.rimraf('out'), util.buildWebNodePaths('out'), task.parallel(watchTask('out', false), watchApiProposalNamesTask)));
  gulp.task(watchClientTask);
  
  // All
  const _compileTask = task.define('compile', task.parallel(monacoTypecheckTask, compileClientTask, compileExtensionsTask, compileExtensionMediaTask));
  gulp.task(_compileTask);
  
  gulp.task(task.define('watch', task.parallel(/* monacoTypecheckWatchTask, */ watchClientTask, watchExtensionsTask)));
  
  // Default
  gulp.task('default', _compileTask);
  
  process.on('unhandledRejection', (reason, p) => {
  console.log('Unhandled Rejection at: Promise', p, 'reason:', reason);
  process.exit(1);
  });
  
  // Load all the gulpfiles only if running tasks other than the editor tasks
  require('glob').sync('gulpfile.*.js', { cwd: __dirname })
  .forEach(f => require(`./${f}`));
  ```
- 可以看到最后一行，会将所有的gulpfile.*.js中的任务全部发布出来。
- 而三条命令中执行的第一条命令的任务名称在gulpfile.vscode.js文件中的455行左右。
  
  ```
  const vscodeTask = task.define(`vscode${dashed(platform)}${dashed(arch)}${dashed(minified)}`, task.series(
  	compileBuildTask,
  	compileExtensionsBuildTask,
  	compileExtensionMediaBuildTask,
  	minified ? minifyVSCodeTask : optimizeVSCodeTask,
  	vscodeTaskCI
  ));
  gulp.task(vscodeTask);
  ```
	- 可以看到我们执行第一个命令的之后，会依次执行上述的五个任务来进行打包。
- 而剩下来的两个任务，则是在gulpfile.vscode.win32.js中定义的。
- 分别在129行附近和159行附近。可以看到分别依次执行了两个任务。