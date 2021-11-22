# 第08课：基础功能之 Points 粒子

这是基础的最后一节，通过前面的课程，基础的相关内容大家应该有了一个大概的认识，我们也能够实现一些简单的场景显示。在这一节，我们将学习到 Sprite 精灵和 Points 粒子，这两种对象共同点就是我们通过相机查看它们时，始终看到的是它们的正面，它们总朝向相机。通过它们的这种特性，我们可以实现广告牌的效果，或实现更多的比如雨雪、烟雾等更加绚丽的特效。

### Sprite 精灵

精灵由于一直正对着相机的特性，一般使用在模型的提示信息当中。通过 `THREE.Sprite` 创建生成，由于 `THREE.Sprite` 和 `THREE.Mesh` 都属于 `THREE.Object3D` 的子类，所以，我们操作模型网格的相关属性和方法大部分对于精灵都适用。和精灵一起使用的还有一个 `THREE.SpriteMaterial` 对象，它专门配合精灵的材质。注意，精灵没有阴影效果。

下面首先创建一个最简单的精灵：

```
var spriteMaterial = new THREE.SpriteMaterial( { color: 0xffffff } );
var sprite = new THREE.Sprite( spriteMaterial );
scene.add( sprite );
```

创建一个带有纹理图片的精灵：

```
var spriteMap = new THREE.TextureLoader().load( "sprite.png" );
var spriteMaterial = new THREE.SpriteMaterial( { map: spriteMap, color: 0xffffff } );
var sprite = new THREE.Sprite( spriteMaterial );
scene.add( sprite );
```

直接使用 canvas 创建精灵：

```
var spriteMap = new THREE.Texture(canvas);
var spriteMaterial = new THREE.SpriteMaterial( { map: spriteMap, color: 0xffffff } );
var sprite = new THREE.Sprite( spriteMaterial );
scene.add( sprite );
```

#### 相关属性

精灵特有的属性就一个，即 center 属性，值是一个二维向量 `THREE.Vector2`，这个属性意义就是当前设置的精灵位置点的位置处于精灵图中的位置。如果 center 的值是 `0,0`旋转的位置就在左下角，如果值为 `1,1` 的话，那旋转的位置则在精灵图的右上角，默认值是 `0.5,0.5`：

```
sprite.center.set(0.5, 0); //设置位置点处于精灵的最下方中间位置
```

接下来，我们查看一下精灵的案例：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/08第八节 points/sprite.html)。

案例代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/08第八节 points/sprite.html)。

案例效果从左到右依次是，普通的精灵，贴图纹理的精灵和 `canvas` 创建的精灵。

### points 粒子

粒子和精灵的效果是一样的，它们之间的区别是，如果当前场景内的精灵过多的话，就会出现性能问题。粒子的作用就是为解决很多精灵而出现的，我们可以使用粒子去模型数量很多的效果，比如下雨，下雪等，数量很多的时候就适合使用粒子来创建，相应的，提高性能的损失就是失去了对单个精灵的操作，所有的粒子的效果都是一样。总的来说，粒子就是提高性能减少的一些自由度，而精灵就是为了自由度而损失了一些性能。

#### 粒子的创建

粒子 `THREE.Points` 和精灵 `THREE.Sprite` 还有网格 `THREE.Mesh` 都属于 `THREE.Object3D` 的一个扩展，但是粒子有一些特殊的情况就是 `THREE.Points` 是它们粒子个体的父元素，它的位置设置也是基于 `THREE.Points` 位置而定位，而修改 `THREE.Points` 的 scale 属性只会修改掉粒子个体的位置。下面我们看下一个粒子的创建方法。创建一个粒子，需要一个含有顶点的几何体，和粒子纹理 `THREE.PointsMaterial` 的创建：

```
//球体
var sphereGeometry = new THREE.SphereGeometry(5, 24, 16);
var sphereMaterial = new THREE.PointsMaterial({color: 0xff00ff});
var sphere = new THREE.Points(sphereGeometry, sphereMaterial);
scene.add(sphere);
```

上面是通过球体几何体创建的一个最简单的粒子特效。

使用任何几何体都可以，甚至自己生成的几何体都可以，比如创建星空的案例：

```
var starsGeometry = new THREE.Geometry();
//生成一万个点的位置
for (var i = 0; i < 10000; i++) {
    var star = new THREE.Vector3();
    //THREE.Math.randFloatSpread 在区间内随机浮动* - 范围 / 2 *到* 范围 / 2 *内随机取值。
    star.x = THREE.Math.randFloatSpread(2000);
    star.y = THREE.Math.randFloatSpread(2000);
    star.z = THREE.Math.randFloatSpread(2000);
    starsGeometry.vertices.push(star);
}
var starsMaterial = new THREE.PointsMaterial({color: 0x888888});
var starField = new THREE.Points(starsGeometry, starsMaterial);
scene.add(starField);
```

使用一个空的几何体，将自己创建的顶点坐标放入，也可以实现一组粒子的创建。如果我们需要单独设置每一个粒子的颜色，可以给 geometry 的 colors 数组添加相应数量的颜色：

```
for (var i = 0; i < 10000; i++) {
    var star = new THREE.Vector3();
    //THREE.Math.randFloatSpread 在区间内随机浮动* - 范围 / 2 *到* 范围 / 2 *内随机取值。
    star.x = THREE.Math.randFloatSpread(2000);
    star.y = THREE.Math.randFloatSpread(2000);
    star.z = THREE.Math.randFloatSpread(2000);
    starsGeometry.vertices.push(star);

    starsGeometry.colors.push(new THREE.Color("rgb("+Math.random()*255+", "+Math.random()*255+", "+Math.random()*255+")")); //添加一个随机颜色
}
```

#### THREE.PointsMaterial 粒子的纹理

如果我们需要设置粒子的样式，还是需要通过设置 `THREE.PointsMaterial` 属性实现：

```
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff}); //设置了粒子纹理的颜色
```

我们还可以通过 PointsMaterial 的 size 属性设置粒子的大小：

```
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, size:4}); //粒子的尺寸改为原来的四倍
//或者直接设置属性
pointsMaterial.size = 4;
```

我们也可以给粒子设置纹理：

```
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, map:texture}); //添加纹理
```

默认粒子是不受光照影响的，我们可以设置 lights 属性为 true，让粒子受光照影响：

```
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, lights:true}); 
//或者
pointsMaterial.lights = true; //开启受光照影响
```

我们也可以设置粒子不受到距离的影响产生近大远小的效果：

```
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, sizeAttenuation: false}); 
//或者
pointsMaterial.sizeAttenuation = false; //关闭粒子的显示效果受距离影响
```

粒子的效果就介绍到这里，希望大家熟练了以后能够做出来各种各样的花哨效果。

接下来展示下我给大家准备的粒子案例：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/08第八节 points/points.html)。

代码查看地址：[点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/08第八节 points/points.html)。

这个案例和精灵的案例区别就是，将球体改成了粒子，然后将立方体修改成了带有 canvas 纹理的粒子，并且在背景里面添加了一万个粒子，希望大家根据代码，自己也能够写出来。