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
title: "OpenGL ES 06 - 3D 中的物体"
type: post
---
<p>原作： Simon Maurice</p>

<p>到目前为止，我们在 2D 上已经花了不少时间，是该来看看 3D 的物体了。不过 3D 物体也不是特别复杂，它们只是比 2D 需要更多顶点信息（如果你用顶点数组来创建），或者如果你是用多个正方形拼成一个正方体的话就需要多一点变换。</p>

<p>其实我本来应该先把点和线讲掉的，不过，既然我们都已经能画出有颜色的三角形和有纹理的正方形了，也不急着关心那些无趣的形状了吧！</p>

<p>不过我确实需要回到变换那块讲讲旋转的一些细节。然后还有就是些我之前没提到的很基本的内容……好吧，其实这只是告诉你我要写很多很多教程！（也意味着我要翻很多很多教程！）</p>

<h3>开始之前，清空我们的 drawView 方法</h3>

<p>对之前的硬编码的内容挥挥手说再见吧，这次我们要把所有东西都砍掉，然后恢复到 drawView 方法最基本的状态。</p>

<p>我们的 drawView 应该是这样的：</p>

<pre>- (void)drawView {
	// Our new object definition code goes here
	[EAGLContext setCurrentContext:context];
	glBindFramebufferOES(GL_FRAMEBUFFER_OES, viewFramebuffer);
	glViewport(0, 0, backingWidth, backingHeight);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	glMatrixMode(GL_MODELVIEW);

	// Our new drawing code goes here
	glBindRenderbufferOES(GL_RENDERBUFFER_OES, viewRenderbuffer);
	[context presentRenderbuffer:GL_RENDERBUFFER_OES];
	[self checkGLError:NO];
}</pre>

<p>掌声可以响起来了，要不是之前我就把所有东西都按着 3D 的要求弄好了现在我们就应该在弄深度缓冲什么的，还要加一堆新代码。现在上面的代码你应该已经相当熟悉了。</p>

<h3>定义 3D 对象</h3>

<p>我们要弄一个正方体出来，因为这东西在 3D 图形中很常见，而且旋转起来的效果也很带感。当然在画正方体之前，我们得知道 3D 的正方体是由 6 个正方形组成的，而正方形则是我们一直用到现在的。这边的定义其实很简单，不过我来把它分解开来。首先看看正面：</p>

<pre>const GLfloat cubeVertices[] = {
	// Define the front face
	-1.0, 1.0, 1.0,		// top left
	-1.0, -1.0, 1.0,	// bottom left
	1.0, -1.0, 1.0,		// bottom right
	1.0, 1.0, 1.0,		// top right</pre>

<p>基本上就和原始的正方形定义一样，只不过我把这个面往前提了一个单位。在教程末尾我会解释这是为什么的。接下来是顶面：</p>

<pre>	// Top face
	-1.0, 1.0, -1.0,	// top left (at rear)
	-1.0, 1.0, 1.0,		// bottom left (at front)
	1.0, 1.0, 1.0,		// bottom right (at front)
	1.0, 1.0, -1.0,		// top right (at rear)</pre>

<p>注意，我在画顶面的时候不仅顺序和正面一样（从背面看过去都是逆时针方向），而且连起始点也是一样的。如果最后我们把整个正方体绕 X 轴转 90º 的话那么顶面就变成正面了。我们定义的第一个点是左上顶点，然后是左下顶点，依次类推。</p>

<p>接下来，背面：</p>

<pre>	// Rear face
	1.0, 1.0, -1.0,		// top right (when viewed from front)
	1.0, -1.0, -1.0,	// bottom right
	-1.0, -1.0, -1.0,	// bottom left
	-1.0, 1.0, -1.0,	// top left</pre>

<p>注意到顶点顺序以及起始点的一致性了吗？我们在剩下的几个面里都这么干：</p>

<pre>	// Bottom Face
	-1.0, -1.0, 1.0,	// Bottom left front
	1.0, -1.0, 1.0,		// right front
	1.0, -1.0, -1.0,	// right rear
	-1.0, -1.0, -1.0,	// left rear</pre>

<p>没错，还是一样的顺序和一样的起始点。想象下把这个面转到正面后顶点是怎么排列的。</p>

<p>最后就是左面和右面：</p>

<pre>	// Left face
	-1.0, 1.0, -1.0,	// top left
	-1.0, 1.0, 1.0,		// top right
	-1.0, -1.0, 1.0,	// bottom right
	-1.0, -1.0, -1.0,	// bottom left
	// Right face
	1.0, 1.0, 1.0,		// top left
	1.0, 1.0, -1.0,		// top right
	1.0, -1.0, -1.0,	// right
	1.0, -1.0, 1.0		// left
};</pre>

<p>这里是正方体的完整定义：</p>

