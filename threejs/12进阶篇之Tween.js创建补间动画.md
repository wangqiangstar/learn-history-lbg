# 第12课：进阶篇之使用 Tween.js 创建补间动画

### Tween 是什么

Tween.js 是 JavaScript 中一个简单的补间动画库，包含各种经典动画算法。Tween.js 支持数字对象的属性和 CSS 样式属性赋值，API 简单且强大，支持链式调用。

补间（动画）（来自 In-Between）是一个概念，允许你以平滑的方式更改对象的属性。你只需告诉它哪些属性要更改，当补间结束运行时它们应该具有哪些最终值，以及这个过程需要多长时间，补间引擎将负责计算从起始点到结束点的值。

在 Three.js 中，我们有一些修改模型位置，旋转和缩放的需求，却无法直接在 WebGL 中使用 CSS3 动画来实现，而 Tween.js 恰好给我们提供了一个很好的解决方案。

比如我们要实现一个模型从 A 点到 B 点的位置移动，常规的实现方法，是使用 setInterval、requestAnimationFrame 手动计算出特定时间的位置点，很不易于管理与查看。而 Tween.js 可以自动根据起始点位置和动画时长计算出所有的位置点，可以很方便地对其进行获取和管理。

#### 简单应用

接下来，我们通过一个案例，带大家了解如何在 Three.js 应用中使用 Tween.js。

> 案例 Demo 查看地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/12第十二节 tween/simple.html)。
>
> 案例代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/12第十二节 tween/simple.html)。

本案例的开发思路是：首先获取目标模型的初始位置，然后实例化 Tween，接着设置目标位置，启动 Tween，在 TWEEN.onUpdate() 回调中改变目标模型的位置，从而实现目标模型从初始位置平滑移动到目标位置的动画。

实现代码如下：

```
 //设置tween
        var position = {x:-40, y:0, z:-30};
        tween = new TWEEN.Tween(position);
        //设置移动的目标和移动时间
        tween.to({x:40, y:30, z:30}, 2000);
        //设置每次更新的回调，然后修改几何体的位置
        tween.onUpdate(function (pos) {
            cube.position.set(pos.x, pos.y, pos.z);
        });
```

上面代码，首先创建一个 position 对象，存储了当前立方体的位置数据。然后，通过当前的对象创建了一个补间 tween。紧接着，设置每一个属性的目标位置，并告诉 Tween 在 2000 毫秒（动画时长）内移动到目标位置。最后，设置 Tween 对象每次更新的回调，即在每次数据更新以后，将立方体位置更新。

Tween 对象不会直接执行，需要我们调用 `start()` 方法激活，即 `tween.start()`。

```
//声明一个保存需求修改的数据对象。
gui = {
    start:function () {
        tween.start();
    }
};
```

想要完成整个过程，我们还需要在每帧里面调用 `TWEEN.update`，来触发 Tween 对象更新位置：

```
function render() {

    //更新Tween
    TWEEN.update();

    control.update();

    renderer.render(scene, camera);
}
```

#### 链式调用

链式调用可以简化大量代码，逻辑清晰集中，便于查看和修改。

Tween 插件也支持链式调用的方法，并且还会修改实例化时传入的对象，如下代码：

```
//设置tween
var position = {x:-40, y:0, z:-30};
tween = new TWEEN.Tween(position);

//设置移动的目标和移动时间
tween.to({x:40, y:30, z:30}, 2000);

//设置每次更新的回调，然后修改几何体的位置
tween.onUpdate(function (pos) {
    cube.position.set(pos.x, pos.y, pos.z);
});
```

可以简化为链式调用：

```
//直接链式实现tween
tween = new TWEEN.Tween(cube.position).to({x:40, y:30, z:30}, 2000);
```

### Tween 对象方法

#### 控制动画方法

Tween 对象控制动画的方法主要包括开始、取消、重复等方法。

- `.start()`

如果你想激活一个补间，请使用这个方法，调用方式如下所示：

```
tween.start();
```

`start()` 方法还接受一个时间参数，添加该参数后，补间不会立即被激活，Tween 动画将在延时该时间数后才开始动画。否则它将立刻开始动画，且在第一次调用 `TWEEN.update` 时开始计时。如果设置的时间已经小于计时的总时间，那计算出来的位置数据将是参数设置时间开始后，运行到的所在位置。

