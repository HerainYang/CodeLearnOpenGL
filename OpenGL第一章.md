# OpenGL第一章

## OpenGL

OpenGL是一个API，由Khronos组织制定和维护，具体实现方法由显卡开发商自行实现。相比早期OpenGL使用的立即渲染模式（固定渲染管线），现在的OpenGL使用的是更为灵活的**核心模式**（core-profile）。

OpenGL本身是一个状态机，用一系列的变量去告知OpenGL应该如何运作。规范来说，OpenGL的状态被称作**上下文**，开发者通过**状态设置函数**来改变上下文，从而使用OpenGL。

使用OpenGL时，最好用OpenGL定义的类型，也就是在原本的类型前加上`GL`，例如`GLfloat`，从而保证在不同平台上程序都能正常运行。

## 配置环境

本人所使用的环境是Ubuntu20.04，运行命令如下：

```shell
sudo apt-get install build-essential libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev cmake xorg-dev libglew-dev
```

这些依赖库包括了OpenGL的Library，Utilities，Utility Toolkit以及GLEW和GLFW。

* GLFW是一个图形库框架，主要包括了创建，管理窗口，管理OpenGL上下文，处理手柄，键盘，鼠标输入的功能
* GLEW是一个跨平台的拓展库，有助于查询和加载OpenGL的拓展，用于管理OpenGL的函数指针，在调用任何OpenGL函数前都需要初始化GLEW。

安装完之后也要记得把库的引用加入到CMakeLists里，如果你用的不是IDE，那在链接的时候记得使用`-l`链接这些库。

```makefile
target_link_libraries(<project name> glfw GL GLEW ${GL_LIBRARY} m)
```

## 创建窗口

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>
//在包含GLFW的头文件之前，GLEW需要先被包含，正如先前所说，GLFW依赖于OpenGL

const GLuint WIDTH = 800, HEIGHT = 600;

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);

int main() {
   glfwInit();
    //初始化GLFW
    //下面一系列glfwWindowHint用于配置GLFW，这个函数接受两个值，第一个是一个枚举值，用于指向想要更改的选项，第二个参数是一个整型，也就是具体要更改的值。
    //具体的枚举值可以在https://www.glfw.org/docs/latest/window.html#window_hints查看。
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    //设定我们要用的OpenGL版本
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    //设定渲染模式，明确告诉了GLFW后如果再使用老版本的函数就会报错
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);
    //不允许用户调整窗口的大小
    
    //如果是macOS，还需要启用GLFW_OPENGL_FORWARD_COMPAT以指定OpenGL是否前向兼容之前的上下文，具体代码还请自行尝试。

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "LearnOpenGL", nullptr, nullptr);
    //创建窗口，前两个函数指的是窗口的宽高，第三个函数是窗口名称，第四个函数在需要全屏时指定屏幕，第五个函数指向另外一个window，这个新建的window会共享指定window的对象（材质，顶点等等）
    
   if(window == nullptr){
       fprintf(stderr, "Failed to create GLFW window");
       glfwTerminate();
       return -1;
   }
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);
    //设置当前窗口的键盘响应事件，响应函数的原型为：
    //void function_name(GLFWwindow* window, int key, int scancode, int action, int mods)
    //key参数表示按下的按键，scancode是键盘扫描码，action表示它是按下还是释放，mods表示是否支持带有Ctrl，Shift等组合键的操作
    

   glewExperimental = GL_TRUE;
    //初始化GLEW，允许使用更多实验性功能
   if(glewInit() != GLEW_OK){
       fprintf(stderr, "Failed to initialize GLEW");
       return -1;
   }

   int width, height;
   glfwGetFramebufferSize(window, &width, &height);
    //获取窗口的大小
   glViewport(0, 0, width, height);
    //这个函数负责把视锥体截取的图像按照规则规定的宽高显示到屏幕上
    //前两个参数x，y代表了视口的左下角位置，后两个参数w，h代表视口的宽高
    //视口可以想象成窗口内的一个大小不定的画布

   while (!glfwWindowShouldClose(window)){
       //检查窗口是否要求被退出
       glfwPollEvents();
       //检查有没有事件发生，如果有就调用对应的回调函数
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       //指定用于清除画面的背景颜色
       glClear(GL_COLOR_BUFFER_BIT);
       //清除屏幕的颜色缓存

       glfwSwapBuffers(window);
       //交换前后缓冲区的颜色缓冲，然后用于绘制输出
       //这个函数用到了双缓冲技术。传统的单缓冲是指在屏幕上直接绘制图像，因为绘制过程是从左到右，从上到下，不是立刻呈现，所以效果不理想。双缓冲则是在后缓冲上绘制完之后再交换给前缓冲，这样图像就能直接呈现给用户。
   }
    //一个类似Unity update()的循环，让程序持续运行
   glfwTerminate();
    //释放GLFW分配的内存
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){
   printf("%d\n", key);
   if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);
    //如果按下退出键，就关闭窗口
}

