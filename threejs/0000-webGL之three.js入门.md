入门建议：

webGL中文翻译教程，基于NeHe的openGL教程：http://www.hiwebgl.com/?p=42 。

WebGL中文网 http://www.hewebgl.com/ ，里面有THREEJS教程，THREEJS初级教程免费 ，其余要收费。但是初级教程也很不错。

three.js官方网站：www.threejs.org/  ，three.js-master包里面有很多很有趣的例子，虽然官方没有给出文档，但是学习一段时间后，自己就能看懂代码了。

推荐书籍：【美】Andreas Anyuru的《WebGL高级编程  --开发Web 3D图形》，【美】Kouichi Matsuda Rodger的《WebGL编程指南》，【美】Jos Dirksen的《Three.js 开发指南》，看这几本差不多就能写出像样的东西了。当然，想深入挖掘的的同学也可以看看opengl、计算机图形学、矩阵论等相关方面的书。

------

# 一、基本需要

## 初始化：

1.渲染器（renderer）——渲染物体的，在web浏览器中必须有一个平台显示3d效果

2.照相机（camera）——视角，展现在我们面前的

3.场景（scene）

4.灯光（light）

5.物体（object）

## 渲染部分：

1.renderer.render(scene, camera);

2.requestAnimationFrame(animate); 

![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602150916633-630000661.png)

 

### 1.初始化渲染器：

three.js的渲染器主要有WebGLRenderer和CanvasRenderer，

渲染的效果不同有细微差异，我们使用的是WebGLRenderer，当使用渲染器时，一般在html页面上加入<div>标签——>这里的div标签是<div id="canvasframe"></div>

```
renderer = new THREE.WebGLRenderer({antialias: true});//开启WebGL抗锯齿

width = document.getElementById('canvasframe').clientWidth;

height = document.getElementById('canvasframe').clientHeight;

renderer.setSize(width, height);

document.getElementById('canvasframe').appendChild(renderer.domElement);
```

你也可以不需要div标签创建渲染器

代码：

```
var renderer = new THREE.WebGLRenderer(); 
renderer.setSize(window.innerWidth, window.innerHeight); 
document.body.appendChild(renderer.domElement);
```

### 2.照相机

Camera是三维世界中的观察者，为了观察这个世界，首先我们要描述空间中的位置。 

![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151523586-909340412.png)

Three中使用采用常见的右手坐标系定位。

three.js提供两类照相机，一类是OrthographicCamera（正交相机），一类是PerspectiveCamera（透视相机）

 ![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151018993-876413959.png)

正交投影与透视投影的区别如上图所示，左图是正交投影，物体发出的光平行地投射到屏幕上，远近的方块都是一样大的；右图是透视投影，近大远小，符合我们平时看东西的感觉。 

```
camera = new THREE.PerspectiveCamera(60, width / height, 1, 10000);

camera.position.set(lengthX,lengthY, 100);

camera.up = new THREE.Vector3(0,0,1);
```

THREE.PerspectiveCamera(fov, aspect, near, far)

fov：视角度数，可以理解为眼睛睁开大小，度数越大，你看到的物体越多，物体越小

aspect：是照相机水平方向和竖直方向长度的比值，一般等于width/height

near和far：是相机最近和最远的距离，均为正值，far>>near

 

camera.position.set(x,y,z);//照相机的位置

camera.up = new THREE.Vector3(0,0,1);

cmmera.up哪个轴是向上的，three.js默认是右手坐标轴，即y轴向上，这里我们让z轴向上

camera.position = new THREE.Vector3(0,0,1)和camera.up.set(0,0,1)一样的效果。

 

#### 正交投影相机：

 ![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151056477-865529649.png)

**注：**图中的”视点”对应着Three中的Camera。 
这里补充一个视景体的概念：视景体是一个几何体，只有视景体内的物体才会被我们看到，视景体之外的物体将被裁剪掉。这是为了去除不必要的运算。 
正交投影相机的视景体是一个长方体，OrthographicCamera的构造函数是这样的：OrthographicCamera( left, right, top, bottom, near, far ) 
Camera本身可以看作是一个点，left则表示左平面在左右方向上与Camera的距离。另外几个参数同理。于是六个参数分别定义了视景体六个面的位置。

可以近似地认为，视景体里的物体平行投影到近平面上，然后近平面上的图像被渲染到屏幕上。

#### 透视投影相机

 ![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151117571-1492030603.png)

透视投影相机的视景体是个四棱台，它的构造函数是这样的：PerspectiveCamera( fov, aspect, near, far ) 
fov对应着图中的视角，是上下两面的夹角。aspect是近平面的宽高比。在加上近平面距离near，远平面距离far，就可以唯一确定这个视景体了。 
透视投影相机很符合我们通常的看东西的感觉，因此大多数情况下我们都是用透视投影相机展示3D效果。

