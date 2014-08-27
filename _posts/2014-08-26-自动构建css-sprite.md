---
layout: post
title:  "自动构建css sprite"
date:   2014-08-28 01：18
categories: nodejs javascript
tag: gulp nodejs javascript css-sprite
---

## 背景

css sprite是我们前端开发再熟悉不过的网页图片处理方式了，这里也不再重复其优势。但是在使用css sprite总会遇到很多问题，例如头痛的拼图定位，css样式微调等，特别是面临项目开发时间紧张，UI设计图总要分期提供，多人协同开发又要考虑是否将所有小图标合到同一个拼图里，是否需要将拼图分组，考虑如何排版才够优雅有序等等。我们总不能每次UI设计给了一个小图又打开PS粘贴图层慢慢地在调整，再导出，这样的工作太重复繁琐了。

## 需求

是否有一个工具能让我们在开发阶段像往常一样不用考虑拼图，非常自然地编写css，单独地引用小图，能够应对随时可能改变的UI设计给的小图，同时又能够人性化地确定小图之间的间距，排版，能兼容IE6这个不支持透明背景图的问题最好了，还有，要是能用less来编写那就完美了。答案是有的，网上现在也有各种的css sprite的构建工具来处理这些事情，下面我们就来看看这些工具的优劣，是否能满足我们的需求。

## css sprite构建工具介绍

[**css-sprite**](https://www.npmjs.org/package/css-sprite)相对来说是一个很强大的小图标合并工具了，支持 less , sass , stylus ,还有 gulp 和 grunt 插件，简单的用法如下：

	//use with gulp
	var gulp = require('gulp');
	var gulpif = require('gulp-if');
	var sprite = require('css-sprite').stream;
	// generate sprite.png and _sprite.scss
	gulp.task('sprites', function () {
	  return gulp.src('./src/img/*.png')
	    .pipe(sprite({
	      name: 'sprite.png',
	      style: 'sprite.less',
	      cssPath: './img',
	      processor: 'less'
	    }))
	    .pipe(gulpif('*.png', gulp.dest('./dist/img/'), gulp.dest('./dist/less/')))
	});

	//生成合拼后的图片和对应的less文件sprite.less后，再在我们自己的less文件 @import 进去使用
	@import 'sprite'; // the generated style file (sprite.less)
	// camera icon (camera.png in src directory)
	.icon-camera {
	  .sprite(@camera);
	}

	// cart icon (cart.png in src directory)
	.icon-cart {
	  .sprite(@cart);
	}
	
通过上面，我们对css-sprite可以得出以下结论：
优点 ： 功能强大，支持less,sass,stylus，还有gulp/grunt插件，支持 retina屏
缺点 ： 需要我们手动引入生成后的css/less/sass文件，再在相应的地方引用得到的变量的样式，不够灵活，开发阶段编写css代码不自然，不支持png8（ie6不适用）；依赖[node-canvas](https://github.com/learnboost/node-canvas)和[Cairo](http://cairographics.org/)来处理图片，这里面的依赖安装很麻烦，稍不留神就安装出错。


[**smart sprite**](http://csssprites.org/)应该是发展了比较久的一个css sprite构建工具了，该有的功能几乎都有了，最近还增加支持了png8。简单实用例子：

	/** sprite: mysprite; sprite-image: url('../img/mysprite.png'); sprite-layout: vertical */
	#web {
	width: 17px; height: 17px;
	background-repeat: no-repeat;
	background-image: url(../img/web.gif); /** sprite-ref: mysprite; */
	}

	#logo {
	width: 50px; height: 50px;
	background-repeat: no-repeat;
	background-position: top right;
	background-image: url(../img/logo.png); /** sprite-ref: mysprite; sprite-alignment: right */
	}

	#main-box {
	background-repeat: repeat-x;
	background-position: 5px left;
	background-image: url(../img/top-frame.gif); /** sprite-ref: mysprite; sprite-alignment: repeat; sprite-margin-top: 5px */
	}

	//将生成如下css代码
	#web { 
	width: 17px; height: 17px; 
	background-repeat: no-repeat; 
	background-image: url('../img/mysprite.png');
	background-position: left -0px;
	} 

	#logo { 
	width: 50px; height: 50px; 
	background-repeat: no-repeat; 
	background-position: top right; 
	background-image: url('../img/mysprite.png');
	background-position: right -17px;
	} 

	#main-box { 
	background-repeat: repeat-x; 
	background-position: 5px left; 
	background-image: url('../img/mysprite.png');
	background-position: left -64px;
	}
优点： 支持png8，支持自定义分组，排版，repeat方式，自定义图片间距等,会修改原css文件得到处理后的css文件，开发阶段编写代码不用手动引用生成的css文件了。
缺点： 依赖Java环境，意味着要装JDK，对前端童鞋来说可能不是那么好玩；需要在每张图片后面写上注释语法使得smart sprite能够识别，并根据语法生成图片，样式；

[**gopng**](http://alloyteam.github.io/gopng/)是腾讯alloyteam开发的一个在线的css sprite生成工具，用canvas来实现，需要手动拖动图片到画布，再生成图片和样式导出，虽省去用PS来调整但是感觉还是太繁琐了。


[**joycss**](http://joycss.org/)是我们大阿里@翰文童鞋开发的一个神器，使用简单，功能强大。支持x/y方向图片平铺，自定义位置间距，png8，支持使用图片url参数来定义图片生成规则，在淘宝内部使用还可以使用upload接口直接把图片传到tps上面省去了自己上传的功夫。总的来说，恰好满足上面所提到的各种需求，也是我目前为止发现的使用最简单，最符合实际需求的一个自动拼图工具了。具体的使用方法可以到[joycss](https://github.com/shepherdwind/joycss)上面看。

[**gulp-joycss**](https://www.npmjs.org/package/gulp-joycss)是我在做[kissy editor-plugins](https://github.com/kissyteam/editor-plugins)构建的时候使用joycss写的一个插件，当时joycss不支持引用png8，生成的css样式也有部分丢失等问题，也只有grunt-joycss插件而已。gulp-joycss的简单使用如下：

	var gulpJoycss = require('gulp-joycss);
	gulp.src('assets/*.less')
    .pipe(gulpJoycss({
        'editor.less' : {  //对应文件的相关配置，每一个文件都必须配置
            imgPath : 'build/assets',   //生成后的图片位置
            dest : 'build/assets/editor.css',  //生成后的文件路径
            prefixUrl : '/editor-plugins/' + packageJson.version + '/assets'   //压缩后的css(用于发布)，引用生成的sprite图的地址前缀（图片在服务器上面的相应位置），而debug文件则不加前缀，用于调试
        },
        'iframe.less' : {  //如果当前文件不需要生成图片，仅是转化less，则不需要配置imgPath
            dest : 'build/assets/iframe.css'
        }
    }))
    .pipe(gulp.dest('build/assets'));
    
总结：自动拼图工具真的很多，开始的时候自己在寻找符合需求的工具花了不少时间，每个工具用起来了才发现这里不符合那里又有问题，总找不到一个完全符合需求的工具。其实想想，与其花那么时间去尝试别人写的工具是否符合自己的需求还不如选一个接近的工具动手修改成符合自己需求的，这样或许收获更大，效率更高！最后，真心感谢@翰文的支持！