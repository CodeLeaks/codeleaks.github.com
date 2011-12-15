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
title: "OpenGL ES 05 - 纹理映射"
type: post
---
<p>原作： Simon Maurice</p>

<p>我决定把纹理映射的内容提到前面来讲，因为相比于面体（3D 物体）来说，纹理映射只涉及到单个面，理论上要简单点。同时这貌似也是很多 iPhone OpenGL ES 程序员容易卡壳的地方。所以我决定在这节教程中来说说纹理映射那点事儿。</p>

<p>之前我跳过了很多 OpenGL 的细节部分，这是为了让你们能尽早在屏幕上显示物体然后有个直观感受，而不是看些什么 OpenGL 的历史、OpenGL ES 和 OpenGL 对比之类的东西。不过今天我要讲讲之前我跳掉的这些技术细节。同时这也就意味着这会是篇很长的教程。</p>

<p>我先说在前面，我们的代码主要需要做的是把纹理载入到我们的程序中，然后传给 OpenGL 引擎以让 OpenGL 能使用它。这点不复杂，只不过需要和 iPhone SDK 打些交道。</p>

<h3>为纹理做准备</h3>

<p>在我们使用纹理之前，我们需要把它载入到应用程序中来，转成 OpenGL 需要的格式，然后告诉 OpenGl 在哪里找到它。做好了这些之后，剩下的就和上节教程中的给正方形上色一样简单。</p>

<p>启动 Xcode 并打开 EAGLView.h。首先我们需要加入一个 OpenGL 所需要的变量，如下所示：</p>

<pre>GLuint textures[1];</pre>

<p>很明显，这是一个长度为 1 的 GLuint 数组。之前你已经见过 GLfloat 了，而 GLuint 则是 OpenGL 中的另一个 typedef，表示无符号整数。我们最好保持用 GLxxxx 这样的 typedef，而不是用 Objective C 中的类型，因为 OpenGL 中的 typedef 是为了 OpenGL 实现而非开发环境而定义的。</p>

<p>接着我们用 OpenGL 的函数 glGenTextures() 来生成这个变量。现在先记得我们定义过这个变量，后面我们会讲到 glGenTexture() 和这个变量的。</p>

<p>在方法声明那里，加入如下方法声明：</p>

<pre>- (void)loadTexture;</pre>

<p>我们将把代码放在这个方法中来载入纹理。</p>

<h3>往项目中添加 CoreGraphics 框架</h3>

<p>为了载入然后处理纹理，我们需要往项目中添加一个 CoreGraphics 框架。它提供了所有我们需要的方法，因此我们不用像在 Windows OpenGL 教程里看到的那样自己去实现底层的代码。</p>

<p>在 Xcode 的 “Groups &amp; Files” 的 “Frameworks” 处右击，然后选择 Add -&gt; Existing Frameworks。在列表中选中 CoreGraphics.framework 后点击 “Add” 加入项目中。</p>

<p>接下来我们需要添加一张纹理图片到我们的项目中。下载 checkerplate.png 然后保存到项目的目录里。右击 “Resources” 处，选择 Add -&gt; Exisiting Files。选择我们的图片，然后它应该就会出现在我们的资源组中了。</p>

<h3>把纹理载入到我们的应用程序及 OpenGL 中</h3>

<p>切到 EAGLView.m，我们来实现 loadTexture 方法。</p>

<pre>- (void)loadTexture {
}</pre>

<p>下面的代码在这个方法中都是顺序出现的，所以我们一行一行往里面填就行了。第一件事情是用下列代码把图片载入到我们的应用程序中来：</p>

<pre>CGImageRef textureImage = [UIImage imageNamed:@"checkerplate.png"].CGImage;
if (textureImage == nil) {
	NSLog(@"Failed to load texture image");
	return;
}</pre>

<p>CGImageRef 是 CoreGraphics 里的一种数据类型，它包含了所有关于图片的信息。要得到信息首先我们得用 UIImage 的类方法 imageName: 来创建一个自动释放的 UIImage 实例，UIImage 按文件名来在我们应用程序的 bundle 中找到这个文件。然后 UIImage 自动创建了 CGImageRef，我们用 CGImage 这个属性就可以访问到。</p>

<p>现在我们需要取一下图片的大小，这个在后面会用上。</p>

<pre>NSInteger texWidth = CGImageGetWidth(textureImage);
NSInteger texHeight = CGImageGetHeight(textureImage);</pre>

<p>CGImageRef 数据中包含了图像的宽度和高度，不过这两个属性不能直接访问。我们需要调用上面这两个函数。</p>

