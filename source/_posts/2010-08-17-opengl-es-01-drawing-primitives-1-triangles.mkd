--- 
categories: 
  - iOS
comments: true
layout: post
published: true
status: publish
tags: 
  - iOS
  - "OpenGL ES"
title: "OpenGL ES 01 - 绘制基础图形1 - 三角形"
type: post
---
原作： Simon Maurice

基础图形就是组成复杂对象的基础对象。在 OpenGL ES 中基础图形包括了点、线以及三角线。那啥，大家都知道它们长啥样吧？

我们直接从代码入手吧。当我们知道这些是怎么回事了之后就可以开始写自己的代码了。

<h3>基础图形#1 ——— 三角形</h3>

三角形是基础图形里最“复杂”的一个了，不过到处都用得着，而且用起来也很方便。三角形应该是你画的第一个 OpenGL 基础图形。绘制的时候，我们只需要告诉 OpenGL 表示三角形的三个顶点坐标，然后它就欢快地渲染去了。

我们接着上次的 OpenGL ES 00 教程开始吧，或者你也可以在这里下载代码：<a href="https://dl.dropbox.com/s/fu8bxmvn0inrhdj/AppleCoder-OpenGLES-00.zip?dl">AppleCoder-OpenGLES-00.zip</a>。在 Xcode 中打开项目，然后直接找到 EAGLView.m 文件中的 drawView 方法。让我们再重复下刘谦同学的那句话吧：这是见证奇迹的时刻！

首先我们得把三角形定义出来。在 OpenGL 中有两种坐标：模型坐标和世界坐标。模型坐标指的是我们要画的这个图形的实际坐标，而世界坐标则告诉 OpenGL 模型坐标相对于观察者来说在哪（观察者在世界坐标中总是在 (0.0, 0.0, 0.0) 位置）。 

我们的第一个例子正好可以演示这一点。首先我们用 3 个 3D 的 (X, Y, Z) 坐标来定义三角形：



``` 
const GLfloat triangleVertices[] = {
	0.0, 1.0, -6.0,  // Triangle top centre
	-1.0, -1.0, -6.0,// bottom left
	1.0, -1.0, -6.0, // bottom right
};
```



注意，我们定义的这 3 个点是以逆时针顺序描述的。当然你也可以改用顺时针的顺序来定义，但是我们必须保证它们顺序的一致性。我建议你开始时还是用逆时针顺序，因为在后面我们使用一些更高级的功能的时候需要这么做。

虽然说这个系列纯粹是为了 iPhone OpenGL ES 而写，不过对于新入行的朋友们我还是简单地描述下三维坐标系统吧。看下这张图：

<img src="http://codeleaks.files.wordpress.com/2010/08/0101.jpg" alt="0101.jpg" title="0101.jpg" border="0" width="456" height="400">

好吧我知道我画得很搓，画图什么的最讨厌了。不过模型空间或者世界空间看起来就是这张图上的样子了。想象一下这就是你的电脑屏幕，X 和 Y 分别表示横轴和纵轴，而 Z 表示的则是深度。坐标系的中心点是 (0.0, 0.0, 0.0)。

于是看看我们在上面描述的三角形。第一个点 (0.0, 1.0, -6.0) 的位置在 Y 轴的中心点，在 X 轴上方 1 个点，然后在屏幕往后 6 个点。第二个点在 Y 轴右边 1 个点，X 轴下方 1 个点（即 Y 值为 -1.0），仍然屏幕往里 6 个点。同理适用于第三个点。

为什么我们要把物体往后放（给其负的 Z 值）呢？是因为只有这样它才能显示出来（记住，视察者，或者说“摄像机”的位置在 (0.0, 0.0, 0.0)），否则它将无法通过 OpenGL 的深度测试，也就压根不会被渲染出来。

有些好学的小朋友要问了：“喂，我们不是应该用模型坐标嘛，怎么你丫给的看上去是个世界坐标！”你说得没错，但是当我们渲染三角形的时候，OpenGL 会把物体放到 (0.0, 0.0, 0.0) 上，所以我们必须它放到屏幕后面使其可见。当我们往后讲到变换（移动、旋转等）后，你会发现我们并不一定要把它的 Z 值设成负数。不过在此之前还是让我们先把 Z 值设成 -6.0 吧。

<h3>绘图代码</h3>

我们已经能描述三角形了。接下来我们就要告诉 OpenGL 我们的数据在哪里，以及怎么样去绘制这个图形。这个也就是几行代码的事情。回到 drawView 方法中，照着下面代码实现：



``` 
- (void)drawView {
	const GLfloat triangleVertices[] = {
		0.0, 1.0, -6.0,	      // Triangle top centre
		-1.0, -1.0, -6.0,     // bottom left
		1.0, -1.0, -6.0       // bottom right
	};
	[EAGLContext setCurrentContext:context];
	glBindFramebufferOES(GL_FRAMEBUFFER_OES, viewFramebuffer);
	glViewport(0, 0, backingWidth, backingHeight);
	// -- BEGIN NEW CODE
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	glVertexPointer(3, GL_FLOAT, 0, triangleVertices);
	glEnableClientState(GL_VERTEX_ARRAY);
	glDrawArrays(GL_TRIANGLES, 0, 3);
	// -- END NEW CODE
	glBindRenderbufferOES(GL_RENDERBUFFER_OES, viewRenderbuffer);
	[context presentRenderbuffer:GL_RENDERBUFFER_OES];
	[self checkGLError:NO];
}
```



