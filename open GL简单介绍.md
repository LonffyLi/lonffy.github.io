* 原文链接：http://www.cnblogs.com/lizhengjin/archive/2010/12/23/1914795.html


介绍
Frame Buffer Object(FBO)扩展，被推荐用于把数据渲染到纹理对像。相对于其它同类技术，如数据拷贝或交换缓冲区等，使用FBO技术会更高效并且更容易实现。
在这篇文章中，我将会快速地讲解一下如何来使用这一扩展，同时会介绍一些在使用过程中我们要注意的地方。学会该技术后，你便可以把一些渲染到纹理(render to texture)的功能加入到你的程序中，实现更快速的运行。

建立
和OpenGL中的其它对像一样，如纹理对像(texture object), 像素缓冲对像(pixel buffer objects) , 顶点缓冲对像(vertex buffer object)等，在使用一个FBO对像之前，你必须先要生成该对像，并取得一个有效的对像标识。

GLuint fbo; glGenFramebuffersEXT(1, &fbo);
要对一个FBO进行任何的操作，你必须先要对它进行绑定。这一步骤与我们平时使用VBO或者纹理的过程很像。绑定对像后，我们便可以对FBO进行各种操作了，以下代码演示如何进行绑定。

glBindFramebufferEXT(GL_FRAMEBUFFER_EXT, fbo);
第一个参数是“目标(target)”，指的是你要把FBO与哪个帧缓冲区进行绑定，目前来说，我个参数就只有一些预定义的选择(GL_FRAMEBUFFER_EXT)，但将来扩展的发展，可能会来现其它的选择，让你把FBO与其它的目标进行绑定。整型变量fbo，是用来保存FBO对像标识的，这个标识我们已在前面生成了。要实现任何与FBO有关的操作，我们必须有一个FBO被绑定，否则调用就会出错



加入一个深度缓存(Depth Buffer)
一个FBO它本身其实没有多大用处，要想让它能被更有效的利用，我们需要把它与一些可被渲染的缓冲区绑定在一起，这样的缓冲区可以是纹理，也可以是下面我们将要介绍的渲染缓冲区(renderbuffers)。

一个渲染缓冲区，其实就是一个用来支持离屏渲染的缓冲区。通常是帧缓冲区的一部份，一般不具有纹理格式。常见的模版缓冲和深度缓冲就是这样一类对像。

在这里，我们要为我们的FBO指定一个渲染缓冲区。这样，当我们渲染的时候，我们便把这个渲染缓冲区作为FBO的一个深度缓存来使用。

和FBO的生成一样，我们首先也要为渲染缓冲区指定一个有效的标识。

GLuint depthbuffer; glGenRenderbuffersEXT(1, &depthbuffer);
成功完成上面一步之后，我们就要对该缓冲区进行绑定，让它成为当前渲染缓冲，下面是实现代码。

glBindRenderbufferEXT(GL_RENDERBUFFER_EXT, depthbuffer);
和FBO的绑定函数一样，第一个参数是“目标(target)”，指的是你要与哪个目标进行绑定，目前来说，只能是一些预定义好的目标。变量dephtbuffer用来保存对像标识。

这里有一个关键的地方，也就是我们生成的渲染缓冲对像，它本身并不会自动分配内存空间。因此我们要调用OpenGL的函数来给它分配指定大小的内存空间，在这里，我们分配一个固定大小的深度缓显空间。

glRenderbufferStorageEXT(GL_RENDERBUFFER_EXT, GL_DEPTH_COMPONENT, width, height);
上面这一函数成功运行之后，OpenGL将会为我们分配好一个大小为width x height的深度缓冲区。注意的是，这里用了GL_DEPTH_COMPONENT,就是指我们的空间是用来保存深度值的，但除了这个之外，渲染缓冲区 还可以用来保存普通的RGB/RGBA格式的数据或者是模板缓冲的信息。

准被好了深度缓存的显存空间后，接下来要做的工作就是把它与前面我们准备好了的FBO对像绑定在一起。

glFramebufferRenderbufferEXT(GL_FRAMEBUFFER_EXT, GL_DEPTH_ATTACHMENT_EXT, GL_RENDERBUFFER_EXT, depthbuffer);
这个函数看起来有点复杂，但其实它很好理解的。它要做的全部工作就是把把前面我们生成的深度缓存对像与当前的FBO对像进行绑定，当然我们要注意一个FBO有多个不同绑定点，这里是要绑定在FBO的深度缓冲绑定点上。



加入用于渲染的纹理
到现在为止，我们还没有办法往FBO中写入颜色信息。这也是我们接下来正要讨论的，我们有以下两种方法来实现它：

把一个颜色渲染缓冲与FBO绑定。
把一个纹理与FBO绑定。
前者在某些地方会用到，后面的章节我们会深入讨论。现在我们先来说说第二种方法。

在你想要把纹理与一个FBO进行绑定之前，我们得先要生成这个纹理。这个生成纹理的过程种我们平时见到的纹理生成没什么区别。

