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
title: "OpenGL ES 02 - 绘制基础图形2 - 正方形"
type: post
---
原作： Simon Maurice

严格地说起来，正方形可不是 OpenGL ES 中的基础图形。不过，画这东西也不比画三角形麻烦啊。在这一篇教程中，我们将利用上一篇里的代码，把三角形变成一个正方形。同样的我们是静态地渲染出图形，但是很快我们会涉及到形变的内容哦（移来移去的）。而且只要我们做好了正方形，我们就能做正方体了，然后就是有纹理映射的正方体……呃又说远了。

<h3>前情回顾及本篇提要</h3>

上一讲，我们在“空白的画布”上画出了一个实心三角形。我们创建了一个顶点数组，用 glVertextPointer() 告诉 OpenGL 数据和格式，让 OpenGL 进入渲染顶点数组的状态，然后就用 glDrawArrays() 渲染出三角形。

今天我们要接着用这个代码来画个正方形。其实我们也仅需更换几行代码而已。首先，也是最明显的，我们要定义 4 个点而不是画三角形时的 3 个。然后我们传另一个绘图模式的参数给 glDrawArrays()，这样 OpenGL 就能画出不一样的东西了。

说改咱就改呀，风风火火一声吼。

<h3>定义正方形的顶点</h3>

打开上节教程的 Xcode 项目，转到 drawView 方法。注释掉 triangleVertices[] 常量——不过别删了，我们在后面讲形变的时候还用得上——然后加入下列代码：



``` 
const GLfloat squareVertices[] = {
	-1.0, 1.0, -6.0,            // Top left
	-1.0, -1.0, -6.0,           // Bottom left
	1.0, -1.0, -6.0,            // Bottom right
	1.0, 1.0, -6.0              // Top right
};

```



这就是正方形了。看到我们是以逆时针顺序定义的没？

然后往下看到函数中间，注释掉画三角形的代码，同样的后面我们还会再用到这段代码的所以别删掉。注释掉 glVertexArray()、 glEnableClientState() 及 glDrawArrays() 这三个方法调用然后加入下列代码：



``` 
glVertexPointer(3, GL_FLOAT, 0, squareVertices);
glEnableClientState(GL_VERTEX_ARRAY);
glDrawArrays(GL_TRIANGLE_FAN, 0, 4);
```



还是那三个函数，只不过有些细微的区别。



``` 
glVertexPointer(3, GL_FLOAT, 0, squareVertices);
```



这边的唯一区别是告诉 OpenGL 使用另外一组顶点数据，我们现在要画正方形而不是三角形了。

glEnableClientState() 还是一样的，我们告诉 OpenGL 我们要用顶点数组来绘图了（而不是颜色数组或者别的什么奇怪的东西）。



``` 
glDrawArrays(GL_TRIANGLE_FAN, 0, 4);
```



最大的改动就是这边。在上个教程中，我们第一个参数传的是 GL_TRIANGLES ，第三个参数传的是 3 。还记得第二个参数表示的是数组中的偏移量吗？在这里仍然是 0 ，因为我们的顶点数组只包含了正方形的顶点信息。

第一个参数是绘图模式。我们已经见过两个了。我想花点时间讲一下绘图模式之间的区别。OpenGL 中的绘图模式有：

<ul>
<li>GL_POINTS</li>
	<li>GL_LINES</li>
	<li>GL_LINE_LOOP</li>
	<li>GL_LINE_STRIP</li>
	<li>GL_TRIANGLES</li>
	<li>GL_TRIANGLE_STRIP</li>
	<li>GL_TRIANGLE_FAN</li>
</ul>由于我们还没讲到点和线，所以现在我们主要看下最后的三个。在我开始之前，我想说的是在一个顶点数组中可以有不止一个三角形，所以虽然目前我们在一个数组中仅描述一个对象，但实际上用起来并没有这种限制。

GL_TRIANGLES - 传这个参数表示 OpenGL 以 3 个顶点一组来遍历我们的数组。所以，数组中的前三个顶点将组成一个三角形，然后是接下去的三个，直到数组末尾。

GL_TRIANGLE_STRIP - OpenGL 先是使用头两个顶点，然后对于后继顶点，它都将和之前的两个顶点一起组成三角形。也就是，对于 squareVertices[6~8] 来说，它将和 squareVertices[0~2] 和 squareVertices[3~5] 组成三角形。对于 squareVertices[9~11] ，它将同 squareVertices[3~5] 及 squareVertices[6~8] 合体。以此类推。

注意，上边写的 squareVertices[0~2] 实际上指的是：
<ul>
<li>squareVertices[0] - X 坐标 </li>
<li>squareVertices[1] - Y 坐标 </li>
<li>squareVertices[2] - Z 坐标 </li>
</ul>如果这样的解释不给力的话，我会在后面用例子来说明。

GL_TRIANGLE_FAN - 在头两个顶点之后，每个后继顶点将与前一个以及第一个顶点一起组成一个三角形。也就是说，对 squareVertices[6~8] 来说，它将与 squareVertices[3~5] （前一个）以及 squareVertices[0~2] （第一个）合体。