### 3.场景

一般只需要scene = new THREE.Scen();新建场景就可以了

源代码中加入了scene.fog = new THREE.FogExp2(0xcccccc,0.002);使场景产生雾效果

FogExp2(hex,density)

hex,雾的颜色如果设置为黑色，远处的物体将呈现黑色

density：定义雾增长速度如何密集。默认为0.00025。

### 4.灯光

有几种灯光，常用的有AmbintLight（环境光），directionalLight（平行光），PointLigth（点光源），spotLight（聚光灯），半球光HemisphereLight，其他的你可以在官方文档里查找使用，初始化都比较简单，以程序里的平行光为例

```
light = new THREE.DirectionalLight( 0xffffff );

light.position.set( 1, 1, 1 );

scene.add( light );
```

设置平行光颜色为白色，在坐标（1,1,1）的位置，平行光类似太阳关照，添加到场景中

### 5.物体

有了相机，总要看点什么吧？在场景中添加一些物体吧。 
Three中供显示的物体有很多，它们都继承自Object3D类，这里我们主要看一下Mesh和Points两种。

#### Mesh

我们都知道，计算机的世界里，一条弧线是由有限个点构成的有限条线段连接得到的。线段很多时，看起来就是一条平滑的弧线了。 
计算机中的三维模型也是类似的，普遍的做法是用**三角形组成的网格**来描述，我们把这种模型称之为Mesh模型。

 

![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151147774-1192761069.png)

 

这是那只著名的斯坦福兔子。它在3D图形中的地位与数字图像处理领域中著名的lena是类似的。 
看这只兔子，随着三角形数量的增加，它的表面越来越平滑/准确。

在Three中，Mesh的构造函数是这样的：Mesh( geometry, material ) 
不止是Mesh，创建很多物体都要用到这两个属性。下面我们来看看这两个重要的属性。

 

普通的物体有两种属性，一个是它的材质material，一个是它的模型geometry（就是它长什么样子）。

```
var material = new THREE.MeshLambertMaterial({color:color });

var geometry = new THREE.CylinderGeometry(radius,radius,height,radius*100,0,false);

geometry.applyMatrix( new THREE.Matrix4().setRotationFromEuler( new THREE.Vector3( Math.PI / 2, Math.PI, 0 ) ) );

mesh = new THREE.Mesh(geometry,material);

scene.add(mesh);
```

#### Geometry，形状

相当直观。Geometry通过存储模型用到的点集和点间关系(哪些点构成一个三角形)来达到描述物体形状的目的。 
Three提供了立方体(其实是长方体)、平面(其实是长方形)、球体、圆形、圆柱、圆台等许多基本形状； 
你也可以通过自己定义每个点的位置来构造形状； 
对于比较复杂的形状，我们**还可以通过外部的模型文件导入**。

#### material——材质

材质其实是物体表面除了形状以为所有可视属性的集合，例如**色彩、纹理、光滑度、透明度、反射率、折射率、发光度**。 
这里讲一下材质(Material)、贴图(Map)和纹理(Texture)的关系。 
材质上面已经提到了，它包括了贴图以及其它。 
贴图其实是‘贴'和‘图'，它包括了图片和图片应当贴到什么位置。 
纹理嘛，其实就是‘图'了。 
Three提供了多种材质可供选择，能够自由地选择漫反射/镜面反射等材质。

three.js有多种材质可选

 ![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151204305-273722322.png)

 

Lambert材质：Lambert 材质（MeshLambertMaterial）是符合 Lambert 光照模型的材质。大部分物体的漫反射效果都适用的。就是说光的颜色和它本身的颜色是啥，它就照成啥

```
THREE.MeshLambertMaterial({color:color });
```

phong材质也比较适合管道：与lambert材质不一样的地方是，它加入了镜面反射的效果，也就是会有光斑

 

```
var material = new THREE.MeshPhongMaterial({

color:color,

specular:0xffffff,

shininess:100

});
```

spcular:光斑的颜色

shininess:光斑大小，值越小，光斑越不明显，不能设为0，30是默认值（0-100）就可。

其他材质的学习可以参照three.js官网和入门指南。（官网在这快有一个很好的例子，可以尝试一下）

 

THREE.CylinderGeometry(radiusTop, radiusBottom, height, radiusSegments, heightSegments, openEnded)函数创建了一个圆柱体，这个函数默认在原点（0,0,0）坐标创建，中心点也在原点，并且方向方向向y轴，你可以通过position和rotation属性分别改变它的位置和旋转角度，rotation属性要注意的一点是它里面的3个值分别表示弧度。

比如cylinder.rotation=new THREE.Vector3(0.1,0.2,0.3);