GLuint img; glGenTextures(1, &img); glBindTexture(GL_TEXTURE_2D, img); glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL);
这个实例中，我们生成一个普通的RGBA图像，大小是width x height,与前面我们生成的渲染缓冲区的大小是一样的，这一点很重要，也就是FBO中所有的绑定对像，都必须要有相同的宽度和高度。还有要注意的就是：这里我们没有上传任何的数据，只是让OpenGL保留分配好的空间，稍后我们将会用到。

生成好纹理之后，接下来的工作就是把这个纹理与FBO绑定在一起，以便我们可以把数据渲染到纹理空间中去。

glFramebufferTexture2DEXT(GL_FRAMEBUFFER_EXT, GL_COLOR_ATTACHMENT0_EXT, GL_TEXTURE_2D, img, 0);
这里再次看到这个看起来非常可怕的函数，当然它也并没有我们想像中那么难理解。参数GL_COLOR_ATTACHMENT0_EXT是告诉OpenGL把纹理对像绑定到FBO的0号绑定点（一个FBO在同一个时间内可以绑定多个颜色缓冲区，每个对应FBO的一个绑定点），参数GL_TEXTURE_2D是指定纹理的格式，img保存的是纹理标识，指向一个之前就准备好了的纹理对像。纹理可以是多重映射的图像，最后一个参数指定级级为0，指的是使用原图像。

最后还有一步要做的工作，就是检查一下FBO的准备工作是否全部完成，是否以经能被正确使用了。

这个测试工作由下面一个函数来完成，它会返回一个当前绑定的FBO是否正确的状态信息。

GLenum status = glCheckFramebufferStatusEXT(GL_FRAMEBUFFER_EXT);
如果所有工作都已经做好，那么返回的状态值是GL_FRAMEBUFFER_COMPLETE_EXT，也就是说你的FBO已经准备好，并可以用来作为渲染对像了。否则就会返回其它一个错误码，通过查找定义文档，可以找到相关的错误信息，从而了角错误大概是在哪一步骤中产生的。
 



渲染到纹理
所有困难的工作就是前面建立FBO环境的部份，剩下来的工作就相当简单了，相关的事情就只是调用一下以下这个函数：glBindFramebufferEXT().

当我们要把数据渲染并输出到FBO的时候，我们只需要用这个函数来把一个FBO对像进行绑定。当我们要停止输出到FBO，我们只要把参数设为0，再重新调用一次该函数就可以了。当然，停止向FBO输出，这也是很重要的，当我们完成了FBO的工作，就得停止FBO，让图像可以在屏幕上正确输出。

glBindFramebufferEXT(GL_FRAMEBUFFER_EXT, fbo); glPushAttrib(GL_VIEWPORT_BIT); glViewport(0,0,width, height); // Render as normal here // output goes to the FBO and it's attached buffers glPopAttrib(); glBindFramebufferEXT(GL_FRAMEBUFFER_EXT, 0);
上面另外三行代码glPushAttrib/glPopAttrib 及 glViewport，是用来确保在你跳出FBO渲染的时候可以返回原正常的渲染路径。glViewport在这里的调用是十分必要的，我们不要常试把数据渲染到一个大于或小于FBO大小的区域。 函数glPushAtrrib 和 glPopAttrib 是用来快速保存视口信息。这一步也是必要的，因为FBO会共享主上下文的所有信息。任何的变动，都会同时影响到FBO及主上下文，当然也就会直接影响到你的正常屏幕渲染。

这里一个重要信息，你可能也注意到了，我们只是在绘制的时候绑定或解除FBO，但是我们没有重新绑定纹理或渲染缓冲区，这里因为在FBO中会一直保存了这种绑定关系，除非你要把它们分开或FBO对像被销毁了。


使用已渲染出来的纹理
来到这里，我们已经把屏幕的数据渲染到了一个图像纹理上。现在我们来看一看如何来使用这张已经渲染好了的图像纹理。这个操作的本身其实是很简单的，我们只要把这张图像纹理当作普通纹理一样，绑定为当前纹理就可以了。

glBindTexture(GL_TEXTURE_2D, img);
以上这一函数调用完成之后，这张图像纹理就成了一个在绘图的时候用于被读取的普通纹理。

根据你在初始化时所指定的不同纹理滤波方式，你也许会希望为该纹理生成多重映像(mipmap)信息。如果要建立多重映像信息，多数的人都是在上传纹理数据的时候，通过调用函数gluBuild2DMipmaps()来实现，当然有些朋友可能会知道如何使用自动生成多重映像的扩展，但是在FBO扩展中，我们增加了第三种生成映像的方法，也就是使用GenerateMipmapEXT()函数。

这个函数的作用就是让OpenGL帮你自动创建多重映像信息。中间实现的过程，根据不同的显卡会有所不同，我们只关心它们最终的结果是一样就行了。值得注意的是：对于这种通过FBO渲染出来的纹理，要实现多重映像的话，只有这一种方法是正确的，这里你不可以使用自动生成函数来生成多重映像，这其中的原因有很多，如果你想深入了解的话，可以查看一下技术文档。