```

## 三角形

将物体从3D空间转换到2D空间的过程由图形渲染管线实现，这个过程分作两个部分，一个是坐标转换，一个是上色。要注意2D坐标和像素是两个概念，2D坐标表示一个精准的点，像素则是近似值，坐标要比像素无限小。

管线被分为几个阶段，每个阶段有对应的函数处理，这些函数被称作**着色器**（shader）。有的着色器允许开发者自行配置与替换，OpenGL的着色器使用OpenGL着色器语言（GLSL）编写，下图蓝色部分为可自定义的着色器。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/04/pipeline.png)

一个顶点是一个3D坐标，**顶点属性**可以包括任何我们想用的数据。

顶点着色器将这组数据映射到另一个3D坐标系中。

> 经过顶点着色器处理的数据会被映射到**标准化设备坐标**（Normalized Device Coordinates）中，这个坐标的xyz值在$[-1.0,1.0]$范围中，在这个范围外的点会被裁剪。
>
> OpenGL采用的是右手坐标系，其中y轴方向向上，x轴方向向右，z轴方向向后，原点在图像中心。
>
> ![coordinate_systems_right_handed](https://learnopengl-cn.github.io/img/01/08/coordinate_systems_right_handed.png)
>
> 在这之后，标准化设备坐标会通过**视口转换**（Viewport Transform）转换为**屏幕空间坐标**（Scrren-space Coordinates）。再之后屏幕空间坐标又会被变换为片段输入到片段着色器中。

图元装配指将三个坐标以某种方式（点，线，三角形）组合到一起。

几何着色器可以构造新的顶点来形成新的形状。

光栅化阶段会把坐标映射到屏幕中的像素。

在片段着色器之前，超出视图范围的像素会被裁切，片段着色器会计算一个像素的最终颜色。

测试混合阶段会检测模板值，深度值，alpha值（通常用于混合颜色）。

具体管线内容可以查看本人Unity日常记录，有一章专门针对管线做出了解释。

---

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>

const GLuint WIDTH = 800, HEIGHT = 600;

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);

//每个着色器都起始于一个版本声明，GLSL的版本号和OpenGL的版本是匹配的，例如此处OpenGL为3.3，core表示使用核心模式
const char *vertexShaderSource = "#version 330 core\n"
                                "layout (location = 0) in vec3 aPos;\n"
                                "void main()\n"
                                "{\n"
                                "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
                                "}\0";
//使用in关键字，声明aPos为输入变量，将输入顶点属性（Input Vertex Attribute）引入顶点着色器中
//layout (location=0)设置了输入变量的顶点属性索引值
//由于每个顶点包含的是一个3D坐标，所以使用vec3来存位置信息
//在GLSL中的向量最多有4个分量，xyz和w，w是齐次坐标
//为了设置顶点着色器的输出，我们把位置数据赋值给预设值的gl_Position变量，第四个参数w暂且设置成1

const char *fragmentShaderSource = "#version 330 core\n"
                                  "out vec4 FragColor;\n"
                                  "void main()\n"
                                  "{\n"
                                  "   FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
                                  "}\n\0";
//是用out关键字，声明FragColor为输出变量
//将颜色值赋值给FragColor，第四个参数是透明度

int main() {
   glfwInit();
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "LearnOpenGL", nullptr, nullptr);

   if(window == nullptr){
       fprintf(stderr, "Failed to create GLFW window");
       glfwTerminate();
       return -1;
   }
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);

   glewExperimental = GL_TRUE;
   if(glewInit() != GLEW_OK){
       fprintf(stderr, "Failed to initialize GLEW");
       return -1;
   }

   int width, height;
   glfwGetFramebufferSize(window, &width, &height);
   glViewport(0, 0, width, height);

   GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    //创建着色器对象，返回值为这个对象的ID，因为我们要创建顶点着色器，所以传递参数是GL_VERTEX_SHADER
   glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    //将着色器源码加载到着色器上，第二个参数为源码字符串数量，第三个是具体要传的源码，第四个参数如果为NULL，则默认字符串以\0结尾，如果为具体数字，则为读取的字符数量
   glCompileShader(vertexShader);
    //编译着色器

   GLint success;
   GLchar infoLog[512];
   glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
    //返回对应着色器，对应属性的状态
   if(!success){
       glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
       fprintf(stderr, "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n%s", infoLog);
   }

   GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    //创建片段着色器
   glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
   glCompileShader(fragmentShader);
   glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
   if(!success){
       glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
       fprintf(stderr, "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n%s", infoLog);
   }

    //着色器程序（Shader Program Object）是将多个着色器合并链接之后的最终版本，要使用编译完成的着色器，就必须把它链接到一个着色器程序中，然后在做渲染的时候激活这个着色器程序，以便在我们发送渲染调用的时候使用它
   GLuint shaderProgram = glCreateProgram();
   glAttachShader(shaderProgram, vertexShader);
    //合并顶点着色器
   glAttachShader(shaderProgram, fragmentShader);
    //合并片段着色器
   glLinkProgram(shaderProgram);
    //链接着色器程序
    //激活着色器程序在渲染循环中实现，代码为glUseProgram(shaderProgram)
   glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
   if(!success){
       glGetShaderInfoLog(shaderProgram, 512, NULL, infoLog);
       fprintf(stderr, "ERROR::SHADER::PROGRAM::LINKING_FAILED\n%s", infoLog);
   }

   glDeleteShader(vertexShader);
   glDeleteShader(fragmentShader);
    //链接完成后，着色器对象不再被需要

   GLfloat vertices[] = {
           -0.5f, -0.5f, 0.0f,
            0.5f, -0.5f, 0.0f,
            0.0f,  0.5f, 0.0f
   };
    //顶点数据的写法不止一种，只要能向顶点着色器解释的形式都可以使用
    
   GLuint VBO, VAO;
    //VBO是顶点缓冲对象（Vertex Buffer Objects），它会在显存中存储大量顶点的信息
    //使用缓冲对象可以一次性发送一大批数据到显卡（批处理），来缓解CPU-GPU数据发送较慢的问题
    //这里的VBO是整型类型，因为在这之后这个变量将用于存储顶点缓冲对象的ID
    
    //VAO是顶点缓冲对象的集合，叫做顶点数组对象（Vertex Array Object），它最多有16的属性指针，每一个指针代表顶点的一组信息（位置，材质，颜色等等），这个指针的索引值也就是前面提到过的顶点属性索引值
    //在配置VAO时，我们把VBO集合解析一遍，把各个属性添加到对应的属性指针上后，在绘制阶段只需要绑定对应的VAO就可以了，这种方式让不同的顶点数据和属性配置的切换变得简单，当我们需要绘制另一个图案时，只需要切换到对应的VAO就可以了
    //如果没有VAO，每次切换VBO我们都要重新解析VBO的各个部分是什么，就会重复耗费时间
    //VAO会存储：
    //        |1.glEnableVertexAttribArray和glDisableVertexAttribArray的调用
    //        |2.通过glVertexAttribPointer设置的顶点属性配置
    //        |3.通过glVertexAttribPointer调用进行的顶点缓冲对象与顶点属性链接
    
   glGenVertexArrays(1, &VAO);
    //生成一个VAO，将其ID存储到“&VAO”中
   glGenBuffers(1, &VBO);
    //生成“1”个缓冲对象，将其ID存储到“&VBO”中
    //第一个参数为生成缓冲对象的个数，第二个为存储缓冲对象的数组（此处只有一个所以传地址就行了）
    //此时的缓冲对象还不是顶点缓冲对象

    //****开始配置VAO****
   glBindVertexArray(VAO);
    //绑定VAO

   glBindBuffer(GL_ARRAY_BUFFER, VBO);
    //通过ID（此刻存储在变量VBO中），让对应的缓冲对象都绑定上对应的缓冲类型
    //GL_ARRAY_BUFFER是顶点缓冲对象的缓冲类型，同一时间，一个缓冲类型只能绑定一个缓冲对象，不然缓冲区在接收数据后，会不知道把数据传给哪个缓冲对象
    //如果不太明白工作原理，可以想象一个流水素面，竹子的出口只能对着一个碗，对准之后，所有放入竹子的面都会流入这个碗中
    //我们可以有很多的VBO，只要他们绑定的是不同的缓冲类型（即不同的顶点信息）
    //绑定完成后，任何传送给GL_ARRAY_BUFFER缓冲的数据都会被用于配置VBO（可以理解为转发到VBO里）
    
   glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    //发送数据给GL_ARRAY_BUFFER，最后一个参数有三种形式：
    //                                          |1.GL_STATIC_DRAW 数据几乎不变
    //                                          |2.GL_DYNAMIC_DRAW 数据会变
    //                                          |3.GL_STREAM_DRAW 数据每次绘制都会变
    //根据这个参数，显卡会判断要不要把数据写到能够高速写入的内存部分

   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*) 0);
    //告诉VAO如何解释顶点数据
    //第一个参数是顶点属性索引值，在顶点着色器代码中我们也通过layout (location=0)指定过这个索引值，此处的操作类似于set，顶点着色器代码中的操作类似于get
    //第二个参数是顶点属性的大小，因为它是个vec3，所以大小是3
    //第三个参数是数据类型
    //第四个参数是是否要标准化这组数据，因为我们数据已经在标准范围内了所以设置为不需要
    //第五个参数是步长（stride），指的是第二组顶点的开始位置距离第一组顶点开始位置有多少字节
    //第六个参数是数据在数组（这个数组和GL_ARRAY_BUFFER绑定，存储于缓冲区中，上一条代码已经把数据发到这个缓冲类型的缓冲区中了）中起始位置的偏移量
    
   glEnableVertexAttribArray(0);
    //启用索引值为0的顶点属性，如果不启用，着色器将无法访问这个数据

   glBindBuffer(GL_ARRAY_BUFFER, 0);
    //解绑VAO
    //****结束配置VAO****

   glBindVertexArray(0);

   while (!glfwWindowShouldClose(window)){
       glfwPollEvents();
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT);

       glUseProgram(shaderProgram);
       //激活着色器程序
       glBindVertexArray(VAO);
       glDrawArrays(GL_TRIANGLES, 0, 3);
       //第一个参数为我们打算绘制的图元类型，第二个参数表示顶点数组的起始索引，第三个数组表示我们打算绘制几个顶点
       glBindVertexArray(0);

       glfwSwapBuffers(window);
   }
   glDeleteVertexArrays(1, &VAO);
   glDeleteBuffers(1, &VBO);

   glfwTerminate();
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){
   printf("%d\n", key);
   fflush(stdout);
   if (key == GLFW_KEY_ESCAPE && mode == GLFW_MOD_SHIFT && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);
}
```

对于顶点数据的解释（图示）：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/04/vertex_attribute_pointer.png)

对于VAO的解释（图示）：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/04/vertex_array_objects.png)

术语解释：

* 输入变量：将传入顶点着色器中的参数赋值给一个变量，这个变量叫做输入变量。
* 输出变量：将片段着色器的运算结果放在一个变量中，并指定这个变量为返回值，这个变量叫做输出变量。
* 顶点属性：任何和这个顶点相关的数据，无论是位置，颜色还是别的。
* 顶点属性索引值：专门针对VAO的一个术语，VAO有16个指针，把这16个指针按地址排列下来，每个指针给一个值，这个值就是顶点属性索引值。
* VBO：一系列关于顶点的数据
* VAO：解析一系列VBO，并分类放到不同的指针里，这系列指针的集合就是VAO

## 正方形

当我们要绘制一个矩形的时候，一般都是用两个三角形拼成一个矩形，但如果像这样做矩形：

```c++
GLfloat vertices[] = {
    // 第一个三角形
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, 0.5f, 0.0f,  // 左上角
    // 第二个三角形
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
};
```

那么会有两个顶点重复了两次。

**索引缓冲对象**（Element Buffer Object）可以帮我们解决这个问题，他的工作方式是，给定一系列点和绘制他们的顺序，EBO会按照这个顺序来绘制图案。