<pre>const GLfloat cubeVertices[] = {
	// Define the front face
	-1.0, 1.0, 1.0,		// top left
	-1.0, -1.0, 1.0,	// bottom left
	1.0, -1.0, 1.0,		// bottom right
	1.0, 1.0, 1.0,		// top right
	// Top face
	-1.0, 1.0, -1.0,	// top left (at rear)
	-1.0, 1.0, 1.0,		// bottom left (at front)
	1.0, 1.0, 1.0,		// bottom right (at front)
	1.0, 1.0, -1.0,		// top right (at rear)
	// Rear face
	1.0, 1.0, -1.0,		// top right (when viewed from front)
	1.0, -1.0, -1.0,	// bottom right
	-1.0, -1.0, -1.0,	// bottom left
	-1.0, 1.0, -1.0,	// top left
	// Bottom Face
	-1.0, -1.0, 1.0,	// Bottom left front
	1.0, -1.0, 1.0,		// right front
	1.0, -1.0, -1.0,	// right rear
	-1.0, -1.0, -1.0,	// left rear
	// Left face
	-1.0, 1.0, -1.0,	// top left
	-1.0, 1.0, 1.0,		// top right
	-1.0, -1.0, 1.0,	// bottom right
	-1.0, -1.0, -1.0,	// bottom left
	// Right face
	1.0, 1.0, 1.0,		// top left
	1.0, 1.0, -1.0,		// top right
	1.0, -1.0, -1.0,	// right
	1.0, -1.0, 1.0		// left
};</pre>

<p>如果你对坐标系统还有问题的话，你得下点功夫想象一下它的形状了。如果想象不出来，那么拿张纸来，用斜 45º 的方式把物体画下来。你得知道一个 3D 的物体是什么样的。</p>

<p>把 cubeVertices 的定义放在新的物体定义这一行注释下面。</p>

<p>好的，现在我们需要把这东西画出来。</p>

<h3>绘制正方体</h3>

<p>最简单的办法就是用你们之前已经见过的代码来画正方体。后面我们会用一些进阶的方法（不过一旦看懂了就是简单到无法啊）来画 3D 对象。现在的话还是让我先来介绍下 3D 里的绘图。</p>

<p>我们以一段“你懂的”代码来开始。在新绘图代码部分加入下列行：</p>

<pre>glLoadIdentity();
glTranslatef(0.0, 0.0, -6.0);
glVertexPointer(3, GL_FLOAT, 0, cubeVertices);
glEnableClientState(GL_VERTEX_ARRAY);</pre>

<p>这边没啥新鲜的。我们把顶点状态复位，把正方体往后移好让我们能看见，告诉 OpenGL 我们的顶点数组和格式，然后启用 OpenGL 的状态来使用它。</p>

<p>接下来的代码也基本上和你之前用到的一样：</p>

<pre>// Draw the front face in Red
glColor4f(1.0, 0.0, 0.0, 1.0);
glDrawArrays(GL_TRIANGLE_FAN, 0, 4);</pre>

<p>也没有什么新鲜的玩意。我们把绘制颜色改成红色，然后让 OpenGL 画我们数组中的顶点 0 到顶点 4。画好之后我们来画顶面，也就是数组中的后 4 个顶点：</p>

<pre>// Draw the top face in green
glColor4f(0.0, 1.0, 0.0, 1.0);
glDrawArrays(GL_TRIANGLE_FAN, 4, 4);</pre>

<p>看一下 glDrawArrays()，如果你记得我之前说过的，你应该想得起来，第二个参数表示起始的偏移量。因为我们画的是正方体中的第二个面，所以我们要告诉 OpenGL 先加上一个 4 的偏移量（就是 cubeVertices[4]，前面 4 个顶点是正面），然后再画四个顶点。</p>

<p>现在让我们来画背面：</p>

<pre>// Draw the rear face in Blue
glColor4f(0.0, 0.0, 1.0, 1.0);
glDrawArrays(GL_TRIANGLE_FAN, 8, 4);
</pre>

<p>还是一样的，这次我们从 cubeVertices[8] 开始。对于后面的 3 个面也是同样的方式：</p>

<pre>// Draw the bottom face
glColor4f(1.0, 1.0, 0.0, 1.0);
glDrawArrays(GL_TRIANGLE_FAN, 12, 4);
// Draw the left face
glColor4f(0.0, 1.0, 1.0, 1.0);
glDrawArrays(GL_TRIANGLE_FAN, 16, 4);
// Draw the right face
glColor4f(1.0, 0.0, 1.0, 1.0);
glDrawArrays(GL_TRIANGLE_FAN, 20, 4);</pre>

<p>我们所做的无外乎就是改变颜色，然后改变起始的偏移量。 </p>

