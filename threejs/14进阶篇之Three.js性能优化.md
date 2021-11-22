# 第14课：进阶篇之 Three.js 性能优化

在接触 Three.js 一段时间后，或多或少都会遇到性能问题。此类问题将直接导致页面帧率过低，严重时会导致页面崩溃。

导致性能问题的原因有很多，比如模型文件太大或顶点数太多导致加载时间过长，甚至失败，又或者代码书写逻辑有问题导致场景帧数太低。面对不同原因，我们需采取不同的应对方案，比如面对模型问题可对模型进行减面、减体积等处理；针对代码问题，则需尽量减少代码的运算。在平时的开发中，我们还要尽可能避免导致这些性能问题的根由。

下面，我将分享几种简单的有助于提升性能的方法。

### 尽量共用几何体和材质

当大批量进行模型渲染时，不可避免地会有大量重复几何体的创建，这时共用相同的几何体和材质可以减少内存和 GPU 的数据传输。

我们具体来看一个实例，比如你需要创建三百个简单的相同颜色的立方体模型，普通的实现方法如下：

```
for (let i = 0; i < 300; i++) {
    let geometry = new THREE.BoxGeometry(10, 10, 10);
    let material = new THREE.MeshLambertMaterial({color: 0x00ffff});
    let mesh = new THREE.Mesh(geometry, material);
    //随机位置
    mesh.position.set(THREE.Math.randFloatSpread(200), THREE.Math.randFloatSpread(200), THREE.Math.randFloatSpread(200));
    group.add(mesh);
}
```

上面代码创建了三百个相同的几何体和材质，该方法中有很多不必要的创建过程，增加运算的同时，还浪费了内存。

要解决这些性能问题，创建时我们可以共用相同的几何体和材质，改良后的代码如下所示：

```
let geometry = new THREE.BoxGeometry(10, 10, 10);
let material = new THREE.MeshLambertMaterial({color: 0x00ffff});
for (let i = 0; i < 300; i++) {
    let mesh = new THREE.Mesh(geometry, material);
    //随机位置
    mesh.position.set(THREE.Math.randFloatSpread(200), THREE.Math.randFloatSpread(200), THREE.Math.randFloatSpread(200));
    group.add(mesh);
}
```

利用上面代码，我们只需创建一套相同的几何体的顶点数据，不仅降低了内存消耗，还提高了添加运算效率。

### 模型删除时，材质和几何体也需从内存中清除

我们使用 `remove()` 将模型从场景内删除后，发现内存占用并没有太大变化。这是因为几何体和材质还保存在内存当中，这时需要手动调用 `dispose()` 方法将其从内存中删除。

下面为删除整个场景组的案例代码：

```
//删除group
function deleteGroup(name) {
    let group = scene.getObjectByName(name);
    if (!group) return;
    //删除掉所有的模型组内的mesh
    group.traverse(function (item) {
        if (item instanceof THREE.Mesh) {
            item.geometry.dispose(); //删除几何体
            item.material.dispose(); //删除材质
        }
    });

    scene.remove(group);
}
```

### 使用 merge 方法合并不需要单独操作的模型

在 Three.js 新版本中，将 merge 方法整合在了几何体上，主要应用于拥有大量几何体且材质相同的模型上。我们可以将多个几何体拼接成单个整体的几何体，从而达到节约性能的目的，但该做法的缺点则是失去了对单个模型的控制。

通过下面代码，我们了解下 merge 的使用方法：

```
//合并模型，则使用merge方法合并
var geometry = new THREE.Geometry();
//merge方法将两个几何体对象或者Object3D里面的几何体对象合并，(使用对象的变换)将几何体的顶点、面、UV 分别合并。
//THREE.GeometryUtils: .merge() has been moved to Geometry. Use geometry.merge( geometry2, matrix, materialIndexOffset ) instead. 如果新版本用老版本的会报这个错
for(var i=0; i<20000; i++){
    var cube = addCube(); //创建了一个随机位置的几何体模型
    cube.updateMatrix(); //手动更新模型的矩阵
    geometry.merge(cube.geometry, cube.matrix); //将几何体合并
}

scene.add(new THREE.Mesh(geometry, cubeMaterial));
```

我们再看一个案例：[点击这里](http://www.wjceo.com/blog/threejs/2018-03-14/123.html)。

在上面案例中，在不选中 combined 的前提下 ，选择 redraw 20000 个模型的话，一般只有十几帧的帧率。但如果选中了 combined，会发现渲染的帧率可达到满帧（60帧），性能得到了巨大提升。

### 在循环渲染中避免使用更新

几何体、材质、纹理等相关数据的更新尽量不要放到循环渲染中进行。循环渲染时，每一帧都会对 GPU 数据进行更新，这将引发很多不必要的数据更新过程。

接下来，我们了解下几何体、材质、纹理数据更新的相关属性。

- 几何体

```
geometry.verticesNeedUpdate = true; //顶点发生了修改
geometry.elementsNeedUpdate = true; //面发生了修改
geometry.morphTargetsNeedUpdate = true; //变形目标发生了修改
geometry.uvsNeedUpdate = true; //uv映射发生了修改
geometry.normalsNeedUpdate = true; //法向发生了修改
geometry.colorsNeedUpdate = true; //顶点颜色发生的修改
```

- 材质

```
material.needsUpdate = true；
```

- 纹理

```
texture.needsUpdate = true;
```

如果它们发生更新，则将其设置为 true，Three.js 会通过判断，将数据重新传输到显存当中，并将配置项重新修改为 false。这是一个十分影响运行效率的过程，我们尽量只在需要的时候修改，且不要放到 `render()` 方法当中循环设置。

### 只在需要的时候渲染

在没有操作的时候，一直循环渲染属于浪费资源。接下来我带给大家一个只在需要时渲染的方法。

首先在循环渲染中加入一个判断，如果判断值为 true 时，才可以循环渲染：

```
var renderEnabled;
function animate() {

    if (renderEnabled) {
        renderer.render(scene, camera);
    }

    requestAnimationFrame(animate);
}

animate();
```

然后设置一个延迟器函数，每次调用后，可以将 renderEnabled 设置为 true，并延迟三秒将其设置为 false，这个延迟时间大家可以根据需求来修改：

```
//调用一次可以渲染三秒
let timeOut = null;
function timeRender() {
    //设置为可渲染状态
    renderEnabled = true;
    //清除上次的延迟器
    if (timeOut) {
        clearTimeout(timeOut);
    }

    timeOut = setTimeout(function () {
        renderEnabled = false;
    }, 3000);
}
```

接下来，我们在需要的时候调用这个 `timeRender()` 方法即可，比如在相机控制器更新后的回调中：

```
controls.addEventListener('change', function(){
    timeRender();
});
```

如果相机位置发生变化，就会触发回调，开启循环渲染，更新页面显示。

如果我们添加了一个模型到场景中，直接调用重新渲染即可：

```
scene.add(mesh);
timeRender();
```

最后一个重点问题，就是材质的纹理为异步渲染，需要在图片添加完成后，触发回调。好在 Three.js 已经考虑到了这一点，Three.js 的静态对象 `THREE.DefaultLoadingManager` 的 onLoad 回调会在每一个纹理图片加载完成后触发回调。依靠它，我们可以在 Three.js 的每一个内容发生变更后触发重新渲染，且闲置状态下停止渲染。

```
//每次材质和纹理更新，触发重新渲染
THREE.DefaultLoadingManager.onLoad = function () {
    timeRender();
};
```