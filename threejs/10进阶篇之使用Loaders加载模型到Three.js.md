# 第10课：进阶篇之使用 Loaders 加载模型到 Three.js

现在市面上的 3D 模型有上百种，每一种格式都有不同的用途，不同的功能和复杂程度。尽管 Three.js 提供了很多的加载器，但选择正确的格式和工作流程将为以后的工作节省大量时间和成本。而且某些格式难以使用，效率低下，甚至有些目前还未完全被支持。

### 推荐使用的模型格式

官方推荐我们使用的 3D 模型的格式为 glTF，由于 glTF 专注于传输，因此它的传输和解析的速度都很快。glTF 模型的功能包括网格、材质、纹理、蒙皮、骨骼、变形动画、骨骼动画、灯光以及相机。

如果当前的首选不是 glTF 格式，那么推荐使用 Three.js 定期维护并且流行的格式 FBX、OBJ 或者 COLLADA 格式，Three.js 也有自己独有的 JSON 格式。我们接下来将介绍这五种格式。

### Three.js 的 JSON 格式

这里的 JSON 格式指的是 Three.js 可以将其转换为场景 3D 对象的 JSON 格式模型。这种格式内部一般必有的四项为：

- metadata：当前模型的相关信息以及生成的工具信息；
- geometries：存储当前模型所使用的几何体的数组；
- materials：存储当前模型所使用的材质的数组；
- object：当前模型的结构以及标示所应用到的材质和几何体标示。

所有的模型网格，几何体和材质都有一个固定的 UUID 标识符，在 JSON 格式中均通过 UUID 引用。

#### 3D 对象转成 JSON

所有的 `THREE.Object3D` 对象都可以转成 JSON 字符串保存成为文件，但我们不能直接将对象转成 JSON，因为 JSON 无法保存函数。Three.js 给我们提供了一个 toJSON() 的方法来让我们将其转换为可存储的 JSON 格式。

```
var obj = scene.toJSON(); //将整个场景的内容转换成为 JSON 对象
var obj = group.toJSON(); //将一个模型组转成 JSON 对象
var obj = mesh.toJSON(); //将一个模型网格转成 JSON 对象
var JSONStr = JSON.stringify(obj); //将 JSON 对象转换成 JSON 字符串
```

按照这种方式，我们就可以将生成的场景模型保存为文件。

#### 使用 ObjectLoader 加载 JSON 模型

这里我们将使用到 Three.js 内置的对象 `THREE.ObjectLoader` 加载模型。

直接加载 Three.js 生成的 JSON 对象，代码如下：

```
var obj = scene.toJSON(); //将整个场景的内容转换成为json对象

let loader = new THREE.ObjectLoader(); //实例化ObjectLoader对象
let scene = loader.parse(obj); //将json对象再转换成3D对象
```

加载外部的 JSON 文件：

```
let loader = new THREE.ObjectLoader(); //实例化ObjectLoader对象

//加载模型，并在回调中将生成的模型对象添加到场景中
loader.load("../js/models/json/file.json", function (group) {
    scene.add(group);
});
```

