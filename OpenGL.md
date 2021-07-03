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
> y轴方向向上，x轴方向向右，z轴方向向后，原点在图像中心。
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
glBindVertexArray(0);
```

包含EBO的VAO（图示）：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/04/vertex_array_objects_ebo.png)
