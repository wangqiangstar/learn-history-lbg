# 第11课：进阶篇之 Three.js 动画

动画一般可以分为两种：一种是变形动画，另一种是骨骼动画。下面，我们先介绍一下变形动画。

### 变形动画

变形动画，通过修改当前模型的顶点位置来实现。比如，一个动画需要变动十次才可以实现，那么我们需要为当前模型的每一个顶点定义每一次所在的位置，Three.js 通过这一次次的修改实现了动画的整个流程。

为了帮助大家更好地理解变形动画的实现与使用，我创建了一个案例，查看地址为：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/11第十一节 animation/morphTargets.html)。

在这个案例的右上角，我们能发现两个可切换的拖拽条。这两个拖拽条对应的是两个变形目标数组，拖拽范围是0-1，即当前的变形目标对本体的影响程度。拖拽它们，可发现界面中的立方体也会跟随之变动，从而影响当前的立方体。接下来我讲解一下，该案例的实现过程。

首先，创建模型的几何体，并为几何体 morphTargets 赋值两个变形目标。morphTargets 是一个数组，我们可以为其增加多个变形目标。在给 morphTargets 添加变形目标时，需要为其定义一个名称和相关的顶点，这个顶点数据必须和默认的模型的顶点数据保持一致，设置完后，我们需要调用 geometry 的 `computeMorphNormals()` 进行更新，代码如下：

```
var cubeGeometry = new THREE.BoxGeometry(4, 4, 4);

// 创建两个影响立方体的变形目标
var cubeTarget1 = new THREE.BoxGeometry(2, 10, 2);
var cubeTarget2 = new THREE.BoxGeometry(8, 2, 8);

// 将两个geometry的顶点放入到立方体的morphTargets里面
cubeGeometry.morphTargets[0] = {name: 'target1', vertices: cubeTarget1.vertices};
cubeGeometry.morphTargets[1] = {name: 'target2', vertices: cubeTarget2.vertices};
cubeGeometry.computeMorphNormals();
```

然后，为当前模型设置材质，变形目标作为参数之一，可以使其变形。

```
var cubeMaterial = new THREE.MeshLambertMaterial({morphTargets: true, color: 0x00ffff});
```

接着，将创建好的网格模型添加到场景中。这时可以在 mesh 对象中找到 morphTargetInfluences 配置项，它也是一个数组，和 geometry 的 morphTargets 相对应，主要用来设置当前变形目标对本体的影响度，默认值为0-1，0为不影响本体，1为完全影响本体：

```
gui = {
    influence1:0.01,
    influence2:0.01,
    update : function () {
        cube.morphTargetInfluences[0] = gui.influence1;
        cube.morphTargetInfluences[1] = gui.influence2;
    }
};
```

至此，我们就手动实现了一个变形动画。在这个过程中，我们发现，变形动画是由于不断修改变形目标对本体的影响度而产生的。我们可以通过这个原理实现其他变形动画。

案例代码查看地址：[请点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/11第十一节 animation/morphTargets.html)。

### 骨骼动画

实现骨骼动画，我们需要生成一个与模型相关的骨架。骨架中的骨骼与骨骼之间存在关联，模型的每一个要动的顶点需要设置影响它的骨骼以及骨骼对顶点的影响度。

和变形动画相比，骨骼动画更复杂一些，但又有更多的灵活性。使用变形动画，我们需要把所有的每一次的变动都存在一个顶点数组中，而骨骼动画，只需要设置骨骼的相关信息，就可以实现更多的动画。

下面我们看一个骨骼动画的简单案例：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/11第十一节 animation/skeleton.html)