```c++
GLfloat vertices[] = {
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
};

GLuint indices[] = { // 注意索引从0开始! 
    0, 1, 3, // 第一个三角形
    1, 2, 3  // 第二个三角形
};

GLuint EBO;
glGenBuffers(1, &EBO);

//初始化代码
glBindVertexArray(VAO);
//1.绑定顶点数组对象

glBindBuffer(GL_ARRAY_BUFFER, VBO);
//2.把我们的顶点数组复制到一个顶点缓冲中，供OpenGL使用

glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
//3.复制我们的索引数组到一个索引缓冲中，供OpenGL使用
//当目标是GL_ELEMENT_ARRAY_BUFFER的时候，VAO会储存glBindBuffer的函数调用。这也意味着它也会储存解绑调用（如果用glBindBuffer解绑，那解绑所用的默认值0会被VAO记录），所以确保你没有在解绑VAO之前解绑索引数组缓冲，否则它就没有这个EBO配置了
//不同的是，解绑GL_ARRAY_BUFFER就没有这个问题，因为他不是自动记录的

glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);


glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);
//4.设定顶点属性指针

glBindVertexArray(0);
//5. 解绑VAO（不是EBO！）

glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
//线框模式，只绘制边框不填充颜色

[...]

//绘制代码（游戏循环中）
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
//我们要绘制6个顶点，索引的类型是GL_UNSIGNED_INT，EBO中偏移量为0
//这个偏移量的单位为字节，假设要从第三个index开始绘制，那么这里应该写作(GLvoid*)(3*sizeof(GLuint))
glBindVertexArray(0);
```

包含EBO的VAO（图示）：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/04/vertex_array_objects_ebo.png)

## 着色器

着色器是运行在GPU上的小程序，它们服务于管线的某些特定部分。

着色器由GLSL写成，这种语言包含一些针对向量和矩阵的操作。

```glsl
#version version_number
//GLSL的开头总是要声明版本

in type in_variable_name;
in type in_variable_name;
//声明输入变量（顶点属性），正如前文所说，顶点属性的数量是有上限的，一般来说为16个包含4个分量的属性

out type out_variable_name;
//声明输出变量

uniform type uniform_name;
//uniform关键字用于声明全局变量，这种变量在着色器程序中必须是独一无二的
//uniform变量可以在同一个着色器程序的不同着色器中共享使用，但想要共享使用，需要在每个着色器中用同样的方法声明同样的变量
//对于着色器来说，uniform变量是只读的，uniform变量的设置只有两种方法：1.初始化的时候设置 2.CPU端设置，也就是通过OpenGL函数设置

int main()
{
  //main函数是程序唯一入口
  out_variable_name = weird_stuff_we_processed;
}
```

GLSL中的数据类型有：

* 基本数据类型：`int`，`float`，`double`，`uint`，`bool`。

* 容器类型：向量，矩阵

  | 类型    | 含义                    |
  | ------- | ----------------------- |
  | `vecn`  | 包含n个float分量的向量  |
  | `bvecn` | 包含n个bool分量的向量   |
  | `ivecn` | 包含n个int分量的向量    |
  | `uvecn` | 包含n个uint分量的向量   |
  | `dvecn` | 包含n个double分量的向量 |

  获取向量的分量，可以通过`xyzw`，`rgba`，`stpq`，也可以随机组合这些字母，组成新的向量，这种操作称为**重组**（Swizzling）。

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>
#include <math.h>

const GLuint WIDTH = 800, HEIGHT = 600;

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);

//下面的两个着色器代码中我们都声明了uniform变量ourColor，虽然顶点着色器我们没有用到这个变量，但他们是共享的
const char *vertexShaderSource = "#version 330 core\n"
                                "layout (location = 0) in vec3 position;\n"
                                "uniform vec4 ourColor;\n"
                                "void main()\n"
                                "{\n"
                                "   gl_Position = vec4(position.x, position.y, position.z, 1.0);\n"
                                "}\0";

const char *fragmentShaderSource = "#version 330 core\n"
                                  "out vec4 FragColor;\n"
                                  "uniform vec4 ourColor;\n"
                                  "void main()\n"
                                  "{\n"
                                  "   FragColor = ourColor;\n"
                                  "}\n\0";


int main() {
   glfwInit();
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "LearnOpenGL", nullptr, nullptr);
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);

   glewExperimental = GL_TRUE;
   glewInit();

   int width, height;
   glfwGetFramebufferSize(window, &width, &height);
   glViewport(0, 0, width, height);

   GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
   glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
   glCompileShader(vertexShader);

   GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
   glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
   glCompileShader(fragmentShader);

   GLuint shaderProgram = glCreateProgram();
   glAttachShader(shaderProgram, vertexShader);
   glAttachShader(shaderProgram, fragmentShader);
   glLinkProgram(shaderProgram);

   glDeleteShader(vertexShader);
   glDeleteShader(fragmentShader);

   GLfloat vertices[] = {
           0.5f,  0.5f, 0.0f,  // Top Right
           0.5f, -0.5f, 0.0f,  // Bottom Right
           -0.5f, -0.5f, 0.0f,  // Bottom Left
           -0.5f,  0.5f, 0.0f   // Top Left
   };

   GLuint indices[] = {
           0, 1, 3,
           1, 2, 3
   };

   GLuint VBO, VAO, EBO;

   glGenBuffers(1, &VBO);
   glGenBuffers(1, &EBO);
   glGenVertexArrays(1, &VAO);

   glBindVertexArray(VAO);
   glBindBuffer(GL_ARRAY_BUFFER, VBO);
   glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);

   glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
   glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*) 0);
   glEnableVertexAttribArray(0);

   glBindBuffer(GL_ARRAY_BUFFER, 0);

   glBindVertexArray(0);

   //glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);

   while (!glfwWindowShouldClose(window)){
       glfwPollEvents();
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT);


       glUseProgram(shaderProgram);
       GLfloat timeValue = glfwGetTime();
       //获取时间
       GLfloat greenValue = (sin(timeValue)/2)+0.5;
       GLint vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
       //在shaderProgram这个着色器程序中查找ourColor这个变量的索引值，后续我们需要通过这个索引值来更改这个变量的具体值，如果返回-1就代表没有找到这个变量
       glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
       //因为ourColor是一个四浮点分量的向量，所以我们通过对应函数去设置它的值
       //在查询之前，必须先激活程序
       glBindVertexArray(VAO);
       glDrawElements(GL_TRIANGLES, 3, GL_UNSIGNED_INT, (GLvoid*)0);
       glBindVertexArray(0);

       glfwSwapBuffers(window);
   }
   glDeleteVertexArrays(1, &VAO);
   glDeleteBuffers(1, &VBO);
   glDeleteBuffers(1, &EBO);

   glfwTerminate();
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){
   printf("%d\n", key);
   fflush(stdout);
   if (key == GLFW_KEY_ESCAPE && mode == GLFW_MOD_SHIFT && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);
}
```

### VAO补充

之前的VAO我们只用了一个变量指针，下面这个代码会展示怎么用两个：

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>
#include <math.h>

const GLuint WIDTH = 800, HEIGHT = 600;

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);

const char *vertexShaderSource = "#version 330 core\n"
                                "layout (location = 0) in vec3 position;\n"
                                "layout (location = 1) in vec3 color;\n"
                                "out vec3 ourColor;\n"
                                "void main()\n"
                                "{\n"
                                "   gl_Position = vec4(position.x, position.y, position.z, 1.0);\n"
                                "   ourColor = color;\n"
                                "}\0";

const char *fragmentShaderSource = "#version 330 core\n"
                                  "out vec4 FragColor;\n"
                                  "in vec3 ourColor;\n"
                                  "void main()\n"
                                  "{\n"
                                  "   FragColor = vec4(ourColor, 1.0f);\n"
                                  "}\n\0";


int main() {
   glfwInit();
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "LearnOpenGL", nullptr, nullptr);
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);

   glewExperimental = GL_TRUE;
   glewInit();

   int width, height;
   glfwGetFramebufferSize(window, &width, &height);
   glViewport(0, 0, width, height);

   GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
   glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
   glCompileShader(vertexShader);

   GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
   glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
   glCompileShader(fragmentShader);

   GLuint shaderProgram = glCreateProgram();
   glAttachShader(shaderProgram, vertexShader);
   glAttachShader(shaderProgram, fragmentShader);
   glLinkProgram(shaderProgram);

   glDeleteShader(vertexShader);
   glDeleteShader(fragmentShader);

   GLfloat vertices[] = {
           0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  // Bottom Right
           -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  // Bottom Left
           0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f   // Top
   };


   GLuint VBO, VAO;

   glGenBuffers(1, &VBO);
   glGenVertexArrays(1, &VAO);

   glBindVertexArray(VAO);
   glBindBuffer(GL_ARRAY_BUFFER, VBO);

   glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*) 0);
   glEnableVertexAttribArray(0);
    
    //下面把颜色值绑定到索引值为1的函数指针上
   glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*) (3 * sizeof(GLfloat)));
   glEnableVertexAttribArray(1);

   glBindBuffer(GL_ARRAY_BUFFER, 0);

   glBindVertexArray(0);

   //glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);

   while (!glfwWindowShouldClose(window)){
       glfwPollEvents();
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT);


       glUseProgram(shaderProgram);

       glBindVertexArray(VAO);
       glDrawArrays(GL_TRIANGLES, 0, 3);
       //最后输出的图片在片段着色器中进行了片段插值，也就是说对于一个不知道颜色的点，着色器会通过它附近的点的颜色来推测它的颜色
       glBindVertexArray(0);

       glfwSwapBuffers(window);
   }
   glDeleteVertexArrays(1, &VAO);
   glDeleteBuffers(1, &VBO);

   glfwTerminate();
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){
   printf("%d\n", key);
   fflush(stdout);
   if (key == GLFW_KEY_ESCAPE && mode == GLFW_MOD_SHIFT && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);
}
```

