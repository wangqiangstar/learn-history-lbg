# 第07课：基础功能之 Camera 相机

相机是 Three.js 抽象出来的一个对象，使用此对象，我们可以定义显示的内容，并且可以通过移动相机位置来显示不同的内容。

下面讲解一下 Three.js 中相机的通用属性和常用的相机对象。

### 相机通用属性和方法

我们常用的相机有正交相机（OrthographicCamera）和透视相机（PerspectiveCamera）两种，用于来捕获场景内显示的物体模型。它们有一些通用的属性和方法。

#### Object3D 的所有属性和方法

由于相机都继承自 THREE.Object3D 对象，所以像设置位置的 position 属性、rotation 旋转和 scale 缩放属性，可以直接对相机对象设置。我们甚至还可以使用 `add()` 方法，给相机对象添加子类，移动相机它的子类也会跟随着一块移动，我们可以使用这个特性制作一些比如 HUD 类型的显示界面。

#### target 焦点属性和 lookAt() 方法

这两个方法的效果一定，都是调整相机的朝向，可以设置一个 `THREE.Vector3`（三维向量）点的位置：

```
camera.target = new THREE.Vector3(0, 0, 0);
camera.lookAt(new THREE.Vector3(0, 0, 0));
```

上面两个都是朝向了原点，我们也可以将相机的朝向改为模型网格的 position，如果物体的位置发生了变化，相机的焦点方向也会跟随变动：

```
var mesh = new THREE.Mesh(geometry, material);
camera.target = mesh.position;
//或者
camera.lookAt(mesh.position);
```

#### getWorldDirection()

getWorldDirection() 方法可以获取当前位置到 target 位置的世界中的方向。方向也可以使用 `THREE.Vector3` 对象表示，所以该方法返回一个三维向量。

### OrthographicCamera 正交相机

使用正交相机 OrthographicCamera 渲染出来的场景，所有的物体和模型都按照它固有的尺寸和精度显示，一般使用在工业要求精度或者 2D 平面中，因为它能完整的显示物体应有的尺寸。

#### 创建正交相机