使用这一函数使方便，你所要做的就是先把该纹理对像绑定为当前纹理，然后调用一次该函数就可以了。

glGenerateMipmapEXT(GL_TEXTURE_2D);
OpenGL将会自动为我们生成所需要的全部信息，到现在我们的纹理便可以正常使用了。

一个重点要注意的地方：如果你打算使用多重映像(如 GL_LINEAR_MIPMAP_LINEAR)，该函数glGenerateMipmapEXT()必须要在执行渲染到纹理之前调用。

在创建纹理的时候，我们可以按以下代码来做。

glGenTextures(1, &img); glBindTexture(GL_TEXTURE_2D, img); glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL); glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE); glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE); glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR); glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR); glGenerateMipmapEXT(GL_TEXTURE_2D);
到现在，这张纹理和普通纹理没什么区别，我们就按处理普通纹理的方法来使用就可以了。

清理
最后，当你完成了所有的FBO操作之后，请别忘了要清理或删除掉那些不要了的FBO对像，和清理纹理对像相似，这一步只要以下一个函数就可以完成：

glDeleteFramebuffersEXT(1, &fbo);
同样的，你如果分配了渲染缓冲对像，也别忘了要把它清理掉。本实例中我们分配的是深度缓存渲染对像，我们用以下函数来清除它：

glDeleteRenderbuffersEXT(1, &depthbuffer);
到这里，所有的FBO对像及渲染缓冲都被释放掉了，我们的清理工作也就完成了。

最后的思考
这一篇文章只是对FBO扩展的一个初步介绍，希望对你有所帮助，更多详细的知识，可以查看一下FBO spec ,或者看一下《More OpenGL Game Programming》这本书中关于扩展部分的章节。

问题的返溃及相关技术的讨论，可以登陆物理开发网的GPGPU/CUDA论坛进行交流。

在文档结束之前，我要说一下在使用FBO来写程序的过程中，一些值得我们去注意的地方： 

就目前来说，你没办法得到模版缓冲的绑定点。虽然在技术上是定义了这么一种深度模版的纹理格式，目的是让我们可以渲染到模版，但这一技术到目前为止还缺乏硬件的支持。
不要频繁地创建及销毁FBO对像。好的做法应该是在程序建立的同时生成FBO对像，然后在我们需要用到的地方使用它。 
一个纹理，如果被定义为用于做渲染纹理，那么我们就要尽量避免使用glTexImage之类的函数来修改该纹理的数据，这样做多数情况下会让你的程序出现问题。
本文示例程序中要注意的地方
对应这篇文章所讨论的内容，我们写了一个相应的程序，其功能就是给FBO加入一个深度缓冲对像及一个纹理对像。我们发现，在ATI的显卡中有一个bug，也就是当我们给同时FBO加入一个深度缓冲及一个纹理的时候，就会出现严重的冲突。从这里也告诉我们，当我们在写好一个FBO相关的程序的时候，一定要在不同的硬件及不同的驱动下进行广泛的测试，直到没有任何渲染问题为止。

I'd also like to put out a big thanks to Rick Appleton for helping me test out and debug the code on NVIDA hardware, couldn't have done it without you mate :)

本程序需要有GLUT函数库的支持才能正确运行，我使用的是FreeGLUT.

FBO_Example.zip程序下载

 

 

在上一篇文章OpenGL FrameBuffer object 101中，我们大概讲述了FBO的一些基础应用，文章中主要介绍了如何生成一个FBO，如何把数据渲染到一个单一的纹理上，以及把这个纹理在别的地方做一些应用。然而FBO扩展并不紧紧只能做到这些。在上一篇文章中我们主要讲述了FBO的一个综合特征:绑定点(attachment point)。

在本篇文章中，我们将会进一步来讲述FBO的一些深层次概念及应用。首先，我们来看一下如何在一个FBO对像中，通来循环多次渲染，实现把数据渲染到多个纹理上。讲完这个之后，我们再来看一下如何通过使用OpenGL高级着色语言(GLSL)，实现在同一时间渲染输出到多个纹理上，当然，这里还需要用到绘图缓冲扩展(Draw Buffers extension)。

一个FBO与多个纹理
在上一篇文章中，我们讲述了如何把一个纹理绑定到一个FBO中，用来作为一个颜色渲染对像(colour render target)。我们主要用到了下面这个函数。

glFramebufferTexture2DEXT(GL_FRAMEBUFFER_EXT, GL_COLOR_ATTACHMENT0_EXT, GL_TEXTURE_2D, img, 0);
或许你还记得，在这个函数中，我们通过img这个保存有纹理标志的变最来把对应的纹理绑定到当前所启用的FBO中去。在篇文章中，我们来着重关注一下第二个参数：GL_COLOR_ATTACHMENT0_EXT。