为了简化构建着色器程序的流程以及规范着色器代码，我们用以下代码封装一下着色器程序：

[Code Viewer. Source code: headers/shader (learnopengl.com)](https://learnopengl.com/code_viewer.php?type=header&code=shader)

## 纹理

纹理是一张2D贴图，使用纹理可以在不增加额外顶点数量的同时，给物体加入更多的细节。

这是下面要用到的纹理贴图：

https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/wall.jpg

纹理的工作原理是，对每一个顶点，我们都指定它对应的纹理坐标位置，对于这组顶点定义的片元，着色器会根据不同模式对其插值。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/tex_coords.png)

纹理坐标的范围是`[0,0]-[1,1]`，我们总是把贴图平铺到这个范围的正方形，不同的插值方式定义了超出纹理坐标范围的图像该怎么绘制：

| 环绕方式             |
| -------------------- |
| `GL_REPEAT`          |
| `GL_MIRRORED_REPEAT` |
| `GL_CLAMP_TO_EDGE`   |
| `GL_CLAMP_TO_BORDED` |

具体表现请看图：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/texture_wrapping.png)

对于纹理的s轴和t轴，我们可以定义不同的环绕方式：

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
//这个是一个2D纹理，S轴和T轴都采用镜像循环的环绕方式

float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
//这是一个2D纹理，如果要用GL_CLAMP_TO_BORDED环绕方式，那么同时需要指定超出范围的材质用什么颜色
//在OpenGL中，函数后缀通常包含了这个函数接受的参数的类型：
//          |后缀        |含义                    |
//          |f          |函数需要一个float         |
//          |i          |函数需要一个int           |
//          |ui         |函数需要一个unsigned int  |
//          |3f         |函数需要一个3个float       |
//          |fv         |函数需要一个float向量或数组 |
```

因为纹理是一张图，他的分辨率就一定有上限，对于如何把纹理像素转换成具体每个纹理坐标的颜色（想象一下PS里吸取某个点颜色的操作），这个工作叫做**纹理过滤**（Texture Filtering）。

1. 近邻过滤（Nearest Neighbor Filtering），选取最近的一个像素作为这个点的颜色。
2. 线性过滤（Bilinear Filtering），选取最近的几个像素计算一个插值作为这个点的像素。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/texture_filtering.png)

当纹理被放大和缩小（应用到不同体积的物体）的时候，我们可以选用不同的过滤方式：

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
//纹理被缩小（Minify），用近邻过滤
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
//纹理被放大（Magnify），用线性过滤
```

如果离物体比较远，但是使用的纹理贴图还是一样的，就会出现两个问题：

1. 纹理过滤时过滤了大部分颜色，最后可能就剩下一两个颜色，这样物体看起来就不真实了。
2. 远距离的物体用很高清的纹理，会徒增显存压力。

OpenGL使用**多级渐远纹理**（Mipmap）来处理这个问题，这是一组纹理的集合，后一个纹理图像总是为前一个纹理图案的一半大，当观察者距离超过一定阈值的时候，OpenGL会切换它所使用的纹理。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/mipmaps.png)

OpenGL有一个`glGenerateMipmaps`函数来帮助生成多级渐远纹理，和普通的纹理一样，多级渐远纹理也有纹理过滤：

| 过滤方式                  | 描述                                       |
| ------------------------- | ------------------------------------------ |
| GL_NEAREST_MIPMAP_NEAREST | 用最匹配的一张纹理，并使用邻近过滤计算颜色 |
| GL_LINEAR_MIPMAP_NEAREST  | 用最匹配的一张纹理，并使用线性过滤计算颜色 |
| GL_NEAREST_MIPMAP_LINEAR  | 用多张接近的纹理，使用邻近过滤计算颜色     |
| GL_LINEAR_MIPMAP_LINEAR   | 用多张接近的纹理，并使用线性过滤计算颜色   |

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
//特别注意此处，因为多级渐远纹理是用来处理纹理被缩小的情况的，对于放大，我们不用多级渐远纹理的过滤方式，如果用了会报错
```

使用纹理的第一件事肯定是要把对应的纹理图案加载到程序中，SOIL是简易OpenGL图像库，它就支持大多数流行的图像格式。因为SOIL的官网垮掉了，所以这里提供一个Github的下载链接：https://github.com/kbranigan/Simple-OpenGL-Image-Library

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>
#include <SOIL.h>
#include "Shader.h"

const GLuint WIDTH = 800, HEIGHT = 600;

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);

int main() {
   glfwInit();
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "LearnOpenGL", nullptr, nullptr);
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);

   glewExperimental = GL_TRUE;
   glewInit();

   glViewport(0, 0, WIDTH, HEIGHT);

   Shader myShader("vertex shader path", "fragment shader path");

   GLfloat vertices[] = {
           // Positions          // Colors           // Texture Coords
           0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f, // Top Right
           0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f, // Bottom Right
           -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f, // Bottom Left
           -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f  // Top Left
   };

   GLuint indices[] = {
           0, 1, 3,
           1, 2, 3
   };


   GLuint VBO, VAO, EBO;

   glGenBuffers(1, &VBO);
   glGenBuffers(1, &EBO);
   glGenVertexArrays(1, &VAO);

   glBindVertexArray(VAO);
   glBindBuffer(GL_ARRAY_BUFFER, VBO);
   glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);

   glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
   glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(GLfloat), (GLvoid*) 0);
   glEnableVertexAttribArray(0);

   glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(GLfloat), (GLvoid*) (3 * sizeof(GLfloat)));
   glEnableVertexAttribArray(1);

   glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(GLfloat), (GLvoid*) (6 * sizeof(GLfloat)));
   glEnableVertexAttribArray(2);
    //将纹理坐标绑定到VAO的第三个顶点属性指针上

   glBindBuffer(GL_ARRAY_BUFFER, 0);

   glBindVertexArray(0);

   GLuint texture;
   glGenTextures(1, &texture);
   glBindTexture(GL_TEXTURE_2D, texture);
    //绑定纹理

   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);

   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

   int width, height;
   unsigned char *image = SOIL_load_image("texture path", &width, &height, 0, SOIL_LOAD_RGB);
    //加载贴图，第二个和第三个参数会返回贴图的宽和高，第四个参数返回图片的通道数，最后一个参数告诉SOIL我们只关心图片的RGB值，函数返回值是一个很大的数组
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, image);
    //第一个参数指定了纹理目标，之前绑定了纹理对象到GL_TEXTURE_2D上，那么和其他绑定的原理一样，对GL_TEXTURE_2D的配置都会应用到这个对象上
    //第二个参数为纹理指定多级纹理的级别，这里设置0，表示基本级别
    //第三个参数表示纹理的存储格式，我们只有RGB，所以存为GL_RGB
    //第四和第五参数表示纹理的宽高
    //第六个参数总是被设置为0，不需要纠结它
    //第七第八个参数指定了源图的格式和数组的数据类型，我们用RGB加载的原图，用char（unsigned byte）存储的原图
    //最后一个是真正的图像数据
   glGenerateMipmap(GL_TEXTURE_2D);
    //因为之前没有加载多级渐远纹理，所以在生成纹理后，我们需要手动调用并生成，这会为当前绑定的纹理自动生成所有需要的多级渐远纹理
    //请注意有个长得非常像的函数叫glGenerateTextureMipmap，两个函数的功能都是一样的，但是参数不同
   SOIL_free_image_data(image);
   glBindTexture(GL_TEXTURE_2D, 0);

   while (!glfwWindowShouldClose(window)){
       glfwPollEvents();
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT);


       glBindTexture(GL_TEXTURE_2D, texture);
       myShader.Use();

       glBindVertexArray(VAO);
       glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
       glBindVertexArray(0);

       glfwSwapBuffers(window);
   }
   glDeleteVertexArrays(1, &VAO);
   glDeleteBuffers(1, &VBO);
   glDeleteBuffers(1, &EBO);

   glfwTerminate();
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){
   printf("%d\n", key);
   fflush(stdout);
   if (key == GLFW_KEY_ESCAPE && mode == GLFW_MOD_SHIFT && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);
}
```