![skeleton](https://images.gitbook.cn/5411e080-8da8-11e8-b093-3f5eaa2cb071)

这是官方提供的一个案例。我对其做了些简单修改，以显示出当前一个柱形图形的骨骼。实现起来比较复杂，我们需要先理解它是怎么实现的。

首先， 我们创建了一个圆柱几何体，通过圆柱的几何体每个顶点的 y 轴坐标位置来设置绑定的骨骼的下标和影响的程度：

```
//遍历几何体所有的顶点
for (var i = 0; i < geometry.vertices.length; i++) {

    //根据顶点的位置计算出骨骼影响下标和权重

    var vertex = geometry.vertices[i];
    var y = (vertex.y + sizing.halfHeight);

    var skinIndex = Math.floor(y / sizing.segmentHeight);
    var skinWeight = (y % sizing.segmentHeight) / sizing.segmentHeight;

    geometry.skinIndices.push(new THREE.Vector4(skinIndex, skinIndex + 1, 0, 0));
    geometry.skinWeights.push(new THREE.Vector4(1 - skinWeight, skinWeight, 0, 0));

}
```

几何体的 skinIndices 属性和 skinWeights 属性分别用来设置绑定的骨骼下标和权重（骨骼影响程度）。

相应的，我们需要一组相关联的骨骼。骨骼具有嵌套关系，才得以实现一个骨架。圆柱体比较简单，我们直接创建一条骨骼垂直嵌套的骨骼：

```
bones = [];

var prevBone = new THREE.Bone();
bones.push(prevBone);
prevBone.position.y = -sizing.halfHeight;

for (var i = 0; i < sizing.segmentCount; i++) {

    var bone = new THREE.Bone();
    bone.position.y = sizing.segmentHeight;
    bones.push(bone); //添加到骨骼数组
    prevBone.add(bone); //上一个骨骼定义为父级
    prevBone = bone;

}
```

创建纹理时，我们还需要设置当前材质属性，并开启骨骼动画对其的修改权限，将材质的 skinning 属性设置为 true：

```
var lineMaterial = new THREE.MeshBasicMaterial({
    skinning: true,
    wireframe: true
});
```

最后，我们需要创建骨骼材质，并将模型绑定骨骼：

```
mesh = new THREE.SkinnedMesh(geometry, [material, lineMaterial]);
var skeleton = new THREE.Skeleton(bones); //创建骨架
mesh.add(bones[0]); //将骨骼添加到模型里面
mesh.bind(skeleton); //模型绑定骨架
```

这样，我们就使用 Three.js 创建了一个简单的骨骼动画。使用 `dat.gui`，便于我们修改每一个骨骼的 poisition、rotation 和 scale 并查看对当前模型的影响。

案例的源码地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/11第十一节 animation/skeleton.html)。

### 两种动画的区别

变形动画主要用于精度要求高的动画，比如人物的面部表情。其优点是动画的展现效果很到位，缺点就是扩展性不强，只能执行设置好的相关动画。

骨骼动画主要用于精度要求相对低一些，但需要丰富多样的动画的场合，就比如人物的走动，攻击防御等动画，我们可以通过一套骨骼，修改相应骨骼的位置信息直接实现相应的效果。它没有变形动画的精度高，但可以实现多种多样的效果。

> **总结：** 我们可以根据项目的需求来设置不同的动画，就比如一个人物模型，说话我们使用变形动画去实现，而肢体动作使用骨骼动画去实现。

### 导入模型动画

在 Three.js 动画系统中，你可以为模型的各种属性设置动画，如骨骼动画，变形动画，以及材质的相关属性（颜色，透明度， 是否可见）。动画属性可以设置淡入淡出效果以及各种扭曲特效，也可以单独改变一个或多个对象上的动画影响程度和动画时间。

为了实现这些，Three.js 动画系统在2015年修改为了类似于 Unity 和虚幻引擎4的架构。接下来我们了解下这套动画系统的主要组件以及它们是如何协同工作的。

#### 动画片段（Animation Clips）

在我们成功导入模型以后，如果模型拥有相关的动画属性，会在返回的模型数据中产生一个名为 animations 的数组，数组的每一个子项都是一个 AnimationClips 对象。

每一个单独 AnimationClips 对象相应的保存着模型的一个动画的数据，假如，如果模型网格是一个人物角色，第一个 AnimationClips 对象有可能保存的是人物走动的动画，第二个 AnimationClips 对象用于跳跃，第三个用于攻击动画等等。

