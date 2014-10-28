---

layout: post
title:  简单描述 heaps(h3d)
date:   2014-05-13 8:26:10
categories: haxelib

---

目前已经更名为 heaps, 看描述似乎要做成一个游戏引擎. 一些改动也比较大,  [**`heaps`**](https://github.com/ncannasse/heaps) 当目标平台为 flash 时将基于 flash.stage3D, 而 平台为 js 时, 将使用 webgl. 如果需要使用以前旧的 h3d 版本, 在 github 页面上找到 h3d 这个分支即可. 


注意: 下边的文档描述为 heaps的 h3d 分支.

<!-- more -->

### h3d

 * **Engine** 

	> 构选函数的 antiAlias 参数,似乎不会起任何作用,像是没实现.
  
	> 一些基础设置,如修改背影颜色,或设置全屏...

 * impl 文件夹

  - **Driver** 核心类不同平台接口.

### h2d

 h2d 似乎在设计上就是作为 UI, 因此一些动画,碰撞检测的方法都比较底层, 可以使用 XML(组件布局) 和 CSS(样式) 配置 UI.

 h2d 游戏源码示例:

 * [ld-25 You are the Villain](https://github.com/ncannasse/ld25)

 * [ld-27 10 seconds](https://github.com/ncannasse/ld27)

 * [ld-28 You only got one](https://github.com/ncannasse/ld28)

 * [ld-30 Connected Worlds](https://github.com/ncannasse/ld30)

#### 源码简析

 * **Scene** 2D 场影.
  
  - 通过 setFixedSize(w, h) 来设置一个宽高的基准值, 每次 resize 事件,将会自动以这个值进行缩放.

		> 所以当出现缩放时 h2d.Scene 的 width,height 和 h3d.Engine,并不一致.对于 像素类的 2D 游戏,通常 会设一个很小的基准高宽值,然后缩放到指定大小. 这类游戏 Engine 的高宽值一般为 Scene 的二倍.



 * **Interactive** 用于交互.

	```haxe
	// 如何给 h2d.Bitmap 添加事件.
	var bmp = new h2d.Bitmap(Res.some_png.toTile(), s2d );

	var it = new Interactive(s2d.width, s2d.height, bmp);
	it.onClick = function(e : hxd.Event){
		trace(e); // ERelease[e.relX,e.relY]
	}
	```	

 * comp

  - **Context**

		```haxe
		// 在初使化 h3d 之前可以修改组件默认的 css 
		public static var DEFAULT_CSS = hxd.res.Embed.getFileContent("h2d/css/default.css");
		// 优先返回缓存 字体
		public static function getFont( name : String, size : Int ):hxd.Font{}
		// 优先返回缓存 Tile
		public static function makeTileIcon( pixels : hxd.Pixels ) : h2d.Tile {} 
		```
  - **Parser** HTML 解析器. 

		> 这个类使用了 **`hscript`** 来动态解析 html 标签上的 脚本.
		
		> 示例: `sample/comps/components.html`, `tools/perlin/PerlinView.hx`, `h3d/parts/Editor.hx`
		
  - **JQuery** 可以在 HTML 标签上写脚本

 * css

  - **Parser** css 解析器.
		
		> 和普通的 css 不一样的是 `:hover` 其实也是解析成 className. 例如:当光标悬浮于组件上时, 会自动执行  addClass(":hover")
		
		> 示例参见 default.css
  
  - **Style** 包含解析后的 CSS 属性对象

  - **Defs** 一些 CSS 值数据定义,用于需要给 style 的一些属性赋值时.

  - **Engine** 将 样式(Style) 应用到 组件(Component) 上去.

  - **default.css** 组件默认的样式配置. 在初使化之前可以通过 comp.Context.DEFAULT_CSS 修改默认组件样式

### hxd
 
 * **Res** 资源管理

	> 定义`-D resourcesPath=dir` 之后,Res.hx 相关宏构建将描描资源文件夹,让 IDE 有智能提示

	> 上行操作并不会嵌入文件,需要主动调用 Res.initEmbed() 将文件嵌入到 flash.

	> 注意:Res类 initEmbed之后默认自动嵌入的字体只包含 Charset.DEFAULT_CHARS

	```haxe
	// hxd/res/FileTree.hx::handleFile() 可以看到资源被解析成的类型.
	switch( ext.toLowerCase() ) {
		case "jpg", "png":
			return { e : macro loader.loadImage($epath), t : macro : hxd.res.Image };
		case "fbx", "xbx", "xtra":
			return { e : macro loader.loadFbxModel($epath), t : macro : hxd.res.FbxModel };
		case "awd":
			return { e : macro loader.loadAwdModel($epath), t : macro : hxd.res.AwdModel };
		case "ttf":
			return { e : macro loader.loadFont($epath), t : macro : hxd.res.Font };
		case "fnt":
			return { e : macro loader.loadBitmapFont($epath), t : macro : hxd.res.BitmapFont };
		case "wav", "mp3":
			return { e : macro loader.loadSound($epath), t : macro : hxd.res.Sound };
		case "tmx":
			return { e : macro loader.loadTiledMap($epath), t : macro : hxd.res.TiledMap };
		default:
			return { e : macro loader.loadData($epath), t : macro : hxd.res.Resource };
	}

	```

 * **App** 主应用

	> 继承这个类,从而快速调用 h3d

 * **Tile** 

 	> 为了避免 dispose, 应该尽量使用 sub 方法从主 Tile 上新建一个出来.

	> scaleToSize(w, h) 是个不错的方法.例如: 当需要 32x32 的图,直接 scaleToSize(32,32) 就行了.

 * **Texture** 

 * **System**

   - 包含 enum Cursor, 用于定义 鼠标的不同形状, 允许自定义

   - 所有方法或属性都是静态类型. 参考 API 或源码.

 * Stage 对 flash.display.Stage 进行了包装.


 * **Event**  h2d.Interactive 事件参数类型.

 * res 文件夹

  - 这个目录大多数都属于 Resource 的了类.

  - **Image** 图片资源解析,只支持 jpg 和 png.

		> 将图片转出为 hxd/res 下的各种格式。

  - **Embed** 

		> 例如: 宏方法 `Embed.embedFont("nokiafc22.ttf");` 将资源文件夹或系统字体文件夹下的字体嵌入到 SWF. 

		> 不支持 otf 类型.嵌入后通过调用 FontBuilder.getFont() 运行时方法,将指字的字符一个一个地画到Tile上. 字体嵌入不是必须的.

		> 宏方法 getFileContent(path2name) 一个从文件中加载字符

		> 宏方法 getResource(path2name) 返回一个 hxd.res.Any 对象.这个方法还展现了如何将一个 Bytes 的变量传递给回宏返回.

  - 使用设备字体输出中文字符,只适用于 flash

		> 由于中文字符太大(3750个字符),在嵌入时发生了 IO 错误.

		```haxe
		// CN_STR 为常用中文字符(3750).
		// FontBuilder 其实一个字符一个字符的 draw 到 bitmapData 中
		var font = FontBuilder.getFont("simfang", 16, { chars: MRes.CN_STR,antiAliasing : false} );

		```
  - **Sound**

		> 不知道,为什么不用 @:sound("file.wav|mp3") 的方式嵌入???
		
		> 这个类播放声音是以 Bytes 的方式加载音乐的.

  - **FileTree**  扫描资源文件夹,res 主要的宏构建类.

		> 如果打算将 wav 转换成 mp3(if options.compressSounds), 则需要下载 `lame` 转换器,并且添加到路径,  另外`LocalFileSystem.hx` 文件(仅限于本地文件系统),LocalEntry 类的方法 convertToMP3,这里需要修改 lame 的正确路径. 结论: 最好在外部先调用命令行转换成 mp3 再放入资源目录.
		
  - **Loader**  建立在 各文件类 上的一个方法汇总

  - **Any**  建立在 Loader 上的一个方法汇总


 * impl 文件夹

  - **Memory**  管理 domainMemory

		> 使用 flash.Memory 前先 select(bytes) ,完了后 调用 end().
		
  - **Tmp**  Bytes 缓存池

		> getBytes(size) 		// 将优先从缓存池里返回Bytes
		
		> saveBytes(bytes)	// 将bytes 释放回缓存池内
  
  - **`FastIO<T>`**  像是一个抽像化的数组.优先会使用 Vector 替代 Array。

  - **FastIntIO**  继承 `FastIO<Int>`, 多了一些针对 **位** 的操作方法.
				
  - **ArrayIterator**  继承这个类,方便 for 便历.

		```haxe
		// from h2d/Sprite.hx
		public inline function iterator() : hxd.impl.ArrayIterator<Object>{
			return new hxd.impl.ArrayIterator(childs);
		}
		```
 * poly2tri 文件夹  多边形 to 三角形???
					 
<br />



#### `tool`

 * `fbx`

	> 可以用来预览 `.fbx`.

 * `parts`

	> `air` 粒子生成器. 因为要保存文件,需要在 AIR 中运行.
	
	```haxe
	// %flex_sdk%\bin\adl parts.app
	import h3d.parts.Data;
	// extends hxd.App
	// default.p 为粒子生成器保存的文件.
	var state:State = Unserializer.run(Embed.getFileContent("res/default.p"));
	// 可以更换 texture
	state.frames = [State.defPart.toTile()];
	
	var emit = new Emitter(state, s3d);	
	```

 * `perlin`

	> flash perlin.
	
	> h2d ui 示例
	
 * `xbx`

	> `neko` 将 `.fbx` 文件压成 `xbx`格式.
	
	> 这是一个很好的如何使用 `format.zip` 和 如何创建一个简单命令行程序的示例



