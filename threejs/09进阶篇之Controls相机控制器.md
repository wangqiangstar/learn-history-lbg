# 第09课：进阶篇之 Controls 相机控制器

我们在第07课中讲解过相机的相关内容，也制作了一个简易的相机控制器。

正常的项目中，大家的需求都是不一样的，又通常会碰上需求中途改变的情况，我们之前做的简易版相机控制器很难满足此类项目对相机的操作需求。而且，造轮子的前提是当前的框架以及插件已经无法满足自身的需求时，才会考虑造轮子。要不然，项目的进度会被拖得很慢，甚至有可能因此而错过红利期。

好在 Three.js 官方和同道中的朋友们给我们提供了很多相关的插件，我们可以根据需求引入相关的插件来实现需求，本文我们就来看一下官方案例中提供的相机控制器。

从官网下载的代码包里可以发现有很多的相机控制器，文件夹地址为：`/examples/js/controls/`，里面的文件插件都是和控制相机和控制模型相关的插件，我们罗列一下相关插件：

- DeviceOrientationControls：陀螺仪相机控制器，实现移动端陀螺仪控制相机。
- DragControls：控制鼠标拖拽移动物体的功能。
- EditorControls：实现相机的旋转、缩放、平移功能，相对于 OrbitControls 的功能差不少，不建议使用。
- FirstPersonControls：第一视角相机控制器。
- FlyControls：飞行相机控制器。
- OrbitControls：轨道控制器。
- OrthographicTrackballControls：正交轨迹球控制器——正交相机使用的轨迹球控制器。
- TrackballControls：轨迹球控制器——透视相机使用的轨迹球控制器。
- PointerLockControls：鼠标锁定相机控制器。
- TransformControls：控制模型位置、缩放、旋转的控制器。
- VRControls：实现 VR 双屏相机控制器。

由于篇幅有限，上面的控制器无法一一介绍，我们将重点介绍三种常用的相机控制器。

### OrbitControls

OrbitControls 控制器是我们常用的相机控制器，它的功能丰富，使用简单，为大多数项目的使用插件。

#### 使用操作

使用 OrbitControls 控制器我们可以实现旋转、缩放、平移等功能，下面简单列一下 OrbitControls 控制器的操作方法：

- 围绕焦点旋转：使用鼠标左键拖拽；
- 放大和缩小：使用鼠标中键按住拖拽或者鼠标中键滑动滚轮；
- 平移相机：按住鼠标右键拖拽或者使用键盘的上下左右键。

#### 控制器引入

在项目中使用 OrbitControls 控制器，可分为下面几步。

- 首先，将插件文件引入到项目中：

```
<script src="../js/OrbitControls.js"></script>
```

- 然后，通过相机和渲染器的 Dom 对象实例化相机：

```
control = new THREE.OrbitControls(camera, renderer.domElement);
```

- 最后，在每一帧渲染里面更新相机的位置：

```
function render() {
    control.update();
    renderer.render(scene, camera);
}
```

这样，我们就完成了一个最简单的 OrbitControls 控制器使用。

#### 属性和方法

OrbitControls 控制器最大的优势就是有丰富的配置项，供我们修改来实现项目中的需求，接下来我们看看有哪些属性配置。

| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| enabled            | 是否开启当前控制器，默认值是 True，如果设置为 False，将无法通过操作修改相机。 |
| target             | 控制器的焦点位置，是一个 `THREE.Vector3` 对象，默认是 `(0, 0, 0)` |
| minDistance        | 相机距离焦点的最近距离，默认值是0。 此属性适用于透视相机 PerspectiveCamera。 |
| maxDistance        | 相机距离焦点的最远距离，默认值是 Infinity（无限远）， 此属性适用于透视相机 PerspectiveCamera。 |
| minZoom            | 相机距离焦点的最近距离，默认值是0，此属性适用于正交相机 OrthographicCamera。 |
| maxZoom            | 相机距离焦点的最远距离，默认值是 Infinity（无限远），此属性适用于正交相机 OrthographicCamera。 |
| minPolarAngle      | 相机位置和焦点与焦点和最上方组成的最小夹角限制，默认值是0。  |
| maxPolarAngle      | 相机位置和焦点与焦点和最上方组成的最大夹角限制，默认值是 Math.PI，也就是180度角。 |
| minAzimuthAngle    | 当前相机沿水平方向顺时针旋转的弧度，默认值是 `- Infinity`。  |
| maxAzimuthAngle    | 当前相机沿水平方向逆时针旋转的弧度，默认值是 `Infinity`。    |
| enableDamping      | 是否开启拖拽惯性移动，即拖拽停止相机会有缓慢停止的距离移动，默认值是 False。 |
| dampingFactor      | 拖拽惯性移动的阻力，默认值是 0.25。                          |
| enableZoom         | 是否开启缩放操作，默认值是 True。                            |
| zoomSpeed          | 缩放速度，默认值是 1.0。                                     |
| enableRotate       | 是否开启相机绕焦点旋转操作，默认值是 True。                  |
| rotateSpeed        | 旋转速度，默认值是 1.0。                                     |
| enablePan          | 是否开启相机平移操作，默认值是 True。                        |
| panSpeed           | 平移的速度，默认值是 1.0。                                   |
| screenSpacePanning | 修改相机平移的方向，默认值是 False，即沿 x 轴正负方向和 y 轴正负方向移动。可选值是 True，可以修改为沿 x 轴正负方向和 y 轴正负方向移动。 |
| keyPanSpeed        | 键盘上下左右键移动相机的速度，默认值是 7.0。                 |
| autoRotate         | 当前相机是否自动旋转，默认值是 False，不自动旋转。           |
| autoRotateSpeed    | 自动旋转的速度，默认值是 2.0，即渲染满60帧的情况下30秒旋转360度。 |
| enableKeys         | 是否开启键盘控制先机平移，默认值是 True。                    |

