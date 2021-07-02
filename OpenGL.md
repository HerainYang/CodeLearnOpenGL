## OpenGL

OpenGL是一个API，由Khronos组织制定和维护，具体实现方法由显卡开发商自行实现。相比早期OpenGL使用的立即渲染模式（固定渲染管线），现在的OpenGL使用的是更为灵活的核心模式（core-profile）。

OpenGL本身是一个状态机，用一系列的变量去告知OpenGL应该如何运作。规范来说，OpenGL的状态被称作上下文，开发者通过状态设置函数来改变上下文，从而使用OpenGL。

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

管线被分为几个阶段，每个阶段有对应的函数处理，这些函数被称作着色器（shader）。有的着色器允许开发者自行配置与替换，OpenGL的着色器使用OpenGL着色器语言（GLSL）编写，下图蓝色部分为可自定义的着色器。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/04/pipeline.png)

顶点数据包括三个3D坐标，用于表示一个三角形。

顶点着色器将这组数据映射到另一个3D坐标系中。

图元装配指将三个坐标以某种方式（点，线，三角形）组合到一起。

几何着色器可以构造新的顶点来形成新的形状。

光栅化阶段会把坐标映射到屏幕中的像素。

在片段着色器之前，超出视图范围的像素会被裁切，片段着色器会计算一个像素的最终颜色。

测试混合阶段会检测模板值，深度值，alpha值（通常用于混合颜色）。

具体管线内容可以查看本人Unity日常记录，有一章专门针对管线做出了解释。

---