圆柱体绕x轴旋转的弧度是0.1，绕y轴旋转的弧度是0.2，绕z轴旋转的弧度是0.3。

 

radiusTop：顶面半径

radiusBottom：底面半径

height：圆柱高

radiusSegments：半径分段，就是一个圆被分成了多少份（像切蛋糕一样），值越大，圆柱就越平滑

heightSegments：高度分段，因为我们圆柱顶面半径，底面半径都是相同的，所以这个值就没有意义了，所以我们直接设为0。

openEnded 是一个布尔值，表示是否没有顶面和底面，缺省值为 false ，表示有顶面和底面。

![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151236336-996789011.png)<——radiusSegments和heightSegments的分段的效果

 

```
mesh = new THREE.Mesh(geometry,material);

scene.add(mesh);
```

把material材质赋给geometry，在场景中添加这一个物体。

 

动画渲染效果：

```
function render(){

       renderer.render( scene, camera );//把照相机给场景

}
```

动画的实现是通过在每秒钟多次重绘画面实现的，为了衡量画面切换速度，引入了每秒帧数FPS(Frames Per Second)的概念，是指每秒画面重绘的次数，FPS越大，则动画效果越平滑，当FPS小于20时，一般就能明显感受到卡顿

requestAnimationFrame:不断重绘

 

three.js里面除了requestAnimationFrame方法，还有setInterval(func,mesc)，可以设置特定的FPS，

func:每过mesc毫秒执行的函数，如果将func定义为重绘画面的函数，就能实现动画效果（Threejs入门指南P88）

```
function animation(){

       requestAnimationFrame(animate);// 反复渲染成为3d效果

       render();//由于requestAnimationFrame只请求一帧画面，除了在init函数中需要调用，//在被其调用的函数中需要再次调用requestAnimationFrame。

}
```

 

#### Points

讲完了Mesh，我们来看看另一种Object——Points。 
Points其实就是一堆点的集合

 

# 二、扩展

## 1.小工具

开源工具还有其他很有用的小工具

①stats.js——查看web场景中的帧数等，很实用！

fps可以实时的FPS信息，从而更好地监测动画效果![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151300727-899523473.png)

 

单击后可以显示每一帧的渲染时间![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602151309414-887638490.png)

```
stats = new Stats();

stats.domElement.style.position = 'absolute';

stats.domElement.style.top = '0px';

stats.domElement.style.zIndex = 100;

container.appendChild(stats.domElement);
```

之后再function animation()函数里面加入stats.update()就可以了

 

②three.js里面的坐标轴，可以在场景的原点加入一个有色坐标，其中红色是x轴，绿色是y轴，蓝色是z轴，比较实用

```
var axisHelper = new THREE.AxisHelper( 200 );//轴长200

scene.add( axisHelper );
```

③显示有线的照相机——能够显示照相机的角度（详情参看three.js官网）。。

```
cameraOrtho = new THREE.OrthographicCamera( 0.3 * window.innerWidth / - 2, 0.3 * window.innerWidth / 2, window.innerHeight / 2, window.innerHeight / - 2, 150, 1000 );
cameraOrthoHelper = new THREE.CameraHelper( cameraOrtho );
scene.add( cameraOrthoHelper );
```

④显示网格grid

```
function initGrid(){
                var helper = new THREE.GridHelper( 1000, 50 );
                helper.setColors( 0x0000ff, 0x808080 );
                scene.add( helper );
            }
```

![img](https://images2015.cnblogs.com/blog/787426/201706/787426-20170602152157477-421468475.png)

在场景中生成网格线。

⑤tween.js可辅助生成动画效果，很实用。

⑥通过.obj的后缀形式把物体加入场景

一般是在3d_max中导出物体的模型.obj和材质.mtl,通过OBJMTLLoader.js和MTLLoader.js实现，代码也很简单，但是记住在3d_max里面会记住模型的xyz轴，所以在3d建模软件中（一般是3dsMax或者Maya）绘制模型时候一定要把模型放在原点中心，这样在three.js引入3d模型时，会直接放入你想放的位置

```
function addInterObject(x,y,z){

var loader = new THREE.OBJMTLLoader();

    loader.addEventListener( 'load', function ( obj ) {

        object = obj.content;

        object.position = new THREE.Vector3( x, y, z);

        scene.add( object );

    });

    loader.load( 'obj/interbig.obj','obj/interbig.mtl' );

}
```

 

## 需要注意的地方

①大多数部件都有这两个属性position（位置）,rotation（旋转）。

②在three.js中，能形成阴影的光源只有THREE.DirectionalLight与THREE.SpotLight;而相对的，能表现阴影效果的材质只有THREE.LambertMaterial与THREE.PhongMaterial。

 ③注意不同环境下的坐标系。