#### 关键帧轨迹（Keyframe Tracks）

在 AnimationClips 对象内部，一般有四个属性：

- name：当前动画的名称；
- uuid：一个不会重复的 uuid；
- duration：当前动画一个循环所需要的时间；
- tracks：轨迹，即当前动画每一次切换动作所需要的数据。

假设当前的动画是骨骼动画，在关键帧轨迹中存储的数据是每一帧骨骼随着时间变动的数据（位置，旋转和缩放等）。

如果当前动画是一个变形动画，在关键帧轨迹中将会把顶点数据的变动存储在其中（比如实现人脸的笑以及哭等动作）。

#### 动画混合器（Animation Mixer）

在动画片段中存储的数据仅仅构成了动画实现的基础，实际的播放权力在动画混合器的手中。你可以想象动画混合器其实不仅仅只是作为动画的播放器，它还可以同时控制几个动画，混合它们或者合并它们。

#### 动画播放器（Animation Actions）

这个英文我更乐意将它翻译成动画播放器，因为我们最终需要将数据生成一个动画播放器来操作当前的动画执行，暂停或者停止，是否使用淡入淡出效果或者将动画加快或减慢。

#### 动画对象组（Animation Object Groups）

如果你希望一组模型对象共享当前的动画，我们可以使用动画对象组来实现。

### 通过导入模型显示动画

![变形动画](https://images.gitbook.cn/316ee240-8e92-11e8-bfb3-8b3f110a427d)

#### 变形动画

我们首先查看一个官方的模型案例，这个案例是一匹马奔跑的动画，我们也可以通过下面地址查看：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/11第十一节 animation/morphAnimation.html)。

接下来我们看一下这匹马是如何实现的。

- 在模型加载成功以后，我们首先将模型创建出来，并将材质的 morphTargets 设置为 ture，使顶点数据信息可以受变形动画影响：

```
mesh = new THREE.Mesh(geometry, new THREE.MeshLambertMaterial({
    vertexColors: THREE.FaceColors,
    morphTargets: true
}));
mesh.castShadow = true;
mesh.scale.set(0.1, 0.1, 0.1);
scene.add(mesh);
```

- 然后我们创建了一个针对于该模型的混合器：

```
mixer = new THREE.AnimationMixer(mesh);
```

- 接着使用变形目标数据创建一个动画片段：

```
var clip = THREE.AnimationClip.CreateFromMorphTargetSequence('gallop', geometry.morphTargets, 30);
```

- 使用混合器和动画片段创建一个动画播放器来播放：

```
var action = mixer.clipAction(clip); //创建动画播放器
action.setDuration(1); //设置当前动画一秒为一个周期
action.play(); //设置当前动画播放
```

- 最后，我们还需要在重新绘制循环中更新混合器，进行动作更新：

```
function render() {

    control.update();

    var time = clock.getDelta();
    //由于模型导入是异步的，所以我们再模型没有加载完之前是获取不到混合器的
    if (mixer) {
        mixer.update(time);
    }

    renderer.render(scene, camera);
}
```

![骨骼动画](https://images.gitbook.cn/27172660-8e95-11e8-8ee0-a17ea463076e)

#### 骨骼动画

骨骼动画模型，我们使用的是 gltf 格式，这个模型是在 Sketchfab 网站下载的，案例是一个小姐姐跳舞的片段，查看地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/11第十一节 animation/skeletonAnimation.html)。

gltf 格式的模型导入进来后，我们可以直接通过 animations 数组创建播放器：

```
mixer = new THREE.AnimationMixer(obj); //通过当前模型创建混合器
action = mixer.clipAction(gltf.animations[0]); //通过动画数据创建播放器
```

直接调用播放器的播放事件让动画播放：

```
action.play();
```

最后，我们还需要在循环渲染中更新混合器，并将每一帧渲染的间隔时间传入：

```
function render() {
    control.update();
    var time = clock.getDelta();
    if (mixer) {
        mixer.update(time);
    }
    renderer.render(scene, camera);
}
```