案例地址：[请点击这里](https://johnson2heng.github.io/GitChat-Three.js/10第十节 loaders/ObjectLoader.html)。

案例的右上角有四个点击事件：

- 添加模型：将在场景内随机生成一组立方体，每次都不相同。
- 导出模型：将场景内这一组立方体导出到本地 JSON 文件。
- 导入模型：可以选择将符合 JSON 文件作解析并导入到场景内。
- 加载模型：将加载服务器上面的一个 JSON 文件。

案例代码地址：[请点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/10第十节 loaders/ObjectLoader.html)。

### glTF 格式文件导入

glTF 格式的 3D 格式文件是官方推荐的使用格式，这种格式的文件我们可以在 Sketchfab 官网下载，这是一个国外比较知名的模型网站。

- 下载地址：[点击这里](https://sketchfab.com/models?date=week&features=downloadable&sort_by=-likeCount&type=models)。

我们可以在这里下载一些免费的 glTF 格式的模型。

我在官网上找了个不错的模型做了一个案例，[点击这里查看](https://johnson2heng.github.io/GitChat-Three.js/10第十节 loaders/GLTFLoader.html)。

![gltf](https://images.gitbook.cn/c4b67350-8394-11e8-81ea-e357bbe10665)

模型加载的速度会有些慢，大家可以等待下便能够看到这个小汽车。

接下来我们便讲解一下加载 glTF 模型的流程。

- 首先，将 GLTFLoader 加载器插件引入到页面，插件在官方包的 `/examples/js/loaders/` 文件夹中，一些文件的导入插件都在这个文件夹内，大家有兴趣可以研究一下：

```
<script src="../js/loaders/GLTFLoader.js"></script>
```

- 然后创建一个加载器：

```
var loader = new THREE.GLTFLoader();
```

- 使用加载器加载模型，并调节一下模型大小在场景内展示：

```
loader.load('../js/models/gltf/scene.gltf', function (gltf) {
    gltf.scene.scale.set(.1,.1,.1);
    scene.add(gltf.scene);
});
```

有时候我们可能不明白，我加载了一个模型，哪一部分是需要导入场景的模型呢？

这里我们可以先将解析的出来的模型对象打印一下，然后通过查看对象属性来了解导入场景内的对象，就比如 glTF 模型转换出来的对象的 scene 属性就是需要导入场景的对象，而 JSON 格式的模型是直接可以导入的对象。

模型加载案例源码：[请点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/10第十节 loaders/GLTFLoader.html)。

### FBX 模型导入

FBX 最大的用途是，在诸如 Max、Maya、Softimage 等软件间进行模型、材质、动作和摄影机信息的互导，这样就可以发挥 Max 和 Maya 等软件的优势。可以说，FBX 是最好的互导方案。

![fbx](https://images.gitbook.cn/e7ead4e0-8396-11e8-a90b-215c3565b75a)

接下来我们看一个 FBX 模型导入的案例，是我在网上下载的一个 FBX 格式的模型，导入到场景内的效果：[请点击这里查看](https://johnson2heng.github.io/GitChat-Three.js/10第十节 loaders/FBXLoader.html)。

接下来，我们看下它的实现过程。

首先我们需要导入 FBXLoader 插件，并且还需要额外增加一个解析二进制文件的插件 `inflate.min.js` ，不导入该文件的话，除了一些字符串存储的 FBX 格式，别的格式都会报错：

```
<script src="../js/loaders/inflate.min.js"></script>
<script src="../js/loaders/FBXLoader.js"></script>
```

创建 FBX 加载器：

```
var loader = new THREE.FBXLoader();
```

修改模型大小，并设置每个模型网格可以投射阴影：

```
loader.load('../js/models/fbx/file.fbx', function (fbx) {
    fbx.scale.set(.1,.1,.1);
    fbx.traverse(function (item) {
       if(item instanceof THREE.Mesh){
           item.castShadow = true;
           item.receiveShadow = true;
       }
    });
    scene.add(fbx);
});
```

这样就实现了 FBX 模型的导入。

案例源码地址：[请点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/10第十节 loaders/FBXLoader.html)。

### OBJ 格式模型导入

OBJ 文件是 3D 模型文件格式。由 Alias|Wavefront 公司为 3D 建模和动画软件 Advanced Visualizer 开发的一种标准，适合用于 3D 软件模型之间的互导，也可以通过 Maya 读写。

OBJ 文件是一种文本文件，可以直接用写字板打开进行查看和编辑修改，但不包含动画、材质特性、贴图路径、动力学、粒子等信息。

OBJ 文件的导出通常会和 MTL 格式一同导出，MTL 作为 OBJ 文件的附属文件，却有着 OBJ 文件需要的贴图材质，所以，我们通常使用时，将它们两个文件一同导入。

![obj](https://images.gitbook.cn/f0138980-8398-11e8-81ea-e357bbe10665)

这是我使用官网提供的一个模型制作的一个案例，查看地址：[请点击这里](https://johnson2heng.github.io/GitChat-Three.js/10第十节 loaders/OBJLoader.html)。

我们看下实现导入的过程。

首先，我们需要将 OBJLoader 插件和 MTLLoader 插件引入页面：

```
<script src="../js/loaders/OBJLoader.js"></script>
<script src="../js/loaders/MTLLoader.js"></script>
```

实例化 MTLLoader ：

```
//创建MTL加载器
var mtlLoader = new THREE.MTLLoader();
//设置文件路径
mtlLoader.setPath('../js/models/obj/');
```

如果有需要，我们还可以设置纹理文件夹地址：

```
//设置纹理文件路径
mtlLoader.setTexturePath('../js/models/obj/');
```

加载 MTL 文件，并在文件加载成功后，创建 OBJLoader 并设置对象应用当前的材质：

```
//加载mtl文件
mtlLoader.load('female02.mtl', function (material) {
    //创建OBJ加载器
    var objLoader = new THREE.OBJLoader();
    //设置当前加载的纹理
    objLoader.setMaterials(material);
    objLoader.setPath('../js/models/obj/');
    objLoader.load('female02.obj', function (object) {
        //添加阴影
        object.traverse(function (item) {
            if(item instanceof THREE.Mesh){
                item.castShadow = true;
                item.receiveShadow = true;
            }
        });
        //缩放
        object.scale.set(.3,.3,.3);
        scene.add(object);
    })
});
```

我们再去加载 OBJ 文件，加载成功的文件就是可以导入到场景内的 3D 对象。

案例源码查看地址：[请点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/10第十节 loaders/OBJLoader.html)。

### COLLADA 模型导入

COLLADA 是一个开放的标准，最初用于 3D 软件数据交换，由 SCEA 发起，现在则被许多著名厂家（如 Autodesk、XSI 等）支持。COLLADA 不仅仅可以用于建模工具之间的数据交换，也可以作为场景描述语言用于小规模的实时渲染。

COLLADA DOM 拥有丰富的内容用于表现场景中的各种元素，从多边形几何体到摄像机无所不包。我们可以通过 COLLADA DOM 库来进行场景文件的读取与处理操作。

![collada](https://images.gitbook.cn/88b97d60-839a-11e8-884a-35c5e2ab8361)

上面是我写的一个模型导入案例，案例地址：[请点击这里](https://johnson2heng.github.io/GitChat-Three.js/10第十节 loaders/ColladaLoader.html)。

我们看下实现步骤。

首先引入 ColladaLoader 插件：

```
<script src="../js/loaders/ColladaLoader.js"></script>
```

接着实例化 ColladaLoader 对象：

```
var loader = new THREE.ColladaLoader();
```

最后加载文件并调整文件大小，添加到场景内：

```
loader.load('../js/models/collada/elf.dae', function (collada) {

    //添加阴影
    collada.scene.traverse(function (item) {
        if(item instanceof THREE.Mesh){
            item.castShadow = true;
            item.receiveShadow = true;
        }
    });
    //缩放
    collada.scene.scale.set(5,5,5);
    scene.add(collada.scene);
});
```

案例源码查看：[请点击这里](https://github.com/johnson2heng/GitChat-Three.js/blob/master/10第十节 loaders/ColladaLoader.html)。

### 注意事项

**1. 如何知道，加载完成的模型需要将哪部分导入到场景？**

一般情况下都是将自身导入，比如 FBX，OBJ，JSON 等，还有一种，会在里面生成一个可导入 scene 属性，如 glTF 和 COLLADA 文件。如果导入哪部分你无法确定，你可以把模型对象打印到控制台查看，然后尝试往场景内导入。

**2.导入到场景内的模型无法查看，而且也没有报错，为什么？**

这种情况可能由多种情况造成的，一般主要有下面两种情况：

- 模型太小或者太大，这种情况可以尝试放大一千倍或者缩小一千倍来查看效果。
- 模型的位置太偏，根本不在相机照射范围内，这种问题我们可以将模型居中到相机照射的焦点位置查看，如何居中我们将在后面的课中讲解。