由于我们用的是 GL_TRIANGLES_FAN ，我们可以在屏幕上看到一个正方形。点击“Build & Go”，你成功了：

<img src="http://codeleaks.files.wordpress.com/2010/08/0201.png" alt="0201.png" title="0201.png" border="0" width="386" height="742">

现在回过头来看看我们的顶点数组。想象一下正方形是怎么样由三角形拼出来的。OpenGL 是这样渲染的： 
<ul>
<li>三角形顶点 1: squareVertices[0~2] —— 正方形左上角 </li>
<li>三角形顶点 2: squareVertices[3~5] —— 正方形左下角 </li>
<li>三角形顶点 3: squareVertices[6~8] —— 正方形右下角 </li>
</ul>OpenGL 用上面的 3 个点画一个三角形，这组成了正方形的左下半边。想象下这个正方形由一条从左上角到右下角的对角线分开，这样不就变成两个三角形了吗？OpenGL 先画了左下半边这个三角形。

注意，上边写的 squareVertices[0~2] 实际上指的是：

<ul>
<li>squareVertices[0] - X 坐标 </li>
<li>squareVertices[1] - Y 坐标 </li>
<li>squareVertices[2] - Z 坐标 </li>
</ul>然后是：

<ul>
<li>三角形顶点 1: squareVertices[9~11] —— 正方形右下角 </li>
<li>三角形顶点 2: squareVertices[6~8] —— 前一个顶点，左下角 </li>
<li>三角形顶点 3: squareVertices[0~2] —— 第一个顶点，左上角 </li>
</ul>只往里面加入了一个新的点，OpenGL 就能画出完整的正方形来了。

<h3>GL_TRIANGLE_STRIP</h3>

回到代码里，把 glDrawArrays() 的第一个参数从 GL_TRIANGLE_FAN 改成 GL_TRIANGLE_STRIP：



``` 
glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
```



点击“Build & Go”然后你就会看到如下的图案：

<img src="http://codeleaks.files.wordpress.com/2010/08/0202.jpg" alt="0202.jpg" title="0202.jpg" border="0" width="372" height="175">

我们来看一下为什么改了一个绘图模式就没有画成正方形了。OpenGL 是以下面这种方法来处理我们的顶点数据的。

<ul>
<li>三角形顶点 1: squareVertices[0~2] —— 左上 </li>
<li>三角形顶点 2: squareVertices[3~5] —— 左下 </li>
<li>三角形顶点 3: squareVertices[6~8] —— 右下 </li>
</ul>OpenGL 用前三个点画了一个三角形，也就是正方形的左下半边，这与前一个例子一样。

<ul>
<li>三角形顶点 1: squareVertices[9~11] —— 右上 </li>
<li>三角形顶点 2: squareVertices[6~8] —— 前一个顶点，右下 </li>
<li>三角形顶点 3: squareVertices[6~8] —— 再前一个顶点，左下 </li>
</ul>OpenGL 用这三个顶点画了一个三角形出来。在我们的例子中，这和我们想要拼成正方形的三角形比起来正好旋转了一个90º 。

如果我们稍微改下我们的顶点数组，我们还是可以用 GL_TRIANGLE_STRIP 来画出一个正确的正方形的。记住我们的绘图方法和顶点数组需要相匹配，否则就像我们从 FAN 改成 STRIP 那样会有奇怪的结果哦。

我们像下面这样改下顶点数组，记住我们现在用的是 GL_TRIANGLE_STRIP 绘图模式：



``` 
const GLfloat stripSquare[] = {
	-1.0, -1.0, -6.0,               // bottom left
	1.0, -1.0, -6.0,                // bottom right
	-1.0, 1.0, -6.0,                // top left
	1.0, 1.0, -6.0                  // top right
 };

```



换好代码之后，前三个点组成第一个三角形，如下所示：

<img src="http://codeleaks.files.wordpress.com/2010/08/0203.png" alt="0203.png" title="0203.png" border="0" width="400" height="400">

现在通过指定右上的顶点后（P4），它将同左上顶点（P3）以及再前面一个右下顶点（P2）一同组成三角形。新的顶点以橙色、绿色和红色表示（原文中确实写的是顶点，可 Simon 大神您画的这是边吧！）：

<img src="http://codeleaks.files.wordpress.com/2010/08/02041.png" alt="0204.png" title="0204.png" border="0" width="400" height="400">

我们的最终产物同样是个正方形。所以，没什么不同的，只要记住，绘图模式和提供的顶点信息之间相互匹配就可以了。

<h3>最后…</h3>

三角形和正方形你都已经见过了。还剩下点和线没讲。这两个都很简单，我们会在后面提到。然后下一次我们也会给现在画出来的东西填上颜色。

一旦填上了颜色，我们就要让它们动起来了，然后就把纹理什么的贴在它们身上，嗯哼。当然这不会是 Doom 3 ，不过你也能开始创建 3D 的对象了啊。到那时候我会开始讲 3D 世界的。

这里是源代码：

<a href="https://dl.dropbox.com/s/fmxjn4kqpn6o2s2/AppleCoder-OpenGLES-02.zip?dl">AppleCoder-OpenGLES-02.zip</a> 

那么，下次见咯。
