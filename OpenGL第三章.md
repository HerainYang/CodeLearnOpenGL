# OpenGL第三章

## 深度测试

深度缓冲和颜色缓冲一样，每个片段都储存了对应的信息，通常是24位的。

当进行深度测试的时候，OpenGL会将一个片段的深度值和深度缓冲的内容（深度缓冲是一块内存区域，它包括了每个像素的深度值）对比，OpenGL会执行一个深度测试，如果这个测试通过了，深度缓冲会更新为新的深度值，反之则丢弃该片段。

深度测试位于片段着色器和模板测试后，在屏幕空间中运行的，所以和`glViewport`定义的视口息息相关，并且可以通过`gl_FragCoord`直接访问，其中`(0,0)`定义的是屏幕左下角，`gl_FragCoord`中包含的z分量则是片段的深度值。

> 目前的GPU都支持一个叫做提前深度测试的功能，它会在片段着色器前运行，如果确定一个片段永远不可见（对于当前帧），就提前丢弃。当然如果我们更改了片段的深度值，这个功能就不会运作了。

```c++
glEnable(GL_DEPTH_TEST);
//开启深度测试
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
//每帧调用，清除深度缓冲
glDepthMask(GL_FALSE);
//如果不希望更新深度缓冲，我们可以设定禁用深度缓冲的写入
glDepthFunc(GL_LESS);
//如果我们想更改深度缓冲的比较方法，可以参考下表来设定这个函数，当然，这个函数不能和上一个同时使用
```

| 函数        | 描述                                         |
| :---------- | :------------------------------------------- |
| GL_ALWAYS   | 永远通过深度测试                             |
| GL_NEVER    | 永远不通过深度测试                           |
| GL_LESS     | 在片段深度值小于缓冲的深度值时通过测试       |
| GL_EQUAL    | 在片段深度值等于缓冲区的深度值时通过测试     |
| GL_LEQUAL   | 在片段深度值小于等于缓冲区的深度值时通过测试 |
| GL_GREATER  | 在片段深度值大于缓冲区的深度值时通过测试     |
| GL_NOTEQUAL | 在片段深度值不等于缓冲区的深度值时通过测试   |
| GL_GEQUAL   | 在片段深度值大于等于缓冲区的深度值时通过测试 |

### 深度值精度

深度缓冲包含一个介于`[0.0,1.0]`之间的深度值，它会和观察者视角的所有片段的z值进行比较，将物体变换到这个范围的方法是：
$$
F_{depth}=\frac{1/z-1/near}{1/far-1/near}
$$
其中如果忘了far和near是什么，请查看坐标系统一章。为什么要用这么一个非线性的函数呢，由下图可以看出，这样一个方程可以在距离接近近平面的时候提供更大的精度。