- `.stop()`

这个方法刚好和 `start()` 方法对应，如果你想取消一个补间，直接调用这个方法即可：

```
tween.stop();
```

- `.update()`

其实每个补间都有一个更新方法，只不过我们多会使用 `TWEEN.update` ，而不会单独调用它（见下方全局函数）。

- `.chain()`

当你按顺序排列不同的补间时，例如当上一个补间结束时立即启动另外一个补间，我们称它为链式补间。关于链式补间的案例请见下面两个链接。

> 案例 Demo 查看地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/12第十二节 tween/chain.html)
>
> 案例代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/12第十二节 tween/chain.html)

调用方法为：

```
tweenA.chain(tweenB);
```

或者，采用一个无限的链式，即 tweenA 与 tweenB 无限循环，便可以写成：

```
tweenA.chain(tweenB);
tweenB.chain(tweenA);
```

在其他情况下，您可能需要将多个补间链接到另一个补间，以使它们（链接的补间）同时开始动画：

```
tweenA.chain(tweenB,tweenC);
```

> 警告：调用 `tweenA.chain（tweenB）` 实际上修改的是 tweenA，tweenB 总在 tweenA 完成时启动。`chain()` 的返回值只是 tweenA，不是一个新的 Tween。
>
> 链接多个补间时，比如 `tweenA.chain(tweenB, tweenC)` 表示 tweenA 动画结束后，tweenB 和 tweenC 动画同时开始，如果因 tweenB 和 tweenC 修改的属性相同，而存在冲突时，经测试写在后面的属性将是最终动画位置。
>
> 注意，一般不要让两个同时开始的补间存在属性冲突。

- `.repeat()`

如果想让一个补间永远重复，可以无限链接自己，但更好的方法是使用 `repeat()` 方法。它接受一个参数，描述第一个补间完成后需要重复多少次，如下代码示例：

```
tween.repeat(10); // 循环10次
tween.repeat(Infinity); // 无限循环
```

我们可以将案例中 `simple.html` 文件里面的调用改成无限循环，代码如下：

```
tween = new TWEEN.Tween(cube.position).to({x:40, y:30, z:30}, 2000).repeat(Infinity);
```

- `.yoyo()`

该方法只有在补间使用 `repeat()` 方法时才会被调用。我们调用 `yoyo()` 以后，位置的切换就变成了从头到尾，从尾到头这样的循环过程。

单个补间无限从头到尾的循环，可以写成这样：

```
 tween = new TWEEN.Tween(cube.position).to({x:40, y:30, z:30}, 2000).repeat(Infinity).yoyo(true);
```

进一步了解 `yoyo()`方法的使用，可查看下面案例。

> 案例 Demo 查看地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/12第十二节 tween/repeat.html)。
>
> 案例代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/12第十二节 tween/repeat.html)。

- `.delay()`

这个方法用于控制激活前的延时，即触发 `start()` 事件后，需要延时到设置的 delay 时间，才会真正激活，使用方法见下面代码所示：

```
tween.delay(1000);
tween.start();
```

### 回调函数

Tween 每次的位置更新后，都会触发 onUpdate 回调函数，我们可以在此回调中修改模型位置。

在之前案例的 `simple.html` 中，我们在每次更新回调中获取更新后的位置信息并修改模型几何体的位置：

```
//设置每次更新的回调，然后修改几何体的位置
tween.onUpdate(function (pos) {
    cube.position.set(pos.x, pos.y, pos.z);
});
```

目前，补间支持的回调函数主要有以下几种。

- onStart

在补间计算开始前的回调，每个补间只能触发一下，即使使用 `repeat()` 方法循环，这个回调也只被触发一次。

- onStop

通过调用 `stop()` 方法停止的补间会触发当前回调，如果是正常完成的补间将不会触发此回调。

- onUpdate

每次补间更新后，我们可以在此回调中获取更新后的值。

- onComplete

当补间正常完成时，将会触发此回调。通过使用 `stop()` 停止的补间将不会触发此回调。

### 总结

在 Three.js 中使用 Tween.js，能够很方便地改变模型的位置，不需要手动计算，便能获取到具体的位置数据，以实现我们的需求。