# 第02课：初识 Three.js

上一篇最后，我分享了一个简单案例，供大家查看 Three.js 的 “Hello World”。案例没有什么复杂的东西，也就是一个旋转的立方体。接下来，我将为大家讲解如何实现这个最简单的案例，以及 Three.js 的一些最基本的构件。

### 构建场景

按照代码的运行顺序，首先，我们需要先将 Three.js 库的 JS 文件引入，代码如下：

```
<script src="https://cdn.bootcss.com/three.js/92/three.js"></script>
```

然后在 body 上面添加了一个初始化的事件 init：

```
<body onload="init()">
```

页面加载完毕后，回调会触发我们设置的 init 事件。init 函数里面一共调用了五个方法，分别是：初始化渲染器、初始化场景、初始化相机、添加模型、添加动画：

```
//初始化函数，页面加载完成是调用
function init() {
    initRenderer();
    initScene();
    initCamera();
    initMesh();

    animate();
}
```

**使用 Three.js 显示创建的内容，我们必须需要的三大件是：渲染器、相机和场景。**相机获取到场景内显示的内容，然后再通过渲染器渲染到画布上面。接下来，我们将分步骤讲解它们的实现。

#### 创建渲染器

```
//初始化渲染器
function initRenderer() {
    renderer = new THREE.WebGLRenderer(); //实例化渲染器
    renderer.setSize(window.innerWidth, window.innerHeight); //设置宽和高
    document.body.appendChild(renderer.domElement); //添加到dom
}
```

查看 initRenderer 函数内的代码，第一行我们实例化了一个 `THREE.WebGLRenderer`，这是一个基于 WebGL 渲染的渲染器，当然，Three.js 向下兼容，还有 CanvasRenderer、CSS2DRenderer、CSS3DRenderer 和 SVGRenderer，这四个渲染器分别基于 canvas2D、CSS2D、CSS3D 和 SVG 渲染的渲染器。由于，作为 3D 渲染，WebGL 渲染的效果最好，并且支持的功能更多，我们以后的课程也只会用到 `THREE.WebGLRenderer`，需要使用其他渲染器时，会重点提示。

第二行，调用了一个设置函数 setSize 方法，这个是设置我们需要显示的窗口大小。案例我们是基于浏览器全屏显示，所以设置了浏览器窗口的宽和高。

第三行，`renderer.domElement` 是在实例化渲染器时生成的一个 Canvas 画布，渲染器渲染界面生成的内容，都将在这个画布上显示。所以，我们将这个画布添加到了 DOM 当中，来显示渲染的内容。

#### 创建相机

接着我们看一下相机的初始化：

```
//初始化相机
function initCamera() {
    camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 200); //实例化相机
    camera.position.set(0, 0, 15);
}
```

Three.js 里面有几个不同的相机，我们这里使用到的是 `THREE.PerspectiveCamera`，这个相机的效果是模拟人眼看到的效果，就是具有透视的效果，近大远小。

第一行，我们实例化了一个透视相机，需要四个值，分别是视野、宽高比、近裁面和远裁面。我们分别介绍一下这四个值：

- 视野：当前相机视野的宽度，值越大，渲染出来的内容也会更多。
- 宽高比：默认是按照画布显示的宽高比例来设置，如果比例设置的不对，会发现渲染出来的画面有拉伸或者压缩的感觉。
- 近裁面和远裁面：这个是设置相机可以看到的场景内容的范围，如果场景内的内容位置不在这两个值内的话，将不会被显示到渲染的画面中。

第二行，我们设置了相机的位置，在讲如何设置位置之前，我觉得应该先讲一下 WebGL 的坐标系统：