这个参数就是告诉OpenGL，把纹理绑定到FBO的0号颜色绑定点中去。然而，一个FBO对像会有多个颜色绑定点可以供我们使用。当前，规格说明书上说允许有16个绑定点(GL_COLOR_ATTACHMENT0_EXT 到 GL_COLOR_ATTACHMENT15_EXT) ，每一个绑定点都可以与一个单独的纹理进行绑定。当然这个绑定点的个数会受到硬件及其驱动的限制，我们可以用以下的函数来查询绑定点个数的最大值：

GLuint maxbuffers;
glGetIntergeri(GL_MAX_COLOR_ATTACHMENTS, &maxbuffers);
在这里，变量maxbuffers保存了颜色绑定点的最大值，在写本文章的时候，当前显卡硬件返回来的这个最大值一般是4。

所以，如果我们想把纹理标识量img与第二个颜色绑定点进行绑定的话，上面对应的函数就得做以下相应的修改：

glFramebufferTexture2DEXT(GL_FRAMEBUFFER_EXT, GL_COLOR_ATTACHMENT1_EXT, GL_TEXTURE_2D, img, 0);
正如你所看到的，想要增加一个绑定纹理，那是相当容易的事情。但是我们又该如何让OpenGL分别把数据渲染到这些纹理上呢？

选择输出目标
OK，我们现在回头来看一下这个特别的函数:glDrawBuffer()，一般我们在OpenGL开始的时候就会用到它。

这个函数，以及与它密切相关的一个函数glReadBuffer()，就是用来告诉OpenGL它应该往哪里写入数据以及应该从哪里读取数据。在默认的情况下，如果是单缓冲环境，两者都是对前缓冲(GL_FRONT)进行读写，而双缓冲环境则是对后缓冲(GLBACK)进行读写。但是在FBO扩展出来之后，这个函数的功能就被修改了，它允许你选择GL_COLOR_ATTACHMENTx_EXT来作为渲染输出或读取的目标（这里'x'指的就是FBO绑定点数字）。

当你绑定启用一个FBO对像的时候，系统会自动把当前颜色输出目标指向GL_COLOR_ATTACHMENT0_EXT，也就是0号绑定点所绑定的纹理。因此，如果你就是想把数据输出到这个默认的颜色绑定点的话，你不需要作任何额外的改动。但是当我们要想输出到别的缓冲区的时候，我们就得亲自告诉OpenGL我们所要的选择。

因此，如果我们想要渲染到GL_COLOR_ATTACHMANT1_EXT，那么我们就必须先启用一个FBO，并正确地指定一个写入缓冲的绑定点。假设我们已经为FBO的1号颜色绑定点绑定好了一个纹理对像，下面就是实现代码：

glBindFrameBuffer(GL_FRAMEBUFFER_EXT, fbo);
glPushAttrib(GL_VIEWPORT_BIT | GL_COLOR_BUFFER_BIT);
glViewport(0,0,width, height);
// Set the render target
glDrawBuffer(GL_COLOR_ATTACHMENT1_EXT);
// Render as normal here
// output goes to the FBO and it抯 attached buffers
glPopAttrib();
glBindFramebufferEXT(GL_FRAMEBUFFER_EXT, 0);
值得注意的是，这里用到了glPushAttrib()函数，它主要是用来保存视口及颜色缓冲的一些属性，因为在FBO运算过程中我们要对这些属性进行修改。在FBO运算完成之后，我们可以使用glPopAttrib()函数来还原之前的设置。这样做主要是因为在FBO运算过程中的一些属性的改变会直接影响到主渲染程序，通过属性还原，能让主程序渲染还原到正常状态。

当我们把多个纹理绑定到一个FBO对像的时候，有一个非常重要的地方，那就是所有这些纹理都要有同样大小的尺寸及颜色深度。所以，我们不能把一个512*512 32bit的纹理与一个256*256 16bit的纹理绑定到同一个FBO对像中去。因而，如果你能够接受这一小小的限制的话，便可以实现用一个FBO渲染输出到多个纹理上，这比起在多个FBO对像中进行切换，速度会快很多。当然，多FBO对像切换也不算是什么极度缓慢的操作，但是尽量避免不必要的开销，通常都是一种比较好的编程习惯。

第一个例子
在第一个示例程序中，演示了如何渲染到2个纹理上，当然这里一个接一个地渲染输出，然后把这些纹理应用到另一个立方体上。代码是基于上一篇文章所写的例子，只是作了一些细小的变动而已。

首先，在初始化函数中，我们是启动FBO的是候把第二个纹理也绑定到FBO对像中去。注意如何有别于与第一个纹理的绑定，这里使用GL_COLOR_ATTACHMENT1_EXT作为绑定点。

对于场景的渲染输出基本上都是相同的，只不过这里我们一共进行了两次绘图，第一次用立方体原来的颜色进行绘图，而第二次绘图的时候把颜色的亮度调为原来的一半。

