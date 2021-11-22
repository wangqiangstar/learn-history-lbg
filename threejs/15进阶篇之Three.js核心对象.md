# 第15课：进阶篇之 Three.js 核心对象

本文是介绍 Three.js 核心技术知识的最后一节，带大家了解经常接触到的对象。第16课，也是课程的最后一节，我们就开始实战演练。

### THREE.Clock

时间对象用于跟踪时间，用于计算每一帧的渲染时间或者从开始渲染到结束的时间。

#### 构造函数

构造函数的使用，如下所示：

```
var clock = new THREE.Clock();
```

实例化时间对象，还可以接收一个布尔类型的参数，用于设置实例化完成后，是否启动时间对象计时，默认为开启。

#### 方法

该类主要有以下几种方法。

- **.start()：**此方法用于启动时间对象开始计时，将所有的信息重置；
- **.stop()：**停止时间对象的计时；
- **.getElapsedTime()：**此方法返回自时间对象开启以后到现在的时间；
- **.getDelta()：**此方法返回上次调用到这次调用此方法的间隔时间，用于计算渲染间隔。

### THREE.Color

THREE.Color 为颜色类，用到颜色的地方，都可以实例化此类，如设置材质颜色、场景背景颜色等。

#### 构造函数

我们可以通过 `new THREE.Color()` 传入一个值来初始化一个颜色对象：

```
//不传参数将默认渲染成白色
var color = new THREE.Color();

//十六进制颜色（推荐）
var color = new THREE.Color( 0xff0000 );

//RGB 字符串
var color = new THREE.Color("rgb(255, 0, 0)");
var color = new THREE.Color("rgb(100%, 0%, 0%)");

//支持所有140个颜色名称
var color = new THREE.Color( 'skyblue' );

//HSL 字符串
var color = new THREE.Color("hsl(0, 100%, 50%)");

//支持区间为从0到1的RGB数值
var color = new THREE.Color( 1, 0, 0 );
```

#### 方法

该类主要有以下几种方法。

- **.clone()：**返回一个相同颜色的颜色对象；
- **.add()：**将一个颜色对象的颜色值和此颜色对象的颜色相加；
- **.multiply()：**将一个颜色对象的颜色值和此颜色对象的颜色相乘；
- **.set()：**重新设置颜色对象的颜色，支持实例化时的所有方式。

### 矢量对象

矢量是 GLSL ES 语言内的标准数据类型，也是 WebGL 原生着色器语言的数据类型。而 Three.js 为了方便和原生融合，内置了 `THREE.Vector2`、`THREE.Vector3` 和 `THREE.Vector4` 三种矢量，分别代表二维向量、三维向量和四维向量。

#### 二维矢量

二维矢量一般表示二维平面上点的位置和方向。

下面代码生成了一个普通的二维矢量：

```
var a = new THREE.Vector2( 0, 1 );
```

我们也可以通过 `set()` 方法，重新修改二维矢量的值：

```
a.set(2, 3);
```

#### 三维矢量

在项目当中，我们经常会用到三维矢量，比如模型的位置属性、缩放属性，都是一个三维矢量。它由三个有序的数字组成的。

使用三维矢量，我们可以表示：

1. 三维空间中一个点的位置；
2. 三维空间中两点连线的方向和长度；
3. 任意有序的三个数字。

我们看下面这个例子：

```
//创建一个普通的三维矢量
var a = new THREE.Vector3( 0, 1, 0 );

//默认空值，将会重置为(0, 0, 0)
var b = new THREE.Vector3( );

//d矢量是从a点到b点的方向和长度
var d = a.distanceTo( b );
```

我们经常进行的模型位置改变以及缩放操作，其实都是通过修改三维矢量的值来实现的，示例代码如下：

```
//创建一个三维矢量
var vec = new THREE.Vector3(0, 1, 0);

//修改模型的位置
mesh.position = vec;

//修改三维矢量的值
vec.set(1, 2, 1);

//修改模型的缩放值
mesh.scale.set(2, 2, 2);
```

我们还可以修改三维向量的单个值：

```
vec.setComponent(0, 20); //将三维向量的第一值修改为20

.setComponent(index, value); 
//index 修改的单个值的下标，可选值为 0 1 2 对应 vec.x vec.y vec.z
//value 修改成的数字
```

#### 四维矢量

四维矢量是由四个数字组成的数据类型，四个值分别代表 x、y、z、w。

四维矢量可以表示的内容有：

1. 思维空间内的一个点；
2. 思维空间里的方向和长度；
3. 任意有序的四个数字。

我们再看下面这个例子：

```
//创建一个普通的四维矢量
var a = new THREE.Vector4( 0, 1, 0, 0 );

//如果不设置参数，默认值为 (0, 0, 0, 1)
var b = new THREE.Vector4( );
```

### 四维矩阵

矩阵是高等代数中的常见工具，也常见于统计分析等应用数学学科中。Three.js 为我们封装了三维矩阵（3x3）四维矩阵（4x4）。

篇幅有限，这里就不介绍三维矩阵了，我们主要看下四维矩阵。

在 3D 计算机图形中，最常见的四维矩阵主要用于转换矩阵。这代表三维空间中的点 Vector3 通过乘以转换矩阵，可以实现如平移、旋转、剪切、缩放、发射、正交或透视投影等变化。