![img](https://learnopengl-cn.github.io/img/04/01/depth_non_linear_graph.png)

### 深度冲突

深度冲突指的是当物体比较近的时候，可能会出现闪烁现象，这是因为精度不够计算它们的先后顺序，要解决这个问题有三种方法：

1. 不要把多个物体放得太近。
2. 将近平面设置的远一点。
3. 使用更高的精度。

## 模板测试

在细说模板测试之前，我们需要了解关于测试的两个东西，一个是模板缓冲，一个是片段/像素的模板值。

模板缓冲里面按照规则存储了参考值，每个参考值都是8位长度，在测试中，对应位置的参考值会和片段的模板进行比较，按照规则选择留下或者丢弃，这些规则有：

* `GL_NEVER`，`GL_ALWAYS`
* `GL_LESS`，`GL_GREATER`
* `GL_LEQUAL`，`GL_GEQUAL`
* `GL_EQUAL`，`GL_NOTEQUAL`

举一个简单的例子：

![img](https://learnopengl-cn.github.io/img/04/02/stencil_buffer.png)

在这个例子中，模板测试应用的是等于的规则，当片段模板和模板缓冲都为1，才会被绘制。

模板测试的大致步骤为：

* 启用模板缓冲写入。
* 渲染物体，更新模板缓冲内容。
* 关闭模板缓冲的写入。
* 渲染其他物体，但根据模板缓冲丢弃特定片段。

和深度测试一样，模板测试也有以下函数用法：

```c++
glEnable(GL_STENCIL_TEST);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
//对应深度测试的glDepthMask(GL_FALSE)，在模板测试中禁用模板缓冲的写入用的是：
glStencilMask(0x00);
//这样设置，输入的值会和0x00做AND运算，输出的结果就都是0x00，相反，如果要完整保留输入，则应该设置为0xFF，注意0x00会导致glClear不能正常清空
```

### 模板函数

说完了模板的设置，那如何具体应用模板呢，这就涉及到模板函数。从上文我们可以知道，“应用”模板包括两个部分：设置比较方法，设置比较完之后的动作。

我们通过`glStencilFunc(GLenum func, GLint ref, GLuint mask)`来设置模板测试通过的条件，通过`glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)`来设置之后的行为。

```c++
glStencilFunc(GLenum func, GLint ref, GLuint mask);
//第一个参数表示如何比较，在上文我们提到过
//第二个参数设置了模板测试的参考值，它会被用来和模板缓冲比较
//第三个参数是一个掩码，在输入的参考值会和它做AND运算，然后再被存储

glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass);
//第一个参数表示模板测试失败时采取的行为
//第二个参数表示模板测试通过但深度测试失败时采取的行为
//第三个参数表示两者都通过的行为
```

| 行为         | 描述                                               |
| :----------- | :------------------------------------------------- |
| GL_KEEP      | 保持当前储存的模板值                               |
| GL_ZERO      | 将模板值设置为0                                    |
| GL_REPLACE   | 将模板值设置为glStencilFunc函数设置的`ref`值       |
| GL_INCR      | 如果模板值小于最大值则将模板值加1                  |
| GL_INCR_WRAP | 与GL_INCR一样，但如果模板值超过了最大值则归零      |
| GL_DECR      | 如果模板值大于最小值则将模板值减1                  |
| GL_DECR_WRAP | 与GL_DECR一样，但如果模板值小于0则将其设置为最大值 |
| GL_INVERT    | 按位翻转当前的模板缓冲值                           |

### 描绘物体轮廓

下面会给出一段代码，它实现了物体的描边功能：

```c++
int main() {
    
   ...

   glEnable(GL_DEPTH_TEST);
   glEnable(GL_STENCIL_TEST);
    //开启模板测试

   glStencilOp(GL_KEEP, GL_KEEP, GL_REPLACE);

   glViewport(0, 0, WIDTH, HEIGHT);

   Shader myShader("/home/herain/Documents/opengl/model/model.vertex", "/home/herain/Documents/opengl/model/model.fragment");
   Shader singleShader("/home/herain/Documents/opengl/model/model.vertex", "/home/herain/Documents/opengl/model/singleColor.fragment");;
    //加载一个只会上单色的片段着色器，用于描边

   Model ourModel("/home/herain/Documents/opengl/model/obj/nanosuit.obj");

   vec3 lightPos(1.2f, 1.0f, 2.0f);

   while (!glfwWindowShouldClose(window)){
       currentFrame = glfwGetTime();
       deltaFrame = currentFrame - lastFrame;
       lastFrame = currentFrame;

       pos_update();

       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);

       glStencilFunc(GL_ALWAYS, 1, 0xFF);
       glStencilMask(0xFF);
       //绘制下面模型的时候，把这个模型所占区域的模板值改成1，可以看到这里始终通过模板测试，看向第9行，我们设置了当通过测试之后，缓冲里面对应区域会更改模板值的选项

       myShader.use();

       myShader.setVec3("viewPos", camera.Position);
       myShader.setVec3("light.position", lightPos);
       myShader.setVec3("light.ambient", vec3(0.2f));
       myShader.setVec3("light.diffuse", vec3(0.5f));
       myShader.setVec3("light.specular", vec3(1.0f));
       myShader.setFloat("shininess", 32);

       mat4 projection = perspective(radians(camera.Zoom), (float)WIDTH / (float)HEIGHT, 0.1f, 100.0f);
       mat4 view = camera.GetViewMatrix();
       myShader.setMat4("projection", projection);
       myShader.setMat4("view", view);

       mat4 model = mat4(1.0f);
       model = translate(model, vec3(0.0, 0.0, 0.0));
       model = scale(model, vec3(1.0f, 1.0f, 1.0f));
       myShader.setMat4("model", model);
       ourModel.Draw(myShader);

       glStencilFunc(GL_NOTEQUAL, 1, 0xFF);
       //上面一次绘制我们把在模型范围内的片段都设置了为1的模板值，其他的地方目前都是0，为了绘制边框的时候不会遮挡原本的模型，我们只在模板值为0的地方绘制边框
       glStencilMask(0x00);
       glDisable(GL_DEPTH_TEST);
       //想象一下市面上应用模型描边的地方，基本都是在模型被其他物体遮挡的时候，所以如果开着深度测试，那么边框就会被遮挡而不会被绘制，所以此处为了穿墙效果，我们暂时把它关掉

       singleShader.use();

       mat4 model1 = mat4(1.0f);
       model1 = translate(model, vec3(0.0, 0.0, 0.0));
       model1 = scale(model, vec3(1.01f, 1.01f, 1.01f));
       singleShader.setMat4("model", model1);
       singleShader.setMat4("projection", projection);
       singleShader.setMat4("view", view);
       ourModel.Draw(singleShader);
       //绘制一个体积大一点的模型，这个模型会全方面包裹真正用于显示的模型

       glStencilFunc(GL_ALWAYS, 0, 0xFF);
       glStencilMask(0xFF);
       glEnable(GL_DEPTH_TEST);
       //最后，为了下一次循环能够正常清空模板值，我们要把模板值设置回可以被更改，另外也要重新打开深度测试

       glfwSwapBuffers(window);
       glfwPollEvents();
   }

   glfwTerminate();
   return 0;
}
```

最后，有的人可能发现，画出来的边框并没有包括住模型，而是朝某个方向偏移了，这是因为模型的中心点不在正中间。为了解决这个问题，我们可以把边框的顶点沿法线方向进行一个平移，即：

```glsl
gl_Position = projection * view * model * vec4(aPos + aNormal * 0.1f, 1.0);
```

## 阴影映射

阴影，是物体遮挡光线后，后面的物体没接收到光的部分会暗于接收到光的部分，而呈现出的效果。以目前技术来说，阴影是消耗性能的大头，目前行业中有用到不同的阴影算法，他们都有自己的优劣，在使用时需要多加斟酌考虑。

游戏中使用较多的是阴影贴图技术，其效果和性能都不错，且容易拓展成更好的算法。

![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_theory.png)

如图中所示，蓝线部分是光线可以到达的区域，黑线部分是被遮挡的区域。联想一下这个和摄像机的相通之处，摄像机中被遮挡的区域，在光线中则是渲染上阴影。

那么怎么判断光线先到达哪个物体呢？很简单，从光源方向发射一条射线，比较射线上碰撞到物体的各个点，选取最近的那一个点为光照投射点，其他则为阴影渲染点。

回想一下深度缓冲相关内容，深度缓冲中存储着从摄像机到片段的距离（归化到`[0,1]`范围），