对，4 行代码就可以画出一个三角形来。让我们一行行地看这个代码，你就会发现实际上这还是很简单的。



``` 
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); 
```



这行是清除屏幕用的。我们传给 OpenGL 的控制比特位是让它用我们在 setupView 中设定的颜色（黑色）来清除屏幕以及深度缓冲。如果我们开着深度缓冲却不去清除它的话，我们的场景是显示不出来的。当然，如果我们没启用深度缓冲的话，也就没必要把 GL_DEPTH_BUFFER_BIT 传给 glClear() 了。

于是这样我们就把之前画在缓冲上的乱七八糟的东西给擦掉了（简称我擦）。记住，我们用的是双缓冲，一个缓冲用来画，另一个用来显示。



``` 
glVertexPointer(3, GL_FLOAT, 0, triangleVertices); 
```



这个函数告诉 OpenGL 我们的数据在哪，以及是什么格式的。传进去的四个参数分别是：

<ol>
<li>大小 - 这表示每个坐标有几个值。在我们这里就是 3 因为我们用的是 (X, Y, Z)。如果我们画的是 2D 的图且没有 Z 值的话就是 2。</li>
	<li>数据类型 - GL_FLOAT 表示我们传的是个浮点数。你要用整数也没问题，不过这 3D 的世界就是个浮点的世界啊所以浮啊浮的你就习惯了。</li>
	<li>步进 - 这个参数告诉 OpenGL 忽略两个坐标中间的若干字节。别想太多，现在先设成 0。这个只在我们从一个文件中载入顶点数据而数据中又有填充数据或者颜色数据时才有用。比如说像 Blender 那样的 3D 程序。</li>
	<li>指向数据的指针 - 显而易见，数据本身。</li>
</ol>缓冲也清了，数据及数据格式也已经告诉 OpenGL 了，我们来看下面这个非常重要的一步：



``` 
glEnableClientState(GL_VERTEX_ARRAY);
```



OpenGL 是一个“状态”机。这表示我们可以用“开启”或“禁用”命令来打开或是关闭功能。之前我们用到了 glEnable() 命令，这个影响的是 OpenGL 的“服务器端”状态。glEnableClientState() 影响的是我们自己的程序（客户端）。我们做的事情是告诉 OpenGL 我们的顶点数据是个数组，并让它开启绘制顶点的功能。当然咯，这个数组也可以是个颜色数组，这样我们就调用 glEnableClientState(GL_COLOR_ARRAY)；或者它还可以是个纹理坐标数组，这样我们就能做纹理映射了。嗯，一步步来，别一口吃成个胖子，我们得把基础内容讲完才会讲到纹理映射呢！

随着我们对 OpenGL 的使用越来越深入，我们会用到各种不同的客户端状态，上面说的也会在使用中变得更容易理解。

最后就是渲染三角形的命令了：



``` 
glDrawArrays(GL_TRIANGLES, 0, 3);
```



这就是召唤神龙的第七颗龙珠啊。OpenGL 用我们之前给它的信息在屏幕上画了个白色的三角形（白色是默认的画图颜色）。我们画出来的是一个有填充色的三角形，如果你要非填充的三角形的话还得换个方法来画。

我们看看这个函数的三个参数吧：

<ol>
<li>绘图方法 - 在这个例子中，我们传过去的 GL_TRIANGLES 参数很明显就是画三角形用的。不过到了画正方形的时候，第一个参数传什么就有讲究了。</li>
	<li>第一个顶点 - 我们的数组仅有三个点，所以我们要让 OpenGL 从第一个点开始画起，按照访问数组的惯例，我们传了个 0 进去。如果我们的顶点数组中有多个基础图形的数据的话，我们就会在这里放上偏移量。我们在后面的教程中会讲到这一块内容，现在的话，就放个 0 吧。</li>
	<li>顶点个数 - 告诉 OpenGL 我们的数组里有几个顶点是需要画的。在这个例子中就是 3 了。正方形是 4 ，线段是 2 (或者更多），点就是 1 或者更多（如果我们渲染多个点的话）。</li>
</ol>像上面这样在 drawView 方法里输好代码后，点下“Build and Go”，你的模拟器上应该和这幅图里看起来的一样：

<img src="http://codeleaks.files.wordpress.com/2010/08/0102.png" alt="0102.png" title="0102.png" border="0" width="386" height="742">

没骗你吧，屏幕当中有个白色三角形哟。

在我们讲其他的基础图形之前，你可以试试改变 Z 值。当把它改成 0.0 的时候你就知道我前面说的是什么意思了。屏幕上啥也没有了。

这么一点代码我们却唠叨了半天，但我希望你能从中了解 OpenGL ES 是怎么工作的。如果你曾经尝试过上手一个“标准”的 OpenGL 教程却无法顺利地进行下去的话，我希望你可以开始看到 OpenGL 和 OpenGL ES 之间有什么区别。

<h3>展望</h3>

下一节的教程中我们就要来画正方形了，真是让人期待啊。如果你照着上面敲进了代码却无法跑起来的话（这多么令人沮丧啊），这里是这次教程的源代码：

<a href="https://dl.dropbox.com/s/f4ouqlufblted6l/AppleCoder-OpenGLES-01.zip?dl">AppleCoder-OpenGLES-01.zip</a>