<p>正如 CGImageRef 名字所示，它并没有包含图像数据，而是一个指向图像数据的引用。所以我们需要分配出一块内存空间来保存我们的真实图像数据：</p>

<pre>GLubyte *textureData = (GLubyte *)malloc(texWidth * texHeight * 4);</pre>

<p>数据的大小应该是宽乘上高再乘上 4。上节我们讲到了 OpenGL 只认 RGBA 的颜色值，所以每个像素都要用 4 个字节来存储，每个字节对应 RGBA 中的一个值。</p>

<p>接下来的几个函数调用看起来很壮观：</p>

<pre>CGContextRef textureContext = CGBitmapContextCreate(
	textureData,
	texWidth,
	texHeight,
	8, texWidth * 4,
	CGImageGetColorSpace(textureImage),
	kCGImageAlphaPremultipliedLast);
CGContextDrawImage(textureContext,
	CGRectMake(0.0, 0.0, (float)texWidth, (float)texHeight),
	textureImage);
CGContextRelease(textureContext);</pre>

<p>第一个函数，正如它的名字所表示的，是一个 CoreGraphics 的函数，用于返回一个 Quartz2D 的图形上下文引用。我们所做的就是告诉它纹理的数据在哪，以及纹理的格式和大小分别是什么。</p>

<p>然后通过我们刚刚创建的这个图形上下文引用，我们把图像画到之前分配出来的内存空间里（textureData 指针所指向的内存）。这个上下文引用中包含所有必要的信息，让它能把数据以 OpenGL 需要的格式复制到我们用 malloc() 创建出来的空间中。</p>

<p>然后和 CoreGraphics 相关的操作就告一段落了，所以我们释放掉我们创建的 textureContext 句柄。</p>

<p>上面的这部分我们只是粗粗地过了一遍，因为我们对 OpenGL 这块更感兴趣些。来处理任何添加到你项目中的 PNG 格式图形纹理时，你都可以重用上面的代码。</p>

<p>现在，欢迎回到 OpenGL 世界。</p>

<p>还记得我们在头文件中声明的变量吗？我们准备用到它了。看看下一行代码：</p>

<pre>glGenTextures(1, &amp;textures[0]);</pre>

<p>我们要把纹理数据从我们的应用程序复制到 OpenGL 引擎那边，所以我们得让 OpenGL 分配出一块内存（我们不能直接分配）。还记得 texture[] 是定义成 GLuinit 的吗？当我们调用 glGenTextures 时，OpenGL 会创建一个“句柄”或者说“指针”。对于每个载入 OpenGL 的纹理都产生一个唯一的引用。OpenGL 返回什么值我们并不关心，只不过每次当我们要用 checkerplate.png 的时候我们间接地用到 textures[0]。我们知道我们实际上想要用的是什么，OpenGL 也知道。</p>

<p>如果需要一次性载入多个纹理，我们就多分配点空间。比方说我们的应用程序中需要 10 个纹理，我们就这么写：</p>

<pre>GLuint textures[10];
glGenTextures(10, &amp;textures[0]);</pre>

<p>在这个例子中，我们只需要一个纹理，所以我们也就只分配了一个。</p>

<p>接下来我们需要启用我们刚刚生成的纹理：</p>

<pre>glBindTexture(GL_TEXTURE_2D, textures[0]);</pre>

<p>第二个参数很明显，就是我们刚刚创建的纹理。第一个参数总是 GL_TEXTURE_2D，因为目前所有的 OpenGL ES 都只接受这个参数。“完整的” OpenGL允许 1D 或者是 3D 的纹理，但是对 OpenGL ES 来说，这个还没提上议事日程。</p>

<p>所以，记得这个是用来启用纹理的就行了。</p>

<p>接着我们把我们的纹理数据（textureData 所指的）传给 OpenGL。OpenGL 在它那边（服务器端）管理纹理数据，所以数据先要根据硬件实现转成相应格式，然后再复制到 OpenGL 的空间里。这个函数各种参数传递，不过由于 OpenGL ES 的限制，大多数参数总是不变的：</p>

<pre>glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, texWidth, texHeight, 0, GL_RGBA, GL_UNSIGNED_BYTE, textureData);</pre>