![OrthographicCamera](http://images.gitbook.cn/54accde0-7573-11e8-ae69-7f0c27c81098)

上面的图片可以清晰的显示出正交相机显示的范围，它显示的内容是一个立方体结构，通过图片我们发现，只要确定 top、left、right、bottom、near 和 far 六个值，我们就能确定当前相机捕获场景的区域，在这个区域外面的内容不会被渲染，所以，我们创建相机的方法就是：

```
new THREE.OrthographicCamera( left, right, top, bottom, near, far );
```

下面我们创建了一个显示场景中相机位置前方长宽高都为4的盒子内的物体的正交相机：

```
var orthographicCamera = new THREE.OrthographicCamera(-2, 2, 2, -2, 0, 4);
scene.add(orthographicCamera); //一般不需要将相机放置到场景当中，如果需要添加子元素等一些特殊操作，还是需要add到场景内
```

正常情况相机显示的内容需要和窗口显示的内容为同样的比例才能够显示没有被拉伸变形的效果：

```
var frustumSize = 1000; //设置显示相机前方1000高的内容
var aspect = window.innerWidth / window.innerHeight; //计算场景的宽高比
var orthographicCamera = new THREE.OrthographicCamera( frustumSize * aspect / - 2, frustumSize * aspect / 2, frustumSize / 2, frustumSize / - 2, 1, 2000 ); //根据比例计算出left，top，right，bottom的值
```

#### 动态修改正交相机属性

我们也可以动态的修改正交相机的一些属性，注意修改完以后需要调用相机 updateProjectionMatrix() 方法来更新相机显存里面的内容，代码如下：

```
var frustumSize = 1000; //设置显示相机前方1000高的内容
var aspect = window.innerWidth / window.innerHeight; //计算场景的宽高比
var orthographicCamera = new THREE.OrthographicCamera(); //实例化一个空的正交相机
orthographicCamera.left = frustumSize * aspect / - 2; //设置left的值
orthographicCamera.right = frustumSize * aspect / 2; //设置right的值
orthographicCamera.top = frustumSize / 2; //设置top的值
orthographicCamera.bottom = frustumSize / - 2; //设置bottom的值
orthographicCamera.near = 1; //设置near的值
orthographicCamera.far = 2000; //设置far的值

//注意，最后一定要调用updateProjectionMatrix()方法更新
orthographicCamera.updateProjectionMatrix();
```

#### 窗口变动后的更新

由于浏览器的窗口可以随意修改，我们有时候需要监听浏览器窗口的变化，然后获取到最新的宽高比，再重新设置相关属性：

```
var aspect = window.innerWidth / window.innerHeight; //重新获取场景的宽高比

//重新设置left right top bottom 四个值
orthographicCamera.left = frustumSize * aspect / - 2; //设置left的值
orthographicCamera.right = frustumSize * aspect / 2; //设置right的值
orthographicCamera.top = frustumSize / 2; //设置top的值
orthographicCamera.bottom = frustumSize / - 2; //设置bottom的值

//最后，记得一定要更新数据
orthographicCamera.updateProjectionMatrix();

//显示区域尺寸变了，我们也需要修改渲染器的比例
renderer.setSize(window.innerWidth, window.innerHeight);
```

### PerspectiveCamera 透视相机

透视相机是最常用的也是模拟人眼视角的一种相机，它所渲染生成的页面是一种近大远小的效果。

#### 创建透视相机

![PerspectiveCamera](http://images.gitbook.cn/e27d7170-7577-11e8-a337-9bdf53091bce)

上面的图片就是一个透视相机的生成原理，我们先看看渲染的范围是如何生成的：

- 首先，我们需要确定一个 fov 值，这个值是用来确定相机前方的垂直视角，角度越大，我们能够查看的内容就越多。
- 然后，我们又确定了一个渲染的宽高比，这个宽高比最好设置成页面显示区域的宽高比，这样我们查看生成画面才不会出现拉伸变形的效果，这时，我们可以确定前面生成内容的范围是一个四棱锥的区域。
- 最后，我们需要确定的就是相机渲染范围的最小值 near 和最大值 far，注意，这两个值都是距离相机的距离，确定完数值后，相机会显示的范围就是一个近小远大的四棱柱的范围，我们能够看到的内容都是在这个范围内的。

通过上面的原理，我们需要设置 fov 垂直角度，aspect 视角宽高比例和 near 最近渲染距离 far 最远渲染距离，就能确定当前透视相机的渲染范围。

下面代码展示了一个透视相机的创建方法：

```
var perspectiveCamera = new THREE.PerspectiveCamera( 45, width / height, 1, 1000 );
scene.add( perspectiveCamera );
```

我们设置了前方的视角为45度，宽度和高度设置成显示窗口的宽度除以高度的比例即可，显示距离为1到1000距离以内的物体。

#### 动态修改透视相机的属性

透视相机的属性创建完成后我们也可以根据个人需求随意修改，但是注意，相机的属性修改完成后，以后要调用 `updateProjectionMatrix()` 方法来更新：

```
var perspectiveCamera = new THREE.PerspectiveCamera( 45, width / height, 1, 1000 );
scene.add( perspectiveCamera );

//下面为修改当前相机属性
perspectiveCamera.fov = 75; //修改相机的fov
perspectiveCamera.aspect = window.innerWidth/window.innerHeight; //修改相机的宽高比
perspectiveCamera.near = 100; //修改near
perspectiveCamera.far = 500; //修改far

//最后更新
perspectiveCamera.updateProjectionMatrix();
```

#### 显示窗口变动后的回调

如果当前场景浏览器的显示窗口变动了，比如修改了浏览器的宽高后，我们需要设置场景自动更新，下面是一个常用的案例：

```
function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight; //重新设置宽高比
    camera.updateProjectionMatrix(); //更新相机
    renderer.setSize(window.innerWidth, window.innerHeight); //更新渲染页面大小
}
window.onresize = onWindowResize;
```

最后，我写了一个案例来查看透视相机和正交相机的显示区别：[点击这里案例 Demo](https://johnson2heng.github.io/GitChat-Three.js/07第七节 camera/index.html)。

左侧是透视相机显示的效果，这种更符合人眼看到的效果，场景更加的立体。而右侧则是正交相机实现的效果，渲染出来的数值更加准确，但是却不符合人眼查看的效果。

案例代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/07第七节 camera/index.html)。

#### 制作相机控制器

在实际生产当中，很多时候我们需要切换相机的位置，以达到项目需求。 Three.js 也有很多相机控制插件，这个我们会在后面的课程当中讲解。

下面是我书写的一个小案例，能够通过鼠标左键拖拽界面让相机围绕模型周围查看模型。

首先，先上案例地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/07第七节 camera/control.html)。

代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/07第七节 camera/control.html)。

其实和以前的代码相比，我们也只是多出来了一个 initControl 方法，在这个方法里面绑定鼠标事件。实现的思路也很简单：

- 首先绑定鼠标按下事件，获取到按下时的相机位置和距离原点的距离，计算出来相对于 Z 轴正方向的偏移。绑定鼠标移动事件。
- 然后，在鼠标移动事件里面获取到距离鼠标按下的偏移，通过鼠标偏移数值计算出现在相机位置，并赋值。

思路就这么简单，虽然实现起来会麻烦，我也尽量写的简单，大家尽量能够自己也写出来。