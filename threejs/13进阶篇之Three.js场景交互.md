# 第13课：进阶篇之 Three.js 场景交互

浏览器是一个 2D 视口，而 Three.js 展示的是 3D 场景。场景交互时，需要在二维平面中控制三维场景的模型，那如何将 2D 视口的 x 和 y 坐标转换成 Three.js 场景中的 3D 坐标呢？

好在 Three.js 已经有了解决相关问题的方案，那就是 `THREE.Raycaster` 射线，用于鼠标拾取（计算出鼠标移过的三维空间中的对象）等。我们看下面这张图片：

![raycaster](https://images.gitbook.cn/d5f05ad0-9a20-11e8-8334-9bfa28241acd)

我们一般都会设置三维场景的显示区域，如果指明当前显示的 2D 坐标给 `THREE.Raycaster`，它将生成一条从显示起点到终点的射线，也就是射线与近视面交点和射线与远视面交点连成的这一条直线。在相机视角下查看，只是一个点，射线会穿过整个显示场景，并按从近到远的顺序返回与模型相交的数据。

### THREE.Raycaster 构造函数和对象方法

#### 实例化

```
new THREE.Raycaster( origin, direction, near, far );
```

该实例化函数 Raycaster 包含了四个参数。

- origin：光线投射的原点矢量；
- direction：光线投射的方向矢量，应该是被归一化的；
- near：投射近点，用来限定返回比 near 要远的结果。near 不能为负数，缺省为 0；
- far：投射远点，用来限定返回比 far 要近的结果。far 不能比 near 小，缺省为无穷大。

#### 属性

`THREE.Raycaster` 的属性可以在实例化对象后有修改需求时再修改。除了上面提到的 origin、direction、near、far 四个属性外，我们还有可能用到另一个属性：

- linePrecision：射线和线相交的精度，浮点数类型的值。

#### 方法

`THREE.Raycaster` 给我们提供了一系列的方法，比如修改射线的位置，判断与某些模型是否相交等，接下来我们列举一些经常使用的方法。

- `.set()`：此方法可以重新设置射线的原点和方向，从而更新射线位置。

```
.set（origin，direction）
```

其中，参数 origin 用来设置射线新的原点矢量位置，direction 用来设置基于原点位置的射线的方向矢量。

- `.setFromCamera()`：使用当前相机和界面的 2D 坐标设置射线的位置和方向。

```
.setFromCamera ( coords, camera )
```

参数 coords 表示鼠标的二维坐标，在归一化的设备坐标（NDC）中，也就是 X 和 Y 分量，应该介于 -1 和 1 之间。camera 表示射线起点处的相机，即把射线起点设置在该相机位置处。

点击事件大多通过鼠标触发，我们用鼠标点击显示区域的位置和当前场景使用的相机对象调用此对象，Three.js 会为我们计算出当前射线的位置。

- `.intersectObject ()` 和 `.intersectObjects ()`

两个方法用来检查射线和物体之间的所有交叉点数据。

如果检测射线和一个对象是否相交，推荐使用 `intersectObject()`，如果判断的是这个对象的子对象，那推荐使用 `intersectObjects()`，将 3D 对象的 children 属性传入。

返回值是一个交叉点对象数组，且按距离排序，最接近的排在首位。

```
.intersectObject ( object, recursive, optionalTarget)
```

参数 object，用来检测和射线相交的物体。如果 recursive 设置为 true，还会向下继续检查所有后代，否则只检查该对象本身，缺省值为 false。optionalTarget 为可选参数，用于设置放置结果的数组，如果缺省，则将会实例化一个新数组，并将获取到的数据放入其中。

```
.intersectObjects ( array, recursive, optionalTarget)
```

`intersectObject()` 和 `intersectObjects()` 的区别在于第一个参数。intersectObject 的第一个参数为 3D 对象，而 intersectObjects 需要传入一个由 3D 对象组成的数组。

**我们知道两个方法的返回值均为对象数组。接下来，我们再进一步了解下这个返回值。**

如果射线与场景内的模型没有相交，将返回一个空数组，否则，将返回一个按从近到远顺序排列的对象数组，数组中每个对象的内容为：

```
[ { distance, point, face, faceIndex, indices, object }, ... ]
```

其中：

- distance：射线的起点到相交点的距离；
- point：在世界坐标中的交叉点；
- face：相交的面；
- faceIndex：相交的面的索引；
- indices：组成相交面的顶点索引；
- object：相交的对象。

当一个网孔（Mesh）对象和一个缓存几何模型（BufferGeometry）相交时，faceIndex 将是 undefined，并且 indices 将被设置；而当一个网孔（Mesh）对象和一个几何模型（Geometry）相交时，indices 将是 undefined。

当计算这个对象是否和射线相交时，Raycaster 把传递的对象委托给检测 3D 对象的 raycast 方法，该方法通过计算，检测出当前模型与射线是否相交。

这允许 Mesh 对光线投射的反应可以不同于 lines 和 pointclouds。Mesh、lines、pointclouds 是三种不同的计算相交的方法。Mesh 对射线与网格对象的每一个面进行相交判断。lines，则是判断两条线之间的距离，至于 pointclouds，则是通过 distanceSqToPoint 判断当前是否相交。

**注意**，对于网格，面（faces）必须朝向射线原点，这样才能被检测到。背面射线的交叉点将无法被检测到。

为了使光线能投射到一个对象的正反两面，你需要设置 material 的 side 属性为 THREE.DoubleSide。

### 模型点击事件的实现

上面讲解了射线的相关内容，接下来，我们来看一下，如何使用射线实现一个普通的点击事件。

首先，我们通过点击事件回调的 event 获取点击的位置：

```
mouse.x = ( event.clientX / window.innerWidth ) * 2 - 1;
mouse.y = - ( event.clientY / window.innerHeight ) * 2 + 1;
```

默认没有经过矩阵转换过的显示区域的宽和高分别是 2，即中心点也是 WebGL 场景的坐标原点，左上角的坐标是 `(-1.0, 1.0, 0.0)`，右下角的坐标是 `(1.0, -1.0, 0.0)`。我们通过单击点的位置计算出当前该点在场景中，没有被矩阵转换过的平面坐标。如果 WebGL 的渲染区域没有占满窗口，我们还需获取显示区域距离窗口左上角的偏移量，再计算位置，计算方法如下：

```
//通过 dom 的 getBoundingClientRect 方法获得当前显示区域距离左上角的偏移量
var left = renderer.domElement.getBoundingClientRect().left;
var top = renderer.domElement.getBoundingClientRect().top;

//根据浏览器的设备类型来获取到当前点击的位置
var clientX = dop.browserRedirect() === "pc" ? event.clientX - left : event.touches[0].clientX - left;
var clientY = dop.browserRedirect() === "pc" ? event.clientY - top : event.touches[0].clientY - top;

//计算出场景内的原始坐标
mouse.x = (clientX / renderer.domElement.offsetWidth) * 2 - 1;
mouse.y = -(clientY / renderer.domElement.offsetHeight) * 2 + 1;
```

获取到坐标以后，我们需要使用射线的 `setFromCamera()` 方法配合场景坐标和相机更新射线的位置：

```
raycaster.setFromCamera( mouse, camera );
```

接着，使用 `intersectObjects()` 方法获取射线和所有模型相交的数组集合：

```
var intersects = raycaster.intersectObjects( scene.children );
```

这里再提醒一句，很多读者可能发现，有时点击后射线并未获取到相交的物体。这是因为我们一般使用 `intersectObject()` 和 `intersectObjects()` 时，只会传入对象，而有的模型由多个模型组成，也就是它的子类，这时我们需要传入第二个值，设置为 true，来提示 Three.js 遍历它的子类。

最后，如果有与射线相交的模型，返回的 intersects 数组的长度将不为零：

```
if(intersects.length > 0){
    alert("有相交的模型");
}
```

这里提供一个点击案例，点中物体后，模型颜色将会变色：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/13第十三节 场景交互/raycaster.html) 。