<p>我们一个个地来看这些参数：</p>
<ol>
	<li>目标 - 简而言之，这边总是 GL_TEXTURE_2D</li>
	<li>级别 - 指定纹理细化的级别，0 表示所有图片允许的细节，高一点的数字表示进入第 n 级的多纹理图像简化。</li>
	<li>内部格式 - 内部格式和下面列出的格式必须一致，所以都是 GL_RGBA。</li>
	<li>宽度 - 图片宽度。</li>
	<li>调试 - 图片高度。</li>
	<li>边框 - 总是设为 0，因为 OpenGL ES 不支持纹理边框。</li>
	<li>格式 - 必须与内部格式一致。</li>
	<li>类型 - 每个象素的类型。还记得每个象素是四个字节吗？所以每个象素都是一个无符号字节（时刻记得 RGBA 啊）。</li>
	<li>象素 - 指向真实图像数据的指针。</li>
</ol>

<p>所以虽然这边参数有点多，不过要么一直都不变，要么就是你之前已经输入过的变量（textureData，texWidth 和 texHeight）。我们要做的就是把纹理数据交给 OpenGL 去处理。</p>

<p>传好数据之后，我们就可以释放 textureData 了：</p>

<pre>free(textureData);</pre>

<p>现在只剩下三个函数要调用：</p>

<pre>glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glEnable(GL_TEXTURE_2D);</pre>

<p>这三个调用做了这些最后的设置，然后让 OpenGL 进入了纹理映射的“状态”。</p>

<p>头两个调用告诉 OpenGL 在缩放的时候如何处理纹理，放大时为 GLTEXTURE_MAG_FILTER，缩小时为 GL_TEXTURE_MIN_FILTER。要让纹理映射能正常工作，你至少需要指定这两者之一。然后两种情况下都设置了 GL_LINEAR 选项。</p>

<p>然后，我们只需要调用一下 glEnable()，以便后面我们在绘制代码中能顺利地让 OpenGL 使用纹理。</p>

<p>最后，我们在 initWithCoder 函数中加入这个方法的调用。</p>

<pre>[self setupView];
[self loadTexture];	// Add this line
</pre>

<p>在 setupView 方法的后面加上一行就行了。</p>

<h3>drawView 的调整</h3>

<p>脏活累活已经做完了，接下来把 drawView 方法的改变不会比前节给正方形上色难。首先，注释掉 squareColours[] 数组，因为这边我们用不上。</p>

<p>现在，回忆下我们是怎么给正方形上色的。对于每个顶点我们都提供了一个颜色值。在纹理映射中，我们也要告诉 OpenGL物体中的每一个顶点应该对应到纹理中的哪一个坐标。</p>

<p>在我们做这事情之前，我们需要知道纹理的坐标。纹理中的 (0, 0) 在左下角，沿着每条边数值分别从 0 过渡到 1。看看我们纹理的这张图：</p>

<p><img src="http://codeleaks.files.wordpress.com/2010/08/0501.png" alt="0501.png" title="0501.png" border="0" width="350" height="380" /></p>

<p>回过头来看一下我们的 squareVertices[] ：</p>

<pre>const GLfloat squareVertices[] = {
	-1.0, 1.0, 0.0,		// Top left
	-1.0, -1.0, 0.0,	// Bottom left
	1.0, -1.0, 0.0,		// Bottom right
	1.0, 1.0, 0.0		// Top right
};</pre>

<p>看到没？我们需要指定的第一个纹理坐标是左上角，也就是 (0, 1)。第二个顶点是正方形的左下角，也就是纹理坐标 (0, 0)。右下角是纹理坐标 (1, 0)，最后是右上角 (1, 1)。于是我们的 squareTextureCoods[] 应该这么写：</p>

<pre>const GLshort squareTextureCoords[] = {
	0, 1,		// top left
	0, 0,		// bottom left
	1, 0,		// bottom right
	1, 1 		// top right
};</pre>

<p>注意这边用的是 GLShort 而不是 GLfloat。把上面的代码加到你的项目中。</p>

<p>和我们在颜色数组中做的事情很像吧？</p>

<p>好的，现在我们需要改下绘图代码。三角形部分不管它，从画正方形之前的 glLoadIdentity() 开始。现在的绘图代码是这样的：</p>

<pre>glLoadIdentity();
glColor4f(1.0, 1.0, 1.0, 1.0);      // NEW
glTranslatef(1.5, 0.0, -6.0);
glRotatef(rota, 0.0, 0.0, 1.0);
glVertexPointer(3, GL_FLOAT, 0, squareVertices);
glEnableClientState(GL_VERTEX_ARRAY);
glTexCoordPointer(2, GL_SHORT, 0, squareTextureCoords);     // NEW
glEnableClientState(GL_TEXTURE_COORD_ARRAY);                // NEW
glDrawArrays(GL_TRIANGLE_FAN, 0, 4);
glDisableClientState(GL_TEXTURE_COORD_ARRAY);               // NEW</pre>