顶点着色器：

```glsl
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 color;
layout (location = 2) in vec2 texCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(position, 1.0f);
    ourColor = color;
    TexCoord = texCoord;
}
```

片段着色器：

```glsl
#version 330 core
in vec3 ourColor;
in vec2 TexCoord;

out vec4 color;

uniform sampler2D ourTexture;
//sample被称为采样器，它以贴图的纹理类型作为后缀
//采样器会和glDrawElements沟通，调用glDrawElements之后，它会把纹理赋值给采样器
//sampler2D被称为不透明类型(Opaque Type)，这是一种不能被实例化的类型，我们只能通过uniform来定义它，如果不用uniform，GLSL会报错
//不透明类型指的是那些内部形式没有被暴露出来的类型
void main()
{
    color = texture(ourTexture, TexCoord);
    //texture函数用于采样纹理的颜色，第一个参数是一个采样器，第二个参数是对应的纹理坐标
}
```

使用多个纹理贴图：

```c++
myShader.Use();

//glActiveTexture(GL_TEXTURE0);
//在绑定纹理单元到对应采样器之前，需要先激活对应的纹理单元，这点和VAO要激活顶点属性类似
//GL_TEXTURE0是默认激活的，所以不用特别说明也行
glBindTexture(GL_TEXTURE_2D, texture);
glUniform1i(glGetUniformLocation(myShader.Program, "ourTexture"), 0);
//使用这个函数，我们会给对应的纹理采样器分配一个索引值，这个索引值被称为纹理单元（Texture Unit），纹理单元和glGetUniformLocation返回的索引值，顶点属性的索引值（VAO指针索引值）是不同的索引值，不会冲突
//glGetUniformLocation返回的是一个指出着色器程序中这个变量位置的值，我们通过这个值，给对应的纹理采样器一个纹理单元，上限是16，它们的定义是连续的，也就是说GL_TEXTURE0+1==GL_TEXTURE1
//虽然纹理单元是被连续定义的，但我们使用的时候并没有被要求一定要把采样器按顺序绑定纹理单元，也就是说绑定完0后，不一定要绑定1
//一个纹理的默认纹理单元是0（所以其实注释掉这段代码也没问题），如果不特别指定，所有的纹理的纹理单元都会是0，所以在多贴图的时候要重新指定纹理单元

glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);
glUniform1i(glGetUniformLocation(myShader.Program, "ourTexture2"), 1);

glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
glBindVertexArray(0);
```

对应的片段着色器

```glsl
#version 330 core
...

uniform sampler2D ourTexture1;
uniform sampler2D ourTexture2;

void main()
{
    color = mix(texture(ourTexture1, TexCoord), texture(ourTexture2, TexCoord), 0.2);
    //mix函数会混合两个值，第三个值表示它们的混合比例
}
```

## 变换

一个向量包含一个**方向**（Direction）和**大小**（Magnitude），起点为原点的向量被称作**位置向量**（Position Vector），向量的运算包括：

* 向量加**标量**（Scalar）

* 向量取反

* 向量加减

* 向量求长度（用标准化算**单位向量**（Unit Vector））

* 向量乘积

  * 点乘：$\vec v \cdot\vec k=\Vert\vec v\Vert\cdot\Vert\vec k\Vert\cdot\cos\theta$，或$\sum _{i=0}^n v_i\times k_i$，点乘可以用来测试两个向量是否垂直或平行

  * 叉乘：两个向量的叉乘结果会垂直于这两个向量
    $$
    \left(
    \begin{array}{c}
    A_x \\
    A_y \\
    A_z
    \end{array}
    \right)
    \times
    \left(
    \begin{array}{c}
    B_x \\
    B_y \\
    B_z
    \end{array}
    \right)
    =
    \left(
    \begin{array}{c}
    A_y\cdot B_z - A_z\cdot B_y \\
    A_z\cdot B_x - A_x\cdot B_z   \\
    A_x\cdot B_y - A_y\cdot B_x
    \end{array}
    \right)
    $$

矩阵的运算包括：

* 矩阵加标量

* 矩阵乘标量