你或许已经注意到了，在示例程序中，当我们渲染输出到FBO的时候，我们要明确地告诉OpenGL，先是渲染到GL_COLOR_ATTACHMENT0_EXT,然后是GL_COLOR_ATTACHMAENT1_EXT。这是因为FBO会记住上一次你让它渲染输出的缓冲区。因此，在绘图函数中当我们第二次绘图的时候，第一个绑定的纹理作为目标输出是不会自动更新的，直到我们调用glDrawBuffer()函数。想要看一下这一函数所影响的效果的话，可以注释掉第103行，这行就是一个glDrawBuffer()函数的调用，你将会看到这时立方左手边的纹理再也不会出现变化。

多个渲染目标(Multiple Render Targets)
现在我们知道如何把多个纹理绑定到一个FBO中去，然而我们仍然是一次只绘制一张纹理，然后通过切换绘图目标实现对多个纹理的写入，有没有更有用更高效的方法呢？在本文章开头曾提及过，我们要介绍如何实现在同一时间里渲染输出到多个纹理上。

其实，一旦你明白如何绑定多个纹理，剩下要做就非常简单了。你现在还需要用到的技术包括有绘图缓冲扩展(Draw buffers extension)和OpenGL着色语言(GLSL)，而这两者现在都成了OpenGL2.0内核的组成部份。

绘图缓冲扩展（ Draw Buffers Extension）
现在介绍第一个扩展：绘图缓冲区的建立。建立绘图缓冲，我们使用一个系统提供的函数glDrawBuffer()，你也许还记得起这个函数，在前面我们说过，它可以用来指定当前渲染输出的颜色缓冲区。但是在绘图缓冲扩展中，这个函数的功能也同时得到了扩展，它可以用来指定多个同时写入的颜色缓冲区。一次可以同时写入的缓冲的个数，可以用以下函数来查询：

GLuint maxbuffers;
glGetIntergeri(GL_MAX_DRAW_BUFFERS, &maxbuffers);
函数正确执行之后，变量maxBuffers保存了我们一次可以同渲染的缓冲区的个数，在我写这个文档的时候，这个数字一般是4，但是最新显卡GeForce8x00系列允许我们一次同时输出到8个缓冲区。

因而，如果我们已经把两个纹理分别绑定到绑定点0和1，现在我们想同时渲染输出到这两个纹理上，我们就可以按以下代码来写：

GLenum buffers[] = { GL_COLOR_ATTACHMENT0_EXT, GL_COLOR_ATTACHMENT1_EXT };
glDrawBuffers(2, buffers);
当上面的函数正确运行之后，OpenGL 便建立了一个双颜色缓冲渲染输出的环境。

使用 FBO 和 GLSL 实现MRT
现在，如果我们使用标准的固定功能管线来进行渲染，两个纹理会得到同样的数据。然而，如果使用GLSL来重写片段着色代码，我们就可以做到把不同的数据发送到这两个纹理上。

通常来说，当你写一个GLSL的片段着色程序的时候，你会把颜色值输出到gl_FragColor中去，正常情况下，这个颜色值便会被写入到帧缓冲区中去。然而，在这里，我们还有第二种颜色信息输出的方法，那就是使用gl_FragData[]数组。

这个特别的变量允许我们直接指定数据往哪里走。数据输出会对应哪一个纹理呢？这就与函数glDrawBuffers()中给定的参数的对应顺序有关。如上面这种情况，缓冲区的对应关系就如下图所示：

glDrawBuffers value
FragData syntax
GL_COLOR_ATTACHMENT0_EXT
gl_FragData[0]
GL_COLOR_ATTACHMENT1_EXT
gl_FragData[1]
上面函数调用的时候，如果参数顺序发生了改变，那么对应的映射关系也会发生改变，如下所示：

GLenum buffers[] = { GL_COLOR_ATTACHMENT1_EXT, GL_COLOR_ATTACHMENT0_EXT };
glDrawBuffers(2, buffers);
glDrawBuffers value
FragData syntax
GL_COLOR_ATTACHMENT1_EXT
gl_FragData[0]
GL_COLOR_ATTACHMENT0_EXT
gl_FragData[1]
假如说我们想把绿色输出到其中一个目标而蓝色输出到另一个，GLSL的代码就可以这样写，如下：

#version 110
void main()
{
gl_FragData[0] = vec4(0.0, 1.0, 0.0);
gl_FragData[1] = vec4(0.0, 0.0, 1.0);
}
第一行是说我们驱动至少支持GLSL 1.10以上（OGL2.0）。函数主体的功能就只是把绿色写入到第一个缓冲区，蓝色定入到第二个。

译注：CG代码如下：

void main(out float4 col0:COLOR0,out float4 col0:COLOR1)
{
col0 = float4(0.0,1.0,0.0,1.0);
col1 = float4(0.0,0.0,1.0,1.0);
}
 

第二个例子
第二个例子是第一个例子与上一篇文章的例子的结合。它实现了第一个例子同样的输出，但是这里我们只把立方体绘制了一次。我们通过使用一个着色程序来实现控制输出。

程序和之前的差不多，主要的区别在于初始化代码。我们把GLSL代码导入放在一边不谈，因为这个不是本文章讨论的范围。而能让多目标渲染正常工作主要代码就是以下两行：