<p>现在，点一下 “Build and Go”，你可以看到一个静止的红色正方形。如果要看到 6 个面的话，我们需要让正方体绕着所有的三个轴转起来。</p>

<p>在 glLoadIdentity() 之前，加入如下赋值：</p>

<pre>rota += 0.5;</pre>

<p>我们的老朋友 rota 童鞋回来了。现在再来看看另一位老朋友，glRotatef()。在 glTranslatef() 后面，加入下面这行：</p>

<pre>glRotatef(rota, 1.0, 1.0, 1.0);</pre>

<p>之前我们只让 glRotatef() 绕着一根轴转。现在我们让它沿着三根轴一起转。</p>

<p>点下“Build and Go”，就是这个样没跑了：</p>

<p><img src="http://codeleaks.files.wordpress.com/2010/08/0601.jpg" alt="0601.jpg" title="0601.jpg" border="0" width="376" height="230" /></p>

<p>代表 3D 物体对你表示惨无人道的欢迎。</p>

<h3>怎么把纹理映射上去呢？</h3>

<p>那啥，我们应该不满足于纯色的物体了对吧？让我们来用上节教程中的纹理，将它映射到物体的六个面上，这样子才好玩嘛。</p>

<p>嗯，我们还保留了上节教程中的纹理和载入代码，所以我们要做的就是改变 drawView 方法。现在你就会意识到，只要前期工作做好了，纹理映射是多简单的一件事啊！</p>

<p>首先，这是上节教程里的：</p>

<pre>const GLshort squareTextureCoords[] = {
	// Front face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
};</pre>

<p>只有一个面的时候上面的代码就够了，不过现在我们需要扩展一下。当然了，这也很简单。或许你觉得刚才定义正方体的时候我们保持每个面的起点和顺序都一致很死板，不过现在你就知道为什么了。</p>

<p>当 OpenGL 把纹理渲染到正方体的面上的时候，由于我们跳过了一个偏移量（glDrawArrays() 里的 4、8、12 等）然后开始每个面的定义，纹理映射也会跳到相应的偏移处，找出每个面的纹理坐标。所以，要映射 6 个面，我们只要把上面的 4 个坐标再复制 5 遍就行了：</p>

<pre>const GLshort squareTextureCoords[] = {
	// Front face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
	// Top face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
	// Rear face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
	// Bottom face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
	// Left face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
	// Right face
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1,		// top right
};</pre>

<p>连郭敬明老师都表示这毫无压力！</p>

<p>现在我们需要的就是几行绘图代码，然后就可以画出我们的纹理映射过的正方形了：</p>

<p>在画第一个面之前，加入下列代码：</p>

<pre>glTexCoordPointer(2, GL_SHORT, 0, squareTextureCoords);
glEnableClientState(GL_TEXTURE_COORD_ARRAY);</pre>

<p>就是这么简单。别删掉每段绘制代码中的 glColour4f()，点一下 “Build and Go” 就可以看到了：</p>

<p><img src="http://codeleaks.files.wordpress.com/2010/08/0602.jpg" alt="0602.jpg" title="0602.jpg" border="0" width="365" height="223" /></p>

<p>怎么样，会动的，有纹理映射的 3D 物体！</p>

<h3>关于纹理映射图像</h3>

<p>上节教程中忘了说了，我希望你们尝试使用你们自己的纹理图片来做映射。不过很重要的一点是，纹理图片的大小应该是 2 的 N 次方。也就是说宽度和高度必须是 1、2、4……32……512、1024……宽度和高度不一定要相等，不过它们都必须是 2 的次幂。所以 32 x 512 和 64 x 64 都是合法的大小，而 30 x 30 就不是。</p>

<h3>关于 glRotatef() 及你物体的顶点</h3>

<p>再次，我希望你们去创建自己的物体。不知道你是否注意到我总是把三角形、正方形的中心点放在 0,0,0 的位置，这样物体的端点正好被 0,0,0 等分？因为在旋转的时候，OpenGL 会沿着物体的中心点来旋转，这个中心点就是我们模型矩阵中的 0,0,0。OpenGL 不会把你的模型先调整到 0,0,0 后再旋转。如果你的物体中心点不在 0,0,0，那么它转起来的时候就会“不平衡”。</p>

<h3>今天就说这么多</h3>

<p>今天的教程就要结束了。希望你觉得这个对你有所帮助，并且和我一样能感受到它的乐趣。</p>

<p>说句事后诸葛亮的话，3D 物体的描述就应该按逆时针顺序来。现在先别想那么多，下次教程中我会讨论下逆时针和顺时针之间的细节。</p>

<p>这里是这节教程的代码：</p>

<p><a href="https://dl.dropbox.com/s/dmnyb295ufqciql/AppleCoder-OpenGLES-06.zip?dl">AppleCoder-OpenGLES-06.zip</a></p>

<p>我们下次再见。</p>