案例源码地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/13第十三节 场景交互/raycaster.html)。

### 简单框选案例的实现

最近有些小伙伴想实现一个简单的框选案例，在这一篇中我来带大家完成。

本案例通过判断模型的位置实现框选，即框选前先获取所有模型在二维平面上的位置，然后再判断这些二维平面上的点是否处于框内。

相对于其它实现方式，这种实现节约性能，简单易懂，能够应付大部分场景。

接下来，我讲解一下这个框选的实现思路。

首先，编写以下代码，当鼠标按下时，记录鼠标按下时的场景坐标：

```
//获取到显示区域距离窗口左上角的偏移量
domClient.x = renderer.domElement.getBoundingClientRect().left;
domClient.y = renderer.domElement.getBoundingClientRect().top;
//计算出当前鼠标距离显示区域左上角的距离
down.x = e.clientX - domClient.x;
down.y = e.clientY - domClient.y;
```

前两行代码求出了当前显示区域距离窗口左上角的偏移量，后两行则计算出来了当前鼠标点击位置距离显示区域左上角的偏移。

接着，使用 box 对象方法计算出模型的包围盒中心位置，适用于多个复杂模型场景。如果只是简单几何体，可以直接使用 Mesh 的位置来计算。通过相机将世界坐标的位置转换为平面坐标，并将模型放到一个数组内以便后期使用：

```
for (let i = 0; i < group.children.length; i++) {
    let box = new THREE.Box3();
    box.expandByObject(group.children[i]);

    //获取到平面的坐标
    let vec3 = new THREE.Vector3();
    box.getCenter(vec3);
    let vec = vec3.project(camera);

    modelsList.push(
        {
            component: group.children[i],
            position: {
                x: vec.x * half.width + half.width,
                y: -vec.y * half.height + half.height
            },
            normalMaterial: group.children[i].material
        }
    )
}
```

上面代码首先通过一个循环，计算出了每一模型在二维平面中的位置。

接下来，绑定 Document 的 mousemove 事件和 mouseup 事件。鼠标移动事件用来判断每个模型是否处于框内，鼠标抬起事件则将绑定的事件清除。

```
//绑定鼠标按下移动事件和抬起事件
document.addEventListener("mousemove", movefun, false);
document.addEventListener("mouseup", upfun, false);
```

在鼠标移动事件中，我们计算出当前四个边的位置，并且循环判断哪些模型的位置处于框内，处于框内的模型的材质将被修改为框选材质：

```
for (let i = 0; i < modelsList.length; i++) {
    let position = modelsList[i].position;
    //判断当前位置是否处于框内
    if (position.x > min.x && position.x < max.x && position.y > min.y && position.y < max.y) {
        modelsList[i].component.material = material;
    }
    else{
        modelsList[i].component.material = modelsList[i].normalMaterial;
    }
}
```

在最后的鼠标抬起事件内，将框选框隐藏，并将所有材质修改为默认材质：

```
function upfun(e) {

    //清除事件
    document.body.removeChild(div);
    document.removeEventListener("mousemove", movefun, false);
    document.removeEventListener("mouseup", upfun, false);

    //将所有的模型修改为当前默认的材质
    for (let i = 0; i < modelsList.length; i++) {
        modelsList[i].component.material = modelsList[i].normalMaterial;
    }
}
```

最后，附上可以查看的案例地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/13第十三节 场景交互/boxselection.html)。

案例源码地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/13第十三节 场景交互/boxselection.html)