在每个 3D 对象中都有三个四维矩阵。

- Object3D.matrix：存储 3D 对象的局部变换，这是 3D 对象相对于父对象的转换；
- Object3D.matrixWorld：3D 对象的全局或世界变换，如果 3D 对象没有父对象，则当前矩阵和其局部变换矩阵相同。
- Object3D.modelViewMatrix：表示 3D 对象相对于摄像机的坐标转换，对象的 modelViewMatrix 是对象的 matrixWorld 再乘以相机的 matrixWorldInverse 获得的结果。

相机对象不但含有 3D 对象的四维矩阵，还额外拥有以下两个四维矩阵。

- Camera.matrixWorldInverse：视图矩阵（相机的 matrixWorld 反转后的四维矩阵）；
- Camera.projectionMatrix：表示场景内的模型如何转换到二维显示区域的变换矩阵。

如果你对矩阵的实现原理感兴趣的话，可[点击这里](https://blog.csdn.net/qq_30100043/article/details/72783071)查看，这里不再过多解释原理。

接下来我们查看一下 Three.js 封装的四维变换矩阵为我们提供了哪些方法。

```
var m = new THREE.Matrix4();

//我们可以通过手动设置每一项的数值来生成变换矩阵
m.set( 11, 12, 13, 14,
       21, 22, 23, 24,
       31, 32, 33, 34,
       41, 42, 43, 44 );

//创建一个矢量，并应用此变换矩阵
var v = new THREE.Vector3();

v.applyMatrix4(m); //应用变换矩阵
```

上面代码创建了一个默认的四维矩阵，三维矢量乘以默认的四维矩阵将不会产生变化。

我们可以使用 `.identity()` 方法来重置变换矩阵：

```
var m = new THREE.Matrix4();
m.identity(); //重置矩阵恢复默认
```

我们可以通过 `.lookAt()` 传入一个三个矢量生成一个旋转矩阵。这三个矢量分别代表眼的位置、查看的物体位置和向上的方向：

```
var m = new THREE.Matrix4();
m.lookAt(eye, center, up); //生成旋转变换矩阵
```

我们也可以通过旋转弧度来生成旋转变换矩阵：

```
var m = new THREE.Matrix4();
//通过绕x轴旋转生成变换矩阵
m.makeRotationX(Math.PI/2); //绕x轴旋转90度变换矩阵
m.makeRotationAxis(new THREE.Vector3(1, 0, 0), Math.PI/2); //效果同上

//通过绕y轴旋转生成变换矩阵
m.makeRotationY(Math.PI); //绕y轴旋转180度变换矩阵
m.makeRotationAxis(new THREE.Vector3(0, 1, 0), Math.PI); //效果同上

//通过绕z轴旋转生成变换矩阵
m.makeRotationZ(Math.PI/4); //绕z轴旋转45度变换矩阵
m.makeRotationAxis(new THREE.Vector3(0, 0, 1), Math.PI/4); //效果同上
```

通过设置缩放来生成缩放变换矩阵：

```
var m = new THREE.Matrix4();

m.makeScale(1, 2, 5); //生成沿x轴不变，y轴放大两倍，z轴放大五倍的缩放变换矩阵
```

通过设置位置平移来生成变换矩阵：

```
var m = new THREE.Matrix4();

m.makeTranslation(10, -20, 100); //生成沿x正轴偏移10，y负轴偏移20，z正轴偏移100的平移变换矩阵
```

### 欧拉角

欧拉角通过在每个轴指定旋转的弧度和指定旋转轴的先后顺序来进行旋转变换。之前介绍场景时已做介绍，不再赘述。这里想说明的是，每个 3D 对象的 rotation 属性都是一个欧拉角对象。

创建欧拉角定义方法如下：

```
var e = new THREE.Euler( 0, 1, 1.57, 'XYZ' ); 
```

前三个值分别代表每个轴上旋转的弧度，第四个值是应用旋转的顺序。默认为“XYZ”，这意味着对象将首先围绕其 X 轴旋转，然后围绕其 Y 轴旋转，最后围绕其 Z 轴旋转。其他可能性有 YZX、ZXY、XZY、YXZ、ZYX，都必须大写。

修改欧拉角的方法如下：

```
//通过set方法重新设置
var e = new THREE.Euler();
e.set(Math.PI, 0, - Math.PI / 2, "YZX"); //先沿y轴旋转180度，再沿z轴旋转0度，最后沿x轴逆时针旋转90度，第四个值可以不填，默认是"XYZ"
```

我们也可以通过变换矩阵来修改当前的欧拉角：

```
var e = new THREE.Euler( 0, 1, 1.57, 'XYZ' );
var m = new THREE.Matrix4();

e.setFromRotationMatrix(m); //在当前的旋转角度，再进行矩阵变换
```

### 结语

Three.js 知识讲解部分，到这里基本就结束了。我第一次写正式的教程，难免有一些瑕疵，有不足的地方欢迎在读者圈给我留言。同时，希望该教程能带给大家一些感悟。

下一篇，也就是课程最后一篇，我将会编写一个简单的小案例。大家经常玩王者荣耀，那 Three.js 能否实现王者荣耀那种人物操作呢？答案是肯定的。我将通过实现逻辑分析加案例解析，带大家完成这个案例。