GLenum mrt[] = { GL_COLOR_ATTACHMENT0_EXT, GL_COLOR_ATTACHMENT1_EXT }
glDrawBuffers(2, mrt);
这两行就是告诉OpenGL，我们希望渲染到两个缓冲区及我们希望渲染到哪两个缓冲区。要记住FBO是有记忆功能的，也就是它会记上一次渲染所使用过的输出目标。通过上面两行代码，我们可以改变FBO渲染输出的目标，让它可以正确地实现同时渲染到多个纹理。

绘图循环函数看起来来前面的十分相似，渲染到FBO还是用到了同样的代码。有所变动的，就是关于绑定并调用GLSL程序的那一部份。GLSL主要就是用来控制颜色的多路输出。后面的代码就基本上和本文第一个例子没什么区别，只是第一个例子中把立方体画了两次，而这里只要画一次就可以了。

关于程序中的两个GLSL着色程序，在这里稍为提及一下，大体上看一下它们是如何让MRT正常工作的。

顶点着色程序，就是对于你发送到显卡上的每一个顶点都会运行一遍的一段代码。本程序中只是简单地把每个顶点的颜色值通过glColor()传递级片段着色程序，并对每个顶点进行了一些必要的矩阵变换，使得我们能在正确的位置绘制一个正方体。

片段着色程序的代码如下：

#version 110
void main(void)
{
gl_FragData[0] = vec4(gl_Color.r, gl_Color.g,gl_Color.b,1.0);
gl_FragData[1] = vec4(gl_Color.r/2.0, gl_Color.g/2.0,gl_Color.b/2.0,1.0);
}
这里关键的地方就是两个gl_FragData，它们用来明确它定我们到底要要把数据写入到哪个缓冲区中去。本实例中gl_FragData[0]指的是第一个纹理，它保存了一份没有被修改过的颜色值，也就是从顶点着颜中传递过来的原始颜色。而对于gl_FragData[1]，它对应的是第二个纹理，同样是用来保存从顶点着色中传递过来的颜色，但颜色的亮度就被改成了原来的一半。从结果上看，它的效果和第一个程序是一样的。

最后的思考
本文章主要通过两个例子是快速地介绍了FBO扩展两种不同的应用。

第一个例子中，允许你使用同一个FBO实现渲染输出到多个纹理中去，从而让我们可以不须要在多个FBO中频繁切换，本例子中所演示的技术是非常有用的，因为当对于在不同的FBO进行切换来说，在同一个FBO中切换不同的渲染目标它的速度要快得多。因此如果你能把你的纹理作一些分组，尽量让多个纹理在同一时间内被渲染，这样会为你节省大量的时间。

第二个例子是让你体会一下这种叫做多渲染目标(Multiple Render Targets)的技术。虽然本文中的关于本技术所举的例子没有很大的实用价值，但是MRT技术是其它许多GPU高级技术的基础，如render-to-vertex buffer及post-processing等，因此这种可以输出到多个颜色缓冲的能力是非常有用的，值得我们大家去深入学习和研究一下。

更多细节，可以查看一下Framebuffer Object 及Draw Buffers 等的规范说明书。在More OpenGL Game Programming 也是一片我写的文章，其中有一个关于FBO和GLSL的章节。对相关的技术也略为讨论了一下。

例子中一些要注意的地方

 

 

 

只有NVidia的N3.0以上的显卡才支持 
只支持少量的浮点纹理格式（GL_RGBA_FLOAT32_ATI） 
老的显卡（GFX系列之前）不支持，GF6以后的才可以。 
稍为要慢一点，GF6600GT的读取纹理的速度是30M/秒，如果该纹理要先经过FBO作处理的话，则会更慢一些。当然，这里的慢是相对于后面将要绍介的第二种方法而言，但如果与把纹理读回到CPU内存的速度相比，这个方法要快得多。

方法 2: 拷贝到像素缓冲对像（PBO）

PBO指的是： pixel buffer object，PBO能直接转换为VBO,用作顶点数组的渲染。

优点:

支技多种PBO的数据格式，而不紧紧是浮点的RGBA。 
更快。在GF6600GT显卡上，渲染FBO + 拷贝FBO到PBO + 渲染VBO，速度是58M/秒。据说现在新的G8显卡，可以省去拷贝这一步，也就是直接渲染到VBO，那速度就更快了。

缺点: 
比较高的显存消耗。 
须要额外的拷贝动作。 
使用多个FBO渲染对像时，每个对像必许用同一种数据格式。例如：如果我们想在一个渲染通道中，把position信息渲染到浮点RGB缓冲区块，然后把normal信息渲染到BYTE-RGB格式的缓冲区，这样做是不行的。 
注意：老的GFX显卡不支持多渲染对像。

实现:
 

方法 1: 在顶点着色其间读取纹理.

步骤 1: 生成 FBO

生成一个FBO，然后把一个用来保存顶点数据的纹理绑定到这个FBO上。

步骤 2: 生成 VBO

生成一个顶点缓冲对像（VBO），用来保保存纹理坐标信息，主要的作用就是为了在顶点着色期间能正确定位并读取到FBO纹理中的数据。

