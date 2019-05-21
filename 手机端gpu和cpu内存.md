* Android中内存架构
(''')
	现在android中用的是统一内存架构，GPU和CPU共享一个物理内存，通常我们有“显存”和“内存”两种叫法，可以认为是这块物理内存的所有者不同，但这段映射到cpu，就是通常意义上的内存；当映射到gpu，就是通常意义上的显存。并且同一时刻只会映射到一个device。
一个简单的纹理创建，首先我们需要先把纹理数据加载到一段内存中A中，然后调用glTexImage2D来上传纹理的时候，会调用gles驱动的内存分配接口来分配一段内存B（最终是调用gralloc分配），并且映射到cpu。然后会调用一个定制的memcpy来把A的数据拷贝到B。这里，虽然都是在同一块物理内存中，但是OpenGL的spec如此，还是需要一次拷贝。渲染的时候，B会被映射到GPU上，让GPU可以读取。
而GPU渲染内容从APP到SF，是不会有搬运，至少Mali和sgx PowerVR不会。厂家的opengl实现，是调用BufferQueue这个类来获取内存来渲染的，gpu渲染完毕再丢回BuffferQueue （Queue/Dequeue）。而surfaceFlinger会去请求有没有可以已经渲染好的东西，以及会把显示完的一帧丢回这个queue（Aquire/Release）。只要进程还活着，还可见，这个queue中往往有3块格式相同的buffer会循环使用。
这个类是实现在SurfaceFlinger模块下，如上提供了两组接口给生产者（Gpu）和消费者（SF/display），所有帧 buffer的传递显然都是直接传递指针，也就是不会有拷贝。这部分是android公共的实现。
(''')