<p>嗯，我们有四行新的代码，然后我删了上一节教程里上色的代码。第一行调用了 glColor4f()，它的作用我们放到后面慢慢说。</p>

<p>后面的三行新代码看起来应该是相当熟悉了吧。这次我们要用的不是物体的顶点，也不是颜色，我们要用的是纹理。</p>

<pre>glTexCoordPointer(2, GL_SHORT, 0, squareTextureCoords);     // NEW
glEnableClientState(GL_TEXTURE_COORD_ARRAY);                // NEW</pre>

<p>第一个调用告诉 OpenGL 我们的纹理坐标数组在哪里以及是什么格式的。和之前的区别是每个顶点有两个值（废话，这是个 2D 的纹理），然后因为我们用的 GLshort 数据类型所以这里对应地换成了 GL_SHORT。这个函数调用中没有用到步进（步进为 0），然后最后一个参数是指向我们坐标的指针。</p>

<p>然后我们让 OpenGL 以刚才我们指定的坐标数组来启用纹理映射的客户端状态。</p>

<p>glDrawArrays() 还是老样子，紧跟着的是这行：</p>

<pre>glDisableClientState(GL_TEXTURE_COORD_ARRAY);               // NEW</pre>

<p>记得上节里说过的吗？为了避免三角形受到影响，我们关掉了颜色数组。同样的，这里我们也把纹理映射关掉，否则 OpenGL 也会将纹理映射作用于三角形身上。</p>

<p>点击“Build and Go”，见证奇迹的时刻：</p>

<p><img src="http://codeleaks.files.wordpress.com/2010/08/05021.jpg" alt="0502.jpg" title="0502.jpg" border="0" width="368" height="209" /></p>

<p>我们这个网纹钢板的纹理就映射到正方形上了，而三角形还是没变。</p>

<h3>进一步的实验</h3>

<p>首先，看一下我们新加进去的 glColor4f()：</p>

<pre>glColor4f(1.0, 1.0, 1.0, 1.0);      // NEW</pre>

<p>这个是把绘画颜色改成了不透明的白色。不过为什么要这么做？当然咯，OpenGL 是个“状态”机，所以除非我们设了什么值，否则它将保持同一状态。（作者你这辈子就指这句话活了吧啊啊啊？）所以在我们把颜色改成白色之前它一直是蓝色。</p>

<p>嗯，这是因为现在我们做纹理映射的时候，OpenGL 在我们当前的颜色（蓝色）和当前纹理象素之间做一个乘积，比方说：</p>

<pre>                       R    G     B    A
指定的颜色：          0.0, 0.0, 0.8, 1.0
纹理的象素颜色：      1.0, 1.0, 1.0, 1.0</pre>

<p>当 OpenGL 画这个象素的时候，它会做这样的乘法：</p>

<pre>颜色中的红色   *  象素中的红色        = 渲染颜色
    0.0        *     1.0              = 0.0
颜色中的绿色   *  象素中的绿色
    0.0        *     0.0              = 0.0
颜色中的蓝色   *  象素中的蓝色
    0.8        *     1.0              = 0.8</pre>

<p>如果我们注释掉绘制代码前面的 glColor4f()，那么你应该会看到：</p>

<p><img src="http://codeleaks.files.wordpress.com/2010/08/05031.jpg" alt="0503.jpg" title="0503.jpg" border="0" width="372" height="194" /></p>

<p>当我们设成白色的时候，我们的乘法是这样的：</p>

<p>指定的颜色： 1.0, 1.0, 1.0, 1.0 乘上</p>
<p>象素的颜色： 0.8, 0.8, 0.8, 1.0</p>

<p>所以我们需要把颜色改成白色。</p>

<h3>是的，就是这样了</h3>

<p>这次的教程就这样了（幸好啊，否则我要翻不动了）。我知道我写了很多，不过正像我开头说的那样，实际上纹理映射的代码也不是很多，主要还是加载和设置纹理方面的东西。下次我们来说说 3D 对象的纹理映射、混色以及其他好玩的东西。</p>

<p>老规矩，这里是代码。</p>

<p><a href="https://dl.dropbox.com/s/uz97ngiqksicg5j/AppleCoder-OpenGLES-05.zip?dl">AppleCoder-OpenGLES-05.zip</a></p>

<p>那么，下次再见咯。</p>