Create a vertex buffer object (VBO) holding texture coordinates for 
referencing the FBO texture (e.g. a position VBO with 
2 float-coordinates per vertex)

( see further down the text how to create )

步骤 3: 渲染 FBO

( 详细说明请看后面的内容 )

步骤 4: 渲染 VBO

这里，我们先从VBO得到顶点坐标，然后用该坐标来访问FBO纹理，并取得纹理数据。

以下例子是一个顶点程序的代码：

 

Code:

// vertex.position is our
// index to the real vertex array
!!ARBvp1.0
OPTION NV_vertex_program3;
PARAM mvp[4] = { state.matrix.mvp };
TEMP real_position;
TEX real_position, vertex.position, texture[0], 2D;
DP4 result.position.x, mvp[0], real_position;
DP4 result.position.y, mvp[1], real_position;
DP4 result.position.z, mvp[2], real_position;
DP4 result.position.w, mvp[3], real_position;
END  ;
 

Quote:
Originally Posted by Nvidia Documentation
顶点纹理的调用会有许多的限制，必须使用GL_TEXTURE_2D的纹理对像，目前只支持GL_LUMINANCE_FLOAT32_ATI 和 GL_RGBA_FLOAT32_ATI 两种数据格式，这两种格式都表示只支持32-bit的浮点数据，前者是单通道，后者是四通道。值得注意的是：如果使用其它的纹理格式，或用了一些不被支持的过滤模式，会造成一些问题，显卡驱动可能会退回到软件模式下进行顶点处理。
以下是一个正确的代码写法。

	GLuint vertex_texture;
glGenTextures(1, &vertex_texture);
glBindTexture(GL_TEXTURE_2D, vertex_texture);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE_FLOAT32_ATI, width, height, 0,GL_LUMINANCE,GL_FLOAT, data);
方法2. 拷贝到像素缓冲对像 (PBO).

步骤 1: 生成一个用作像素缓冲对像的VBO:

示例:

GLuint vbo_points_handle;
glGenBuffersARB(1, &vbo_vertices_handle);
glBindBufferARB(GL_PIXEL_PACK_BUFFER_EXT, vbo_vertices_handle); 	glBufferDataARB(GL_PIXEL_PACK_BUFFER_EXT, vbo_points.size()*4*sizeof(float ),NULL,GL_DYNAMIC_DRAW_ARB );
步骤 2: 生成一个 FBO.

多个渲染对像可以帮助我们实现同一时间写入顶点/法线/副法线。以下是一个生成FBO的示例：

 

GLuint fb_handle;
glGenFramebuffersEXT(1,&fb_handle);
fbo_tex_vertices = NewFloatTex(tex_width,tex_height,0);
fbo_tex_normals = NewFloatTex(tex_width,tex_height,0);
这段代码是演示如何生成浮点数据的纹理。