![坐标系](http://images.gitbook.cn/9bf88bf0-6108-11e8-b977-f33e31f528f0)

WebGL 坐标系统作为 3D 坐标，在原来的 2D 坐标 x、y 轴上面又多了一个 z 轴，大家注意 z 轴的方向，是坐标轴朝向我们的方向是正轴，我们眼看去的方向是 z 轴的负方向。

`camera.position.set` 函数是设置当前相机的位置，函数传的三个值分别是 x 轴坐标，y 轴坐标和z 轴坐标。我们只是将相机放到了 z 正轴坐标轴距离坐标原点的15的位置。相机默认的朝向是朝向0点坐标的，我们也可以设置相机的朝向，这将在后面介绍相机时，专门介绍相关知识。

#### 创建场景

初始化场景方法里面代码很简单，只有一行，就是实例化一个场景对象：

```
//初始化场景
function initScene() {
    scene = new THREE.Scene(); //实例化场景
}
```

场景只是作为一个容器，我们将需要显示的内容都放到场景对象当中。如果我们需要将一个模型放入到场景当中，则可以使用 `scene.add` 方法，如：

```
scene.add(mesh); //添加一个网格（模型）到场景
```

### 创建第一个模型

渲染器，场景和相机都全了，是不是就能显示东西了？不能！因为场景内没有内容，即使渲染出来也是一片漆黑，所以我们需要往场景里面添加内容。接下来，我们将查看 initMesh 方法，看看如何创建一个最简单的模型：

```
//创建模型
function initMesh() {
    geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
    material = new THREE.MeshNormalMaterial(); //创建材质

    mesh = new THREE.Mesh( geometry, material ); //创建网格
    scene.add( mesh ); //将网格添加到场景
}
```

创建一个网格（模型）需要两种对象：几何体和材质。

- 几何体代表模型的形状，它是由固定的点的位置组成，点绘制出面，面组成了模型。
- 材质是我们看到当前模型显示出来的效果，如显示的颜色，质感等。

在创建模型函数的第一行代码里，我们实例化了一个 `THREE.BoxGeometry` 立方体的几何体对象，实例化的三个传值分别代表着立方体的长度，宽度和高度。我们全部设置成相同的值，将渲染出一个标准的正立方体。

在第二行里面，我们实例化了一个 `THREE.MeshNormalMaterial` 材质，这种材质的特点是，它会根据面的朝向不同，显示不同的颜色。

紧接着，通过 `THREE.Mesh` 方法实例化创建了一个网格对象，`THREE.Mesh` 实例化需要传两个值，分别是几何体对象和材质对象，才可以实例化成功。

最后，我们通过场景的 add 方法，将网格添加到了场景当中，我们没有给立方体设置位置，立方体则会默认显示在坐标的原点上面。这时，相机就可以拍摄到我们创建的立方体的画面了。

### 让场景动起来

动画，就是多幅图片一直切换便可显示动画的效果。为了能显示动画的效果，我们首先要了解一个函数 requestAnimationFrame，这个函数专门为了动画而出现。它与 setInterval 相比，优势在于不需要设置多长时间重新渲染，而是在当前线程内 JS 空闲时自动渲染，并且最大帧数控制在一秒60帧。所以，我们书写了一个可以循环调用的函数：

```
function animate() {
    requestAnimationFrame(animate); //循环调用函数
    ...
}
```

在循环调用的函数中，每一帧我们都让页面重新渲染相机拍摄下来的内容：

```
renderer.render( scene, camera ); //渲染界面
```

渲染的 render 方法需要两个值，第一个值是场景对象，第二个值是相机对象。这意味着，你可以有多个相机和多个场景，可以通过渲染不同的场景和相机让画布上显示不同的画面。

但是，如果现在一直渲染的话，我们发现就一个立方体在那，也没有动，我们需要做的是让立方体动起来：

```
mesh.rotation.x += 0.01; //每帧网格模型的沿x轴旋转0.01弧度
mesh.rotation.y += 0.02; //每帧网格模型的沿y轴旋转0.02弧度
```

每一个实例化的网格对象都有一个 rotation 的值，通过设置这个值可以让立方体旋转起来。在每一帧里，我们让立方体沿 x 轴方向旋转0.01弧度，沿 y 轴旋转0.02弧度（1π 弧度等于180度角度）。

### Three.js 的性能检测插件

在 Three.js 里面，遇到最多的问题就是性能问题，所以我们需要时刻检测当前的 Three.js 的性能。现在 Three.js 常使用的一款插件叫 stats。接下来我们看看如何将 stats 插件在 Three.js 的项目中使用。

- 首先在页面中引入插件代码：

```
<script src="http://www.wjceo.com/lib/js/libs/stats.min.js"></script>
```

我这里也直接给出了一个 CDN 的地址，可以直接引入即可。

- 然后，我们需要实例化一个 stats 对象，然后把对象内生成的 DOM 添加到页面当中。

```
stats = new Stats();
document.body.appendChild(stats.dom);
```

- 最后一步，我们需要在 requestAnimationFrame 的回调里面更新每次渲染的时间：

```
function animate() {
    requestAnimationFrame(animate); //循环调用函数
    stats.update(); //更新性能插件
    renderer.render( scene, camera ); //渲染界面
}
```

在场景当中，我们能够看到左上角会有一个性能检测的小框：

![性能检测插件](http://images.gitbook.cn/8b74f250-6108-11e8-b977-f33e31f528f0)

前面的数值代表当前每秒的渲染帧率，后面括号内的值是当前场景渲染的帧率范围。

说到这里，我们的 Three.js 的“HELLO WORLD”也就实现了，我将在下一篇，给大家讲解场景的数据结构和开发的辅助插件。

使用了性能检测插件以后，我们的案例代码结构应该是这样的：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset=utf-8>
    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Stats插件案例</title>
    <style>
        body {
            margin: 0;
        }

        canvas {
            width: 100%;
            height: 100%;
            display: block;
        }
    </style>
</head>
<body onload="init()">
<script src="https://cdn.bootcss.com/three.js/92/three.js"></script>
<script src="http://www.wjceo.com/lib/js/libs/stats.min.js"></script>
<script>
    //声明一些全局变量
    var renderer, camera, scene, geometry, material, mesh, stats;

    //初始化渲染器
    function initRenderer() {
        renderer = new THREE.WebGLRenderer(); //实例化渲染器
        renderer.setSize(window.innerWidth, window.innerHeight); //设置宽和高
        document.body.appendChild(renderer.domElement); //添加到dom
    }

    //初始化场景
    function initScene() {
        scene = new THREE.Scene(); //实例化场景
    }

    //初始化相机
    function initCamera() {
        camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 200); //实例化相机
        camera.position.set(0, 0, 15);
    }

    //创建模型
    function initMesh() {
        geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
        material = new THREE.MeshNormalMaterial(); //创建材质

        mesh = new THREE.Mesh( geometry, material ); //创建网格
        scene.add( mesh ); //将网格添加到场景
    }

    //运行动画
    function animate() {
        requestAnimationFrame(animate); //循环调用函数

        mesh.rotation.x += 0.01; //每帧网格模型的沿x轴旋转0.01弧度
        mesh.rotation.y += 0.02; //每帧网格模型的沿y轴旋转0.02弧度

        stats.update(); //更新性能检测框

        renderer.render( scene, camera ); //渲染界面
    }

    //性能检测框
    function initStats() {
        stats = new Stats();
        document.body.appendChild(stats.dom);
    }

    //初始化函数，页面加载完成是调用
    function init() {
        initRenderer();
        initScene();
        initCamera();
        initMesh();
        initStats();

        animate();
    }
</script>
</body>
</html>
```