OrbitControls 控制器的属性配置介绍完了，我们看看 OrbitControls 控制器还有那些方法。

##### **update()**

OrbitControls 控制器更新相机的方法，需要在每一帧里面调用。

##### **reset()**

重置方法，相机回到初始位置。

##### **dispose()**

销毁当前实例化的 OrbitControls 控制器。

##### **change 回调**

我们还可以监听相机改变回调，如果控制器修改了相机，将会产生一个回调：

```
controls.addEventListener('change', function(){
    console.log("相机动了！");
});
```

最后，我附上 OrbitControls 控制器案例：

- [点击这里](https://johnson2heng.github.io/GitChat-Three.js/09第九节 controls/OrbitControls.html)

也可以从这里获取案例源码：

- [点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/09第九节 controls/OrbitControls.html)

### TrackballControls

TrackballControls 控制器比 OrbitControls 控制器更自由，TrackballControls 控制器能够沿焦点进行球形旋转，没有死角，但比 OrbitControls 控制器少一些相关的功能配置。如何选择使用它们还是看项目需求，接下来还是先看如何操作。

注意，透视相机和正交相机使用的不是一个插件，此插件为透视相机使用，如果是正交相机请使用 OrthographicTrackballControls。

#### 使用操作

使用 TrackballControls 控制器我们可以实现旋转、缩放、平移等功能，下面说一下如何使用 TrackballControls 控制器进行操作：

- 围绕焦点旋转：使用鼠标左键拖拽；
- 放大和缩小：使用鼠标中键按住拖拽或者鼠标中键滑动滚轮；
- 平移相机：按住鼠标右键拖拽或者使用键盘的上下左右键。

TrackballControls 控制器和 OrbitControls 控制器的操作相同，没什么可说的。

#### 控制器引入

在项目中使用 TrackballControls 控制器和 OrbitControls 控制器的方法雷同，分为下面几步。

- 首先，将插件文件引入到项目中：

```
<script src="../js/TrackballControls.js"></script>
```

- 然后，通过相机和渲染器的 Dom 对象实例化相机：

```
control = new THREE.TrackballControls(camera, renderer.domElement);
```

- 最后，在每一帧渲染里面更新相机的位置：

```
function render() {
    control.update();
    renderer.render(scene, camera);
}
```

#### 属性和方法

| 属性                 | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| enabled              | 是否开启当前控制器，默认值是 True，如果设置为 False，将无法通过操作修改相机。 |
| rotateSpeed          | 控制相机旋转速度，默认值是 3.0。                             |
| zoomSpeed            | 控制相机缩放速度，默认值是 1.2。                             |
| panSpeed             | 控制相机平移速度，默认值是 0.3。                             |
| noRotate             | 关闭相机旋转，默认 False，开启。                             |
| noZoom               | 关闭相机缩放，默认 False，开启。                             |
| noPan                | 关闭相机移动，默认 False 开启。                              |
| staticMoving         | 关闭拖拽惯性移动 默认值 False，开启。                        |
| dynamicDampingFactor | 拖拽惯性移动阻力，默认值是 0.2。                             |
| minDistance          | 相机距离焦点的最近距离，默认值是 0。                         |
| maxDistance          | 相机距离焦点的最远距离，默认值是 Infinity（无限远）。        |

相对于 OrbitControls 控制器，TrackballControls 控制器的属性少一些，但是相关的功能还是比较全面的。TrackballControls 控制器的方法也和 OrbitControls 控制器的方法雷同。

##### **update()**

TrackballControls 控制器更新相机的方法，需要在每一帧里面调用。

##### **reset()**

重置方法，相机回到初始位置。

##### **dispose()**

销毁当前实例化的 TrackballControls 控制器。

##### change 回调

我们还可以监听相机改变回调，如果控制器修改了相机，将会产生一个回调：

```
controls.addEventListener('change', function(){
    console.log("相机动了！");
});
```

最后，附上 TrackballControls 控制器案例：

- [点击这里](https://johnson2heng.github.io/GitChat-Three.js/09第九节 controls/TrackballControls.html)

也可以从这里获取案例源码：

- [点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/09第九节 controls/TrackballControls.html)

### DeviceOrientationControls

最后，我们介绍的这个控制器只兼容含有陀螺仪的移动端。DeviceOrientationControls 控制器可以通过获取设备的陀螺仪状态来控制相机的朝向。

如果你还对陀螺仪不了解，[请点击查看这里](https://blog.csdn.net/qq_30100043/article/details/73323617)，在这里不多说。

DeviceOrientationControls 的内容配置较少，我们先看一下案例。

使用手机打开网址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/09第九节 controls/DeviceOrientationControls.html) ，然后手机朝下然后移动，你会发现能够通过手机的转向来控制相机的朝向，是不是很神奇。接下来我们看看如何引入到项目中。

- 首先，将插件文件引入到项目中：

```
<script src="../js/DeviceOrientationControls.js"></script>
```

- 然后，通过相机对象实例化相机：

```
control = new THREE.DeviceOrientationControls(camera);
```

- 最后，在每一帧渲染里面更新相机的位置：

```
function render() {
    control.update();
    renderer.render(scene, camera);
}
```

这样我们就完成了对 DeviceOrientationControls 控制器的添加。

DeviceOrientationControls 控制器相关的配置也很少，只有一个 Enabled 属性，设置为 True，则控制器会更新相机的位置，反之，设置 False 将无法更新相机位置。

还有一个方法就是销毁当前控制器的方法：

```
controls.dispose(); //销毁当前控制器
```

最后，附上源码：

- [点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/09第九节 controls/DeviceOrientationControls.html)