/** * Sets up a floating point texture with NEAREST filtering.
* (mipmaps etc. are unsupported for floating point textures) */
void setupTexture (const GLuint texID,int texSize_w,int texSize_h)
{
// make active and bind
glBindTexture(textureParameters.texTarget,texID);
// turn off filtering and wrap modes
glTexParameteri(textureParameters.texTarget, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(textureParameters.texTarget, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(textureParameters.texTarget, GL_TEXTURE_WRAP_S, GL_CLAMP);
glTexParameteri(textureParameters.texTarget, GL_TEXTURE_WRAP_T, GL_CLAMP);
// define texture with floating point format
glTexImage2D(textureParameters.texTarget,0,textureParameters.texInternalFormat,
texSize_w,texSize_h,0,textureParameters.texFormat,GL_FLOAT,0);
// check if that worked
if (glGetError() != GL_NO_ERROR)
{
printf("glTexImage2D():[FAIL]  ");
// PAUSE();
exit (ERROR_TEXTURE);
}
else if (mode == 0)
{
printf("glTexImage2D():[PASS]  ");
}
// printf("Created a %i by %i floating point texture.  ",texSize,texSize);
}
 

注意：即使我们可以生成RGB的纹理，但它内部格式可能还是RGBA的，用glReadPixels()来读取数据时由于要进行格式转换，可能会减慢运行的速度。因此，多数情况下，我们尽量使用RGBA的格式。

步骤 3: 渲染 FBO

输入纹理中包含有必要的数据（如：顶点位置、法线等），经过运算之后，把数据保存到输出纹理中。

下面代码是绑定FBO缓冲区：

	  //glBindFramebufferEXT(GL_attach two textures to FBO
glFramebufferTexture2DEXT(GL_FRAMEBUFFER_EXT, attachmentpoints[0],
textureParameters.texTarget, outTexID, 0);
// check if that worked
if (!checkFramebufferStatus())
{
printf("glFramebufferTexture2DEXT():	 [FAIL]  ");
// PAUSE();
exit (ERROR_FBOTEXTURE);
}
else if (mode == 0)
{
printf("glFramebufferTexture2DEXT():	 [PASS]  ");
}
下面代码通过渲染一个四边形来触发FBO的运算。

	  // make quad filled to hit every pixel/texel
// (should be default but we never know)
glPolygonMode(GL_FRONT,GL_FILL);
if (textureParameters.texTarget == GL_TEXTURE_2D)
{
// render with normalized texcoords
glBegin(GL_QUADS);
glTexCoord2f(0.0, 0.0); 	glVertex2f(0.0, 0.0);
glTexCoord2f(1.0, 0.0); 	glVertex2f(outTexSizeW, 0.0);
glTexCoord2f(1.0, 1.0); 	glVertex2f(outTexSizeW, outTexSizeH);
glTexCoord2f(0.0, 1.0); 	glVertex2f(0.0, outTexSizeH);
glEnd();
}
else
{
// render with unnormalized texcoords
glBegin(GL_QUADS);
glTexCoord2f(0.0, 0.0); 	glVertex2f(0.0, 0.0);
glTexCoord2f(outTexSizeW, 0.0); 	glVertex2f(outTexSizeW, 0.0);
glTexCoord2f(outTexSizeW, outTexSizeH); 	glVertex2f(outTexSizeW, outTexSizeH);
glTexCoord2f(0.0, outTexSizeH); 	glVertex2f(0.0, outTexSizeH);
glEnd();
} 
gluOrtho2D是必须要有的! 如果没有, glReadPixels运行时会出错

 

	  /** * Creates framebuffer object, binds it to reroute rendering operations
* from the traditional framebuffer to the offscreen buffer
*/
void initFBO(void)
{
// create FBO (off-screen framebuffer)
glGetIntegerv(GL_DRAW_BUFFER, &_currentDrawbuf);
// Save the current Draw buffer
glGenFramebuffersEXT(1, &fb);
// bind offscreen framebuffer (that is, skip the window-specific render target)
glBindFramebufferEXT(GL_FRAMEBUFFER_EXT, fb);
// viewport for 1:1 pixel=texture mapping
glMatrixMode(GL_PROJECTION);
glLoadIdentity();
gluOrtho2D(0.0, outTexSizeW, 0.0, outTexSizeH);
glMatrixMode(GL_MODELVIEW);
glLoadIdentity();
glViewport(0, 0, outTexSizeW, outTexSizeH);
}
步骤 4: 拷贝 FBO 的数据到 PBO

Example:

	  /** *Copy from FBO to PBO * */
void copyFromTextureToPBO(GLuint pboID,int texSize_w,int texSize_h)
{
glReadBuffer(attachmentpoints[0]);
glBindBufferARB(GL_PIXEL_PACK_BUFFER_EXT, pboID);
glReadPixels(0, 0, texSize_w,texSize_h,		textureParameters.texFormat,GL_FLOAT, 0);
glReadBuffer(GL_NONE);
glBindBufferARB(GL_PIXEL_PACK_BUFFER_EXT, 0 );
}
( vbo_vertices.size() == tex_width * tex_height )

步骤 5: 渲染 VBO:

	  glBindBufferARB(GL_ARRAY_BUFFER_ARB, vbo_vertices_handle);
glEnableClientState(GL_VERTEX_ARRAY);
glVertexPointer  ( 4, GL_FLOAT,4*sizeof(float), (char *) 0);
glBindBufferARB(GL_ARRAY_BUFFER_ARB, vbo_normals_handle);
glEnableClientState(GL_NORMAL_ARRAY);glNormalPointer(GL_FLOAT, 4*sizeof(float), (char *) 0 );
glDrawArrays( GL_TRIANGLES, 0,vbo_vertices.size() );
glDisableClientState(GL_NORMAL_ARRAY);
glDisableClientState(GL_VERTEX_ARRAY);
示例说明
 

本文章所带的例子，实现了在GPU中计算B样条曲线的功能，用到的技术有：VBO,FBO,Render to vertex,CG,B-spline.实现过程如图所示：

 

主要分为三个阶段：

第一阶段：GPU片段着色运算，生成FBO顶点数据。

把样条控制点的数据发送到GPU的一个输入纹理（控制点纹理）。 
在片段处理单元中读取控“制点纹理”中的数据，使用B样条插值函数，计算插值顶点，把结果保存到FBO所绑定的输出纹理（插值纹理）中。
第二阶段：FBO拷贝到PBO

把插值纹理通过使用glReadPixels()函数，拷贝到PBO中。

第三阶段：渲染VBO

使用glDrawArrays(); 来渲染样条曲线。当然这里我们要把前面生成的PBO数据指定为一个VBO对像。

整个过程的插值运算及数据拷贝，都是在GPU中进行，最终的顶点数据直接用顶点数组来作渲染，数据没有返回到CPU中因此速度会非常快。



## Surface和Texture
    > 在sdl2里，surface都是在系统内存里存储的，texture都是在显示内存里存储的。
　　  只有texture可以享受硬件加速带来的好处。
　　  用OpenGLSDL_texture就没有用了。
　　 > 那本来就是对OpenGL的贴图的封装。