* 矩阵相乘：

  ![Matrix Multiplication](https://learnopengl-cn.github.io/img/01/07/matrix_multiplication.png)

* 单位矩阵

上面的都是基础中的基础，比较重要的三个变换操作是缩放，位移和旋转。

要看懂这三个操作，首先要弄清变换矩阵右下角这个常量的意义。在前面提到过，一个有四个分量的向量可以被表示为xyzw，这个w就是这个向量，也叫做齐次坐标。如果没有齐次坐标，我们就无法位移坐标。所以如果齐次坐标为0，这个向量就代表一个方向向量。如果不是方向向量，那它的齐次坐标就是1。

缩放
$$
\left[
\begin{array}{cccc}
S_1 & 0   & 0   & 0\\
0   & S_2 & 0   & 0\\
0   & 0   & S_3 & 0\\
0   & 0   & 0   & 1
\end{array}
\right]
\cdot
\left(
\begin{array}{c}
x \\
y \\
z \\
1
\end{array}
\right)
=
\left(
\begin{array}{c}
S_1 \\
S_2 \\
S_3 \\
1
\end{array}
\right)
$$
如果**缩放因子**（Scaling Factor）都一样，那么这个缩放操作就是均匀缩放。

位移
$$
\left[
\begin{array}{cccc}
1   & 0   & 0   & T_x\\
0   & 1   & 0   & T_y\\
0   & 0   & 1   & T_z\\
0   & 0   & 0   & 1
\end{array}
\right]
\cdot
\left(
\begin{array}{c}
x \\
y \\
z \\
1
\end{array}
\right)
=
\left(
\begin{array}{c}
x + T_x \\
y + T_y \\
z + T_z \\
1
\end{array}
\right)
$$


旋转

> 弧度转角度：$角度=弧度\times (180/\pi)$
>
> 角度转弧度：$弧度=角度\times (\pi/180)$

沿x轴旋转
$$
\left[
\begin{array}{cccc}
1   & 0   & 0   & 0\\
0   & \cos\theta& -\sin\theta   & 0\\
0   & \sin\theta   & \cos\theta   & 0\\
0   & 0   & 0   & 1
\end{array}
\right]
\cdot
\left(
\begin{array}{c}
x \\
y \\
z \\
1
\end{array}
\right)
=
\left(
\begin{array}{c}
x \\
\cos\theta \cdot y-\sin\theta\cdot z \\
\sin\theta \cdot y+\cos\theta\cdot z \\
1
\end{array}
\right)
$$
沿y轴旋转
$$
\left[
\begin{array}{cccc}
\cos\theta & 0   & \sin\theta   & 0\\
0   & 1 &  0  & 0\\
-\sin\theta   &  0  & \cos\theta   & 0\\
0   & 0   & 0   & 1
\end{array}
\right]
\cdot
\left(
\begin{array}{c}
x \\
y \\
z \\
1
\end{array}
\right)
=
\left(
\begin{array}{c}
\cos\theta \cdot x +\sin\theta\cdot z\\
 y \\
-\sin\theta \cdot x +\cos\theta\cdot z \\
1
\end{array}
\right)
$$
沿z轴旋转
$$
\left[
\begin{array}{cccc}
\cos\theta & -\sin\theta   & 0   & 0\\
\sin\theta   & \cos\theta &  0  & 0\\
0   &  0  &  1  & 0\\
0   & 0   & 0   & 1
\end{array}
\right]
\cdot
\left(
\begin{array}{c}
x \\
y \\
z \\
1
\end{array}
\right)
=
\left(
\begin{array}{c}
\cos\theta \cdot x -\sin\theta\cdot y\\
\sin\theta \cdot x +\cos\theta\cdot y \\
z \\
1
\end{array}
\right)
$$
了解这些运算后，如何将其运用到代码中呢，首先是要装好矩阵运算需要的库`GLM`，下载链接如下：https://github.com/g-truc/glm/tags

```c++
glm::mat4 trans = glm::mat4(1.0f)
//声明一个矩阵，并初始化为（1.0，1.0，1.0，1.0）
trans = glm::translate(trans, glm::vec3(1.0f, 1.0f, 0.0f));
//平移
trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
//围绕glm::vec3(0.0, 0.0, 1.0)旋转90度，因为这个函数接受的是弧度制，所以需要转换一下
trans = glm::scale(trans, glm::vec3(0.5, 0.5, 0.5));
//缩放
unsigned int transformLoc = glGetUniformLocation(ourShader.ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
//我们要设置一个4行4列的矩阵，那么后缀是Matrix4，fv代表函数想要一个矩阵作为参数
//第二个参数表示我们需要传送几个矩阵
//第三个参数询问我们是否要转置矩阵，在线性代数中，我们用的是行主序，但是在OpenGL中我们用的是列主序矩阵，幸运的是GLM默认布局也是列主序，所以此处不需要转置
//第四个参数表示真正的矩阵数据，因为GLM的数据并不是OpenGL接受的那种，所以需要用value_ptr转换一下
```

对应的顶点着色器：

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

uniform mat4 transform;

void main()
{
    gl_Position = transform * vec4(aPos, 1.0f);
    //转换矩阵的应用是左乘
    TexCoord = vec2(aTexCoord.x, 1.0 - aTexCoord.y);
}
```

## 坐标系统

这一章节会讨论五个不同的坐标系统：

1. 局部空间（Local Space），相对一个局部原点的坐标。
2. 世界空间（World Space），相对一个全局原点的坐标。
3. 观察空间（View Space），每个坐标都是从摄像机或者观察者角度进行观察的。
4. 裁剪空间（Clip Space），裁剪坐标的范围是`[-1,1]`，在这之外的点会被裁剪。
5. 屏幕空间（Screen Space），视口变换会把裁剪坐标变换到由glViewport函数所定义的坐标范围内。

从上到下，是一个顶点在被转换成片段之前要经历的所有不同状态。

要将一个空间转换到另一个空间，需要用到几个重要的变换矩阵：**模型**（Model），**观察**（View），**投影**（Projection），下图（其中观察空间的描述不够恰当，可以想象观察空间是以摄像机为原点构建的一个坐标系）展示了整个流程以及各个变换过程做了什么：

![coordinate_systems](https://learnopengl-cn.github.io/img/01/08/coordinate_systems.png)

裁切空间中，如何判断什么应该被裁切掉是很重要的一点，为此我们需要定义一个**投影矩阵**，它定义了一个范围值，在这个范围内的坐标会被标准化到`[-1.0, 1.0]`，超出范围的会被裁减。如果某个物体只有一部分超出了**剪裁体积**（Clipping Volumn），那么OpenGL会为此重建一个会多个三角形（图元）让其能够适合这个范围。

由投影矩阵描绘的观察箱被称为**平截头体**（Frustum），在平截头体范围内的坐标最终会出现在屏幕上，这个坐标转换过程被称为投影。这个转换过程有两种不同的形式，每种都定义了不同的平截头体。

正射投影（Orthographic Projection Matrix）

![orthographic projection frustum](https://learnopengl-cn.github.io/img/01/08/orthographic_frustum.png)

正交投影的平截头体是一个立方体，平截头体由宽，高，近平面，远平面指定。

GLM的内置函数提供了创建正交矩阵的方法：

```c++
glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
//前两个参数指定了平截头体的左右坐标（width）
//第三第四个参数指定了平截头题的底部和顶部坐标（height）
//第五第六个参数定义了近平面和远平面的坐标
```

透视投影（Perspective Projection Matrix）

透视投影因为考虑了近大远小的关系，看起来会更贴近真实。

![ perspective_frustum](https://learnopengl-cn.github.io/img/01/08/perspective_frustum.png)

在描述透视投影之前，我想先理清一下为什么OpenGL表示一个三维的点要用一个4分量的向量，以及最后一个向量w的意义是什么，我们从三个方向分析：

1. 在一个坐标系中，向量和点的表示方式完全是一模一样的，那怎么区分向量和点呢？这时候w就可以派上用场了，当w为1的时候，它表示一个点，当w为0的时候，它表示一个向量。这样表示有个很巧妙地地方，我们知道一个向量是两个点做减法得出的，当两个点相减后，w恰好为0，我们又指定一个向量加一个点可以表示这个点经过这个向量能到达的终点，这是w恰好又为1。

2. 虽然从小的数学教育告诉我们，同一个平面的两条平行直线不会相交，但是从视觉角度来说，这个定理是不成立的，比如下图这种情况：

   ![img](https://img-blog.csdnimg.cn/20190709195103643.png)

   可以看到两条铁路在很远的地方是相交的，那我们应该怎么表示这种情况呢，同样，也是引入齐次坐标w。在有的教学中认为距离越远，w越小，但是我觉得应该是距离越远，w越大。这里用w转换坐标到笛卡尔坐标系（也就是裁剪空间），即$(x,y,z,w)\rightarrow(\frac{x}{w},\frac{y}{w},\frac{z}{w})$，如果w足够大，那么任何点都会汇聚在原点。在透视投影中如何理解w的存在呢，可以想象w是沿着z轴上运动的一个点（要注意z轴方向是从屏幕出发指向我们），随着点的远离，w会增大。
   
3. 对于缩放和旋转，我们都可以用3乘3的变换矩阵来做乘法运算，但是平移加法运算的结果，有没有办法可以让平移也做乘法运算呢？答案是升维，正如之前给出的公式，在四维空间中，三维的点的平移也可以是乘法运算。

了解了w的意义之后，那么再来看看w的应用，在透视投影中，任何笛卡尔坐标（$(x,y,z,w)\rightarrow(\frac{x}{w},\frac{y}{w},\frac{z}{w})$）在`[-1,1]`范围外的点都会被裁切。关于透视投影矩阵的证明过程在我的Unity记录里有记载，欢迎查阅。

GLM的内置函数提供了创建透视矩阵的方法：

```c++
glm::mat4 proj = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
//第一个参数定义了FOV，即视野大小，一般来说45度比较真实
//第二个参数定义了视口的宽高比，比如上面的铁路图就比较接近1：1
//第三第四个参数定义了平截头体近平面和远平面的坐标
```

学会了投影的方法之后，空间转换的前四步就都不是问题了，至于最后一步，回想一下代码中`glViewport`这个函数，它帮我们干了最后一件事，视口转换。用线代知识总结一下前四步的过程：
$$
V_{clip}=M_{projection}\cdot M_{view}\cdot M_{model}\cdot V_{local}
$$
这里帮大家回忆一下一个知识点，对某个矩阵进行一个操作，等于这个矩阵*左乘*这个操作对应的变换矩阵。

下面是一个简单的3D物体渲染的代码，请注意其中关于Z缓冲的解释：

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>
#include <SOIL.h>
#include "Shader.h"
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
const GLuint WIDTH = 800, HEIGHT = 600;

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);

using namespace glm;

int main() {
   glfwInit();
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "Hengine", nullptr, nullptr);
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);

   glewExperimental = GL_TRUE;
   glewInit();

   glViewport(0, 0, WIDTH, HEIGHT);

   Shader myShader("/home/herain/Documents/opengl/Shader/texture/shader.vertex", "/home/herain/Documents/opengl/Shader/texture/shader.fragment");

   float vertices[] = {
           -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,
           0.5f, -0.5f, -0.5f,  1.0f, 0.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,

           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
           -0.5f,  0.5f,  0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,

           -0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           -0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           -0.5f,  0.5f,  0.5f,  1.0f, 0.0f,

           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,

           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           0.5f, -0.5f, -0.5f,  1.0f, 1.0f,
           0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
           0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,

           -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           -0.5f,  0.5f,  0.5f,  0.0f, 0.0f,
           -0.5f,  0.5f, -0.5f,  0.0f, 1.0f
   };


   GLuint VBO, VAO;

   glGenBuffers(1, &VBO);
   glGenVertexArrays(1, &VAO);

   glBindVertexArray(VAO);
   glBindBuffer(GL_ARRAY_BUFFER, VBO);

   glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat), (GLvoid*) 0);
   glEnableVertexAttribArray(0);

   glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat), (GLvoid*) (3 * sizeof(GLfloat)));
   glEnableVertexAttribArray(1);

   glBindBuffer(GL_ARRAY_BUFFER, 0);

   glBindVertexArray(0);

   GLuint texture, texture2;
   int width, height;
   glGenTextures(1, &texture);
   glGenTextures(1, &texture2);

   glBindTexture(GL_TEXTURE_2D, texture);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   unsigned char *image = SOIL_load_image("/home/herain/Documents/opengl/Shader/texture/ti.png", &width, &height, 0, SOIL_LOAD_RGB);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, image);
   glGenerateMipmap(GL_TEXTURE_2D);
   SOIL_free_image_data(image);

   glBindTexture(GL_TEXTURE_2D, texture2);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   unsigned char *image2 = SOIL_load_image("/home/herain/Documents/opengl/Shader/texture/container.jpg", &width, &height, 0, SOIL_LOAD_RGB);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, image2);
   glGenerateMipmap(GL_TEXTURE_2D);
   SOIL_free_image_data(image2);

   glBindTexture(GL_TEXTURE_2D, 0);

   glEnable(GL_DEPTH_TEST);
    //启用深度缓冲，OpenGL将所有深度信息都存在一个Z缓冲里，这个缓冲是GLFW自动生成的
    //对于每个片段，OpenGL会把它的深度值和z缓冲的深度值进行比较，如果这个片段的深度值在缓冲的深度值后面，就会被丢弃，不然就覆盖，最后只有深度值等于缓冲里深度值的片段会被渲染。
    //一般来说深度测试时关闭的，需要手动打开


   while (!glfwWindowShouldClose(window)){
       glfwPollEvents();
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
       //上次循环的深度值需要被清除，这个不用多说


       myShader.Use();

       mat4 model = mat4(1.0f);
       model = rotate(model, (float)glfwGetTime(), vec3(0.5f, 1.0f, 0.0f));

       mat4 view = mat4(1.0f);
       view = translate(view, vec3(0.0f, 0.0f, -3.0f));
       //解释一下这个操作，在着色器中我们是对物体进行矩阵操作，摄像头往后退，相对来说等于物体远离摄像头，那么如果摄像头为定点，那我们相对移动物体就行了

       mat4 projection = mat4(1.0f);
       projection = perspective(radians(45.0f), (float)WIDTH/(float)HEIGHT, 0.1f, 100.0f);

       int modelLoc = glGetUniformLocation(myShader.Program, "model");
       glUniformMatrix4fv(modelLoc, 1, GL_FALSE, value_ptr(model));

       int viewLoc = glGetUniformLocation(myShader.Program, "view");
       glUniformMatrix4fv(viewLoc, 1, GL_FALSE, value_ptr(view));

       int persLoc = glGetUniformLocation(myShader.Program, "projection");
       glUniformMatrix4fv(persLoc, 1, GL_FALSE, value_ptr(projection));

       //glActiveTexture(GL_TEXTURE0);
       glBindTexture(GL_TEXTURE_2D, texture);
       //glUniform1i(glGetUniformLocation(myShader.Program, "ourTexture"), 0);

       glActiveTexture(GL_TEXTURE1);
       glBindTexture(GL_TEXTURE_2D, texture2);
       glUniform1i(glGetUniformLocation(myShader.Program, "ourTexture2"), 1);

       glBindVertexArray(VAO);
       glDrawArrays(GL_TRIANGLES, 0, 36);
       glBindVertexArray(0);

       glfwSwapBuffers(window);
   }
   glDeleteVertexArrays(1, &VAO);
   glDeleteBuffers(1, &VBO);

   glfwTerminate();
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){
   printf("%d\n", key);
   fflush(stdout);
   if (key == GLFW_KEY_ESCAPE && mode == GLFW_MOD_SHIFT && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);
}
```

## 摄像机

在之前的例子中，我们没有处理摄像机的位置，而是让场景本身移动，然而这样操作是反常识的。那么如何构建摄像机呢？首先要构建以摄像机为原点的坐标系，一个坐标系包括原点和前，右，上三个轴。

原点位置

原点位置很容易获取，想让摄像机在哪，哪里就是摄像机位置。

**前轴**

要注意一点，摄像机指向方向和之前提到过的坐标系方向一样，是-z方向，而此处我们要获取的前轴方向是+z方向，所以是摄像机位置减去目标位置。

**右轴**

计算右轴方向，我们可以定义一个上向量，用叉乘算出答案。

**上轴**

利用刚算出来的右轴叉乘前轴，也可以简单算出答案。

有了三个轴和原点的信息，我们就可以利用它们构造`LookAt` 矩阵了：
$$
LookAt=
\left[\begin{array}{cccc}
R_x & R_y & R_z & 0 \\
U_x & U_y & U_z & 0 \\
D_x & D_y & D_z & 0 \\
0   & 0   & 0   & 0 \\
\end{array}\right]
\cdot
\left[\begin{array}{cccc}
1   & 0   & 0   & -P_x \\
0   & 1   & 0   & -P_y \\
0   & 0   & 1   & -P_z \\
0   & 0   & 0   & 1    \\
\end{array}\right]
$$
其中，R是右向量，U是上向量，D是方向向量，P是摄像机位置。为什么位置要取反呢，因为原理上来说，LookAt函数是通过移动世界坐标到相反的方向来模拟摄像机运动的。

大致了解摄像机位置的原理之后，我们继续了解一下摄像机角度，表示旋转我们使用欧拉角：

![img](http://www.learnopengl.com/img/getting-started/camera_pitch_yaw_roll.png)

在摄像机系统中，我们只需考虑俯仰角（pitch）和偏航角（yaw）。

请注意两点：

* 正如图中所示，随着数值（指pitch，yaw，roll）增加，物体呈顺时针旋转，此时实际角度减小。
* 当pitch和yaw都为0时，物体默认看向+z轴方向。

下面给出的代码请注意看：

1. 怎么绑定鼠标监听函数
2. 如何设置摄像机角度以及为何这么设置
3. yaw变量的初始值
4. LookAt函数的设置
5. delta时间的解释

```c++
#include <GL/glew.h>
#include <stdio.h>
#include <GLFW/glfw3.h>
#include <SOIL.h>
#include "Shader.h"
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
const GLuint WIDTH = 800, HEIGHT = 600;

void mouse_callback(GLFWwindow* window, double xpos, double ypos);
void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode);
void scrool_callback(GLFWwindow* window, double xoffset, double yoffset);
void pos_update();

using namespace glm;

vec3 cameraPos = vec3(0.0f, 0.0f, 3.0f);
vec3 cameraFront = vec3(0.0f, 0.0f, -1.0f);
vec3 cameraUp = vec3(0.0f, 1.0f, 0.0f);

bool keys[1024];
GLfloat currentFrame = 0.0f;
GLfloat lastFrame = 0.0f;
GLfloat deltaFrame = 0.0f;

GLfloat gPitch = 0.0f;
GLfloat gYaw = 180.0f;
//请注意这里的yaw初始值是180，因为默认0的时候摄像机是看向+z方向的，而我们需要摄像机看向-z方向，所以逆时针转180度
GLfloat fov = 45.0f;
GLfloat lastX = WIDTH/2.0;
GLfloat lastY = HEIGHT/2.0;

bool initMouse = true;

int main() {
   glfwInit();
   glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
   glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
   glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
   glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

   GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "Hengine", nullptr, nullptr);
   glfwMakeContextCurrent(window);
   glfwSetKeyCallback(window, key_callback);
   glfwSetCursorPosCallback(window, mouse_callback);
    //绑定鼠标监听函数
   glfwSetScrollCallback(window, scrool_callback);
    //绑定滚轮监听函数

   glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
    //设置光标，让光标不可见

   glewExperimental = GL_TRUE;
   glewInit();

   glViewport(0, 0, WIDTH, HEIGHT);

   Shader myShader("/home/herain/Documents/opengl/Shader/texture/shader.vertex", "/home/herain/Documents/opengl/Shader/texture/shader.fragment");

   float vertices[] = {
           -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,
           0.5f, -0.5f, -0.5f,  1.0f, 0.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,

           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
           -0.5f,  0.5f,  0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,

           -0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           -0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           -0.5f,  0.5f,  0.5f,  1.0f, 0.0f,

           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,

           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
           0.5f, -0.5f, -0.5f,  1.0f, 1.0f,
           0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
           0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
           -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
           -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,

           -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
           0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
           -0.5f,  0.5f,  0.5f,  0.0f, 0.0f,
           -0.5f,  0.5f, -0.5f,  0.0f, 1.0f
   };

   glm::vec3 cubePositions[] = {
           glm::vec3( 0.0f,  0.0f,  0.0f),
           glm::vec3( 2.0f,  5.0f, -15.0f),
           glm::vec3(-1.5f, -2.2f, -2.5f),
           glm::vec3(-3.8f, -2.0f, -12.3f),
           glm::vec3( 2.4f, -0.4f, -3.5f),
           glm::vec3(-1.7f,  3.0f, -7.5f),
           glm::vec3( 1.3f, -2.0f, -2.5f),
           glm::vec3( 1.5f,  2.0f, -2.5f),
           glm::vec3( 1.5f,  0.2f, -1.5f),
           glm::vec3(-1.3f,  1.0f, -1.5f)
   };


   GLuint VBO, VAO;

   glGenBuffers(1, &VBO);
   glGenVertexArrays(1, &VAO);

   glBindVertexArray(VAO);
   glBindBuffer(GL_ARRAY_BUFFER, VBO);

   glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat), (GLvoid*) 0);
   glEnableVertexAttribArray(0);

   glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat), (GLvoid*) (3 * sizeof(GLfloat)));
   glEnableVertexAttribArray(1);

   glBindBuffer(GL_ARRAY_BUFFER, 0);

   glBindVertexArray(0);

   GLuint texture, texture2;
   int width, height;
   glGenTextures(1, &texture);
   glGenTextures(1, &texture2);

   glBindTexture(GL_TEXTURE_2D, texture);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   unsigned char *image = SOIL_load_image("/home/herain/Documents/opengl/Shader/texture/ti.png", &width, &height, 0, SOIL_LOAD_RGB);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, image);
   glGenerateMipmap(GL_TEXTURE_2D);
   SOIL_free_image_data(image);

   glBindTexture(GL_TEXTURE_2D, texture2);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
   glTextureParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   unsigned char *image2 = SOIL_load_image("/home/herain/Documents/opengl/Shader/texture/container.jpg", &width, &height, 0, SOIL_LOAD_RGB);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, image2);
   glGenerateMipmap(GL_TEXTURE_2D);
   SOIL_free_image_data(image2);

   glBindTexture(GL_TEXTURE_2D, 0);

   glEnable(GL_DEPTH_TEST);

   while (!glfwWindowShouldClose(window)){
       currentFrame = glfwGetTime();
       deltaFrame = currentFrame - lastFrame;
       lastFrame = currentFrame;
       //为什么这里要计算一个delta时间？因为在不同机器上对一个图形的渲染速度是不同的，想象我们有两台机子，一台是现在顶尖配置一台是10年前最拉的配置，如果我们在两台机子上渲染同一个图像然后运动，第一台电脑渲染速度快，帧之间间隔小，那么单位时间内收到的运动指令就更多，相反，10年前电脑可能就受到几个指令，那么好电脑在观感上物体就运动更多，差的电脑运动的就更少
       //但如果我们记录渲染需要的时间，乘以这个时间就能保证两个电脑上的物体运动速度一样（但后者帧率肯定惨不忍睹）
       
       glfwPollEvents();
       pos_update();
       glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
       glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

       myShader.Use();



       mat4 view = mat4(1.0f);
       view = lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
       //第一个函数时相机位置，第二个函数是目标位置，第三个函数是世界坐标的朝上方向，虽然这里写的意思像相机朝上方向，但实际上是世界坐标！

       mat4 projection = mat4(1.0f);
       projection = perspective(radians(fov), (float)WIDTH/(float)HEIGHT, 0.1f, 100.0f);

       myShader.setMat4("view", view);
       myShader.setMat4("projection", projection);

       //glActiveTexture(GL_TEXTURE0);
       glBindTexture(GL_TEXTURE_2D, texture);
       //glUniform1i(glGetUniformLocation(myShader.Program, "ourTexture"), 0);

       glActiveTexture(GL_TEXTURE1);
       glBindTexture(GL_TEXTURE_2D, texture2);
       glUniform1i(glGetUniformLocation(myShader.Program, "ourTexture2"), 1);

       glBindVertexArray(VAO);
       for(unsigned int i = 0; i < 10; i ++){
           mat4 model = mat4(1.0f);
           model = translate(model, cubePositions[i]);
           float angle = 20.0f * i;
           model = rotate(model, radians(angle), vec3(1.0f, 0.3f, 0.5f));
           myShader.setMat4("model", model);

           glDrawArrays(GL_TRIANGLES, 0, 36);
       }
       glBindVertexArray(0);

       glfwSwapBuffers(window);
   }
   glDeleteVertexArrays(1, &VAO);
   glDeleteBuffers(1, &VBO);

   glfwTerminate();
   return 0;
}

void key_callback(GLFWwindow *window, int key, int scancode, int action, int mode){

   if (key == GLFW_KEY_ESCAPE && mode == GLFW_MOD_SHIFT && action == GLFW_PRESS)
       glfwSetWindowShouldClose(window, GL_TRUE);

   if(action == GLFW_PRESS){
       keys[key] = true;
   }  else if (action == GLFW_RELEASE){
       keys[key] = false;
   }
}

void pos_update(){
   GLfloat cameraSpeed = 5.0f * deltaFrame;
   if(keys[GLFW_KEY_W])
       cameraPos += cameraSpeed * cameraFront;
   if(keys[GLFW_KEY_S])
       cameraPos -= cameraSpeed * cameraFront;
   if(keys[GLFW_KEY_A])
       cameraPos -= normalize(cross(cameraFront, cameraUp)) * cameraSpeed;
   if(keys[GLFW_KEY_D])
       cameraPos += normalize(cross(cameraFront, cameraUp)) * cameraSpeed;
}

void mouse_callback(GLFWwindow* window, double xpos, double ypos){
   if(initMouse){
       lastX = xpos;
       lastY = ypos;
       initMouse = false;
   }

    //下面我们要获取鼠标被移动了多少
   GLfloat xOffset = xpos - lastX;
   GLfloat yOffset = lastY - ypos;
    //这里x，y的获取方法不同，原因是y坐标是从底部到顶部依次增大的
   lastX = xpos;
   lastY = ypos;

   GLfloat h_sensitivity = 0.005f;
   GLfloat v_sensitivity = 0.005f;
   xOffset *= v_sensitivity;
   yOffset *= h_sensitivity;
    //这里我分开设置了x轴和y轴的速度，因为速度一致可能会导致一些人头晕

   gYaw += xOffset;
   gPitch += yOffset;

   if(gPitch > 89.0f)
       gPitch = 89.0f;
   if(gPitch < -89.0f)
       gPitch = -89.0f;

   vec3 front;
   front.x = cos(radians(gPitch)) * sin(-radians(gYaw));
   front.y = sin(radians(gPitch));
   front.z = cos(radians(gPitch)) * cos(-radians(gYaw));
    //这里的操作涉及到三角形方面的知识，设斜边为h，那么我们可以求出对边为h*sin theta，临边为h*cos theta
    //   |   
    //   |  /|
    //   | / | sin theta   
    //   |/  |
    // --+------------
    //   cos theta
    //然而就算了解这个知识，应用到实际还是比较难想象的，这里提供一个更直观的理解方法，首先先求pitch变换
    //两手指尖朝前，左手在下右手在上合在一起，之前提到过我们把相机反转了180度，所以现在指尖朝向是-z轴
    //绕x轴的顺时针运动是z，y，-z，-y，所以当我们接受一个正的pitch值时，就做顺时针运动，想象y轴从下往上穿过掌根，那么顺时针运动就是左手手掌向下分离（掌根紧贴），可以发现随着pitch值增加，实际角度也在增加所以根据三角知识，可以算出xyz的变化
    //下面求yaw的变换
    //两手指尖朝前，左手右手都在同一平面合在一起，绕y轴的顺时针运动是z，x，-z，-x，所以接收到一个正的yaw值时，左手不动右手分离（掌根紧贴），可以发现随着yaw值增加，实际角度实在减小的，所以带入运算时的角度要取反
    cameraFront = normalize(front);
    //我们只想要方向，所以标准化
}

void scrool_callback(GLFWwindow* window, double xoffset, double yoffset){
   fov -= (float) yoffset;
   if(fov < 1.0f)
       fov = 1.0f;
   if(fov > 45.0f)
       fov = 45.0f;
}
```

