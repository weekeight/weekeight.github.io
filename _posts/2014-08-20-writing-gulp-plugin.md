---
layout: post
title:  "如何编写Gulp插件&gulp-kmd"
date:   2014-08-20 00：59
categories: nodejs javascript
tag: gulp nodejs javascript
---

## gulp简单介绍

`gulp`正如官网所说的那样是一个`The streaming build system`，一个充分利用了`Node.js Streams API`的一个构建任务管理工具，更简单清爽的插件使用习惯。ATA上*@剑平*也介绍过`gulp`的简单用法([传送门](http://www.atatech.org/articles/13458))

## grunt vs gulp

`grunt`是目前最流行的前端构建工具了，拥有强大的社区，丰富多样的插件。个人觉得`grunt`与`gulp`两者无所谓*好*与*坏*，各自都有自己的特点和缺点，如何选择其实更加偏向个人喜欢什么样的编码风格。关于两者的区别分析详情有兴趣的可以看下[这篇文章]()，下面主要做一下两者区别的总结：

- `grunt`拥有强大的社区，`gulp`发展在前者之后，社区相对差一些
- `grunt`拥有非常多的插件，`gulp`插件覆盖度没那么广，但是质量相对高一下，[常用的插件](http://gulpjs.com/plugins/)也差不多都有
- `grunt`相对有点配置‘优先’的感觉，插件配置较多，而`gulp`更好地诠释了[**code over configuration**]()，插件使用清爽优雅
- `grunt`相对频繁使用`IO`，而`gulp`利用`stream`把一切都用`pipe`(管道)优雅方便地串连起来了。

## gulp简单插件编写基础

关于`gulp`插件编写，*@剑平*也有写过一篇相似的文章[Gulp.js深入讲解](http://www.atatech.org/articles/14002)介绍。`gulp`插件编写最重要的是理解`Node.js`中的三种`stream` : `readable streams`,`writeable streams`,`transform stream`，以及最常用的`through2`,`gulp-util`模块，还有`gulp`提供的几个API：`gulp.src`,`gulp.dest`。下面先对这些需要使用的基本模块，API做个介绍，最后通过一个简单实例来讲述一个`gulp`插件如何编写。

在`node.js`里面有四种`stream` : `readable streams`(只读的流),`writeable streams`（只写的流）,`transform stream`（读/写流）,`duplex streams`（循环流，这里不讨论）。

在`node.js`里面最简单的写法就是：

	var fs = require('fs');
	fs.createReadStream('xx.js').pipe(fs.createWriteStream('yy.js'))
	

意指将一个文件以*只读流*的方式读取内容再通过`pipe`（管道）导入一个*只写流*写进文件。

那么，如果我们需要在`只读流`和`只写流`间做一些修改再通过`pipe`写到最终文件里面应该怎么做呢？这也就需要使用`读写流`来充当一个类似*中转站*效果了，而在`gulp`插件编写中，最常用的就是使用`through2`这个模块来替代`node.js`原生的`transform stream`了。（注：gulp官网示例用的就是这个模块，[Gulp.js深入讲解](http://www.atatech.org/articles/14002)这篇文章介绍的`gulp-clean`用的是`event-stream`）。

[through2](https://github.com/rvagg/through2)其实就是对`transform stream`做了一层简单封装。在这里有必要说一下 through2() 和 through2.obj() 的区别，用法上面其实就是差了一个配置  through2({ objectMode: true }  == through2.obj()，那么不禁会问这个配置的意义是什么了，根据[node.js Object Mode API](http://nodejs.org/documentation/api/)解释，个人理解 through2.obj 能让我们在函数transformFunction里面对一个一个文件进行处理，而不只是片段，所以gulp插件中几乎都是使用through.obj这个函数的（这点不知是否理解有误，清楚的朋友希望能告诉我一下）

`gulp.src`其实就是用[node-glob](https://github.com/isaacs/node-glob)去读取相应文件并创建一个简单封装后的流能通过`pipe`传给后面处理

## gulp插件编写示例 ~ gulp-kmd

一个简单的gulp插件需要多少行代码？我的回答是只要**24**行代码，对的，就是24行代码。今天在做`kissy editor-plugin`的构建的时候，用到*@杰少*写的kmd和gulp-kmc这两个工具，但是苦于没有gulp-kmd，默认的kmd模块只能在命令行下使用，不符合简化gulpfile.js的原则。于是乎看了下kmd源码提供的接口，找到`kmd.kissy2cmd.parse`这个接口就开始做一个极简单的gulp插件了，以后喜欢使用gulp的童鞋也可以免去那二十几行代码写在gulpfile.js里面了。

[gulp-kmd传送门](https://www.npmjs.org/package/gulp-kmd)

代码比较少，直接贴代码了，解释写在代码注释中：

	var kmd = require('kmd'),
	through2 = require('through2'),
	gutil = require('gulp-util'),
	PluginError = gutil.PluginError;  //官网强烈推荐标准的错误处理模块
	var kissy2cmd = kmd.kissy2cmd;
	const PLUGIN_NAME = 'gulp-kmd';
	function confirmCmd(){
		var stream = through2.obj(function(file, enc, callback){
			//这里的file就是一个对象，有三个属性 contents（Buffer，文件内容）、base（路径）、path（路径）
			//enc指编码
			if(file.isNull()){  //如果文件为空则不处理
				return callback();
			}
			if(file.isStream()){
				this.emit('error', new PluginError(PLUGIN_NAME, 'Streams are 				not supported!'));  //规范的报错处理
      			return cb();
			}
			var fileInCmd = kissy2cmd.parse(file.contents.toString(),				{ fromString : true });
			file.contents = new Buffer(fileInCmd);
		 	this.push(file);  //把内容放到流中使得下一个任务可以处理
			callback();  //一定要调用callback告诉这个文件处理完了
		});
		return stream;
	}
	module.exports = confirmCmd;
	
此外，说明一下，如果直接用fs.createReadStream会是怎样呢？如下代码：

	var fs = require('fs'),
	through = require('through2');
	fs.createReadStream('test.txt')
		.pipe(through.obj(function(file, enc, cb){
			//这样拿到的file纯粹是一个文件的Buffer，和gulp.src拿到的不一样，也是gulp.src是对stream做了一个简单封装的表现
			debugger;
		}))
		
## 总结

编写一个gulp插件还是比较简单的，重要的是先要把gulp的常用几个API熟悉，以及对stream的理解，还有就是多看别人的源码和多动手debug。要获取更多的信息请移步至[gulp官网](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/README.md)看吧。