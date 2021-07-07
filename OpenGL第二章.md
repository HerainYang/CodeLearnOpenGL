# OpenGL第二章

因为第一章写了一万多字Typora就开始有点卡了，所以决定一章一个文件，从现在开始第二章就逐渐进入光照的介绍了。

## 颜色

OpenGL里对颜色的处理是符合生活常理的，即：眼睛看到的颜色是物体拒绝吸收的光的颜色，也就是说一个蓝色的物体其实是它拒绝吸收蓝色。

> 当我们把光源的颜色与物体的颜色值相乘，所得到的就是这个物体所反射的颜色（也就是我们所感知到的颜色）。

```glsl
glm::vec3 lightColor(1.0f, 1.0f, 1.0f);
glm::vec3 toyColor(1.0f, 0.5f, 0.31f);
glm::vec3 result = lightColor * toyColor; // = (1.0f, 0.5f, 0.31f);
```

## 基础光照

现实中的光照非常复杂，出于性能考量，我们在OpenGL中使用的是简化的光照模型，其中一个常用的模型叫做**冯氏光照模型**（Phong Lighting Model），这种模型由三个分量组成：

* 环境（Ambient）
* 漫反射（Diffuse）
* 镜面（Specular）

环境光照

```glsl
float ambientStrength = 0.1;
//常量环境因子
vec3 ambient = ambientStrength * lightColor;
```

漫反射

![img](https://learnopengl-cn.github.io/img/02/02/diffuse_light.png)

漫反射的原理是，光照越垂直一个物体，它对物体的影响会越大，为了测量这个垂直程度，我们要用到法向量，正如先前所说，两角点乘越接近1，它们的夹角越小，这里用的就是这个原理。

```glsl
vec3 norm = normalize(Normal);
vec3 lightDir = normalize(lightPos - FragPos);
float diff = max(dot(norm, lightDir), 0.0);
//如果两角为钝角，点乘会返回负数，但是这不是我们想要的结果
vec3 diffuse = diff * lightColor;
```

在万事大吉之前，还有一件事要注意一下，还记得我们处理模型位置的时候，第一步做了什么吗？从模型空间转换到世界空间。既然模型位置需要转换，法向量要不要转换呢？答案是要。

但是和位置信息不一样，法向量是一个向量，正如定义，向量包含位置信息，所以：

1. model矩阵的位移操作应该被去除。
2. 我们应该专注于缩放和旋转变换。

不过如果模型执行了不等比的缩放，会导致法向量不再垂直于表面，为了纠正这个问题，我们使用**法线矩阵**（Normal Matrix）。

![img](https://learnopengl-cn.github.io/img/02/02/basic_lighting_normal_transformation.png)

法线矩阵实质上是模型矩阵左上角的逆矩阵的转置矩阵，在顶点着色器中我们可以自己生成这个矩阵：

```glsl
Normal = mat3(transpose(inverse(model))) * aNormal;
//            转置矩阵   逆矩阵   模型矩阵
```

镜面光照

所谓镜面光照其实就是**高光**（Specular Highlight），它的原理如图，结合生活常识应该不难理解：

![img](https://learnopengl-cn.github.io/img/02/02/basic_lighting_specular_theory.png)

最终高光呈现出来的效果是，夹角越小，镜面光越强烈。

```glsl
float specularStrength = 0.5;
//镜面强度
vec3 viewDir = normalize(viewPos - FragPos);
//算出入射向量
vec3 reflectDir = reflect(-lightDir, norm);
//算出出射向量，在漫反射的时候我们算的lightDir是从镜面出发到光源位置，所以这里要取反
float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
//32指代反光度，反光度越大，散射越少
//这部操作还算了出射向量和观察向量的角度
vec3 specular = specularStrength * spec * lightColor;
```

算出三个光，我们再将它们汇总，这样就算出了最后的冯氏着色：

```glsl
vec3 result = (ambient + diffuse + specular) * objectColor;
FragColor = vec4(result, 1.0);
```

这章的最后再提一下Gouraud着色，Gouraud着色本质是再顶点着色器实现的冯氏着色，相比片段着色器来说，顶点着色器要处理的顶点要少很多，所以Gouraud着色会更加高效，但因为顶点少了，更多的地方要通过线性插值来计算光照颜色，这样处理的光照可以说是靠猜，就不够真实。