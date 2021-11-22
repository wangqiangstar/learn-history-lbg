### [学习文档](https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene)

1. 创建开发环境三种方式

   1. 编译threejs文件到本地引入

   2. [使用在线编辑器环境](https://codepen.io/_alvin/pen/vlGeB?editors=1010)

   3. [基于webpack和babel搭建本地开发环境可以在编辑器中语法提示](https://github.com/rossning92/t-rex)

      ```
      import * as THREE from 'three'
      ```

      

2. 创建一个场景对象scence

   ```
   const scene = new THREE.Scene()
   ```

   scence里面以树形结构存放了所有可见的、不可见的三维物体等等包括光源等

   1. 立方体Mesh
   2. 茶壶实体 Group
   3. 平行光 Directional Light

3. 创建一个立方体 Mesh(三角网)

   ```
   const geometry = new THREE.BoxGeometry()
   const material = new THREE.MeshBasicMaterial({color: 0x00ff00})
   const cube = new THREE.Mesh(geometry, material)
   scene.add(cube)
   ```

   立方体是一个Mesh（三角网），并且他由Geometry（几何形状）和Material（材质）组成

4. 创建一个摄像机Camera

   ```
   const camera = new THREE.PerspectiveCamera(
   	75,//fov 垂直方向上视角的大小，视角越大物体越远
   	window.innerWidth / window.innerHeight, // aspect ratio 表示投影平面的纵宽比，类似拉伸二位平面，取决于分辨率的纵宽比
   	0.1, // near 代表相机能够看到的最近的平面，超出这个平面的场景会被自动剪切掉
   	1000 // far 代表相机能够看到的最远的平面，超出这个平面的场景会被自动剪切掉
   );
   camera.position.z = 5;
   ```

   它决定了我们应当从什么角度、什么位置去观察这个三维场景

5. 定义一个Renderer渲染器

   ```
   const renderer = new THREE.WebGLRenderer();
   renderer.setSize(window.innerWidth, window.innderHeight);
   document.body.appendChild(renderer.domElement);
   ```

   渲染器负责将定义的场景树渲染成屏幕中看到的二位图

6. 创建一个循环

   ```
   const animae = () => {
   	requestAnimationFrame(animate);
   	cube.rotation.x +=0.01;
   	cube.rotation.y += 0.01;
   	
   	renderer.render(scene, camera);
   };
   animate();
   ```

   创建一个循环，并在循环中调用renderer的render()函数来渲染每一帧图像，但由于javascript是单线程的，如果我们直接写一个死循环会导致界面卡死，正确的做法是调用js中提供的requestAnimationFrame()函数，来请求浏览器定时回调这里的这个animate()函数，最后运行代码可以看到浏览器中一个绿色的一致旋转的立方体。



#### 简单修改场景及扩展

1. three.js中三维物体的创建

   1. 之前代码中立方体是使用Mesh(三角网)表示的，三角网由很多很多的三角形构成

      为什么要使用Mesh三角网呢？因为三角形的三个点可以在空间中唯一地确定一个平面，不存在歧义。因此也是最常见的用于表示三维物体的一种方式

   2. Mesh由两部分构成：几何形状Geometry和Material

      1. Geometry：集合形状，也就是看到的一个个三角形，以及构成他们的所有顶点

         - 另外threejs中内置了许多基本的几何图元（Primitives），上面代码中用到的立方体是BoxGeometry
           - BoxGeometry立方体
           - SphereGeometry球体
           - ConeGeometry圆锥体
           - CylinderGeometry圆柱体
           - 在文档地址中搜索，他们都已geometry结尾

      2. Material代表三维物体的材质

         - 不同材质在光照下会呈现不一样的效果
           - MeshBasicMaterial
           - MeshToonMaterial
           - MeshPhongMaterial
           - MeshLambertMaterial
           - 在文档地址中搜索，他们都已geometry结尾
         - 通过材质，可以给物体表面设定不同的颜色、光泽度或者给物体加上各式各样的贴图等等
         - 上面代码中用到的是最最基本的材质BasicMaterial，可以给他设置一个颜色，不过由于它不参与光照计算(Unlit)，因此不会产生阴影，使用这种材质的三维物体看上去也很不真实

      3. Lighting光照 

         - 要实现物体表面的光照效果，可以使用Phong / Lambert 这两种材质，他们实现了图形学中最最基本的光照模型，并且可以通过shininess来调节物体表面的光泽度

         - 通过map 给物体表面贴上纹理，通过Threejs提供的TextureLoader，并调用它的load函数加载一张纹理贴图

         - Threejs会根据顶点的UV坐标，将纹理映射到物体的表面上

           ```
           const material = new THREE.MeshBasicMaterial({
           	color: 0x00ff00,
           	shininess: 200,
           	map: new THREE.TextureLoader().load('doge.jpg')	
           })
           ```

         - 除了之前的两种材质，Threejs还提供效果更好的、也是被广泛使用的基于物理的材质--基于物理的渲染Physically Based Rendering

           - MeshStandardMaterial

           - MeshPhysicalMaterial

           - 他们的计算量更大，但是可以让场景看起来足够逼真

           - 比如这里先用map属性给物体表面贴上基本的纹理

           - 然后用roughnessMap来设定表面不同位置的粗糙程度

           - 接着可以用normalMap也就是法线贴图,来给每个像素点设置不同的法向量，法线会影响光照的计算，因此可以用它来模拟物体表面凹凸不平的效果

           - 进一步，可以使用displacementMap来指定一个位移贴图或者叫高度贴图，它会上下偏移物体表面的顶点坐标，从而做到真正的物体表面的起伏

           - 最后还可以通过aoMap（Ambient Occlusion）指定一个“环境光遮罩”的贴图，简单来说，它会让被遮蔽的区域（比如坑洞）看起来更暗，从而进一步提升场景的真实感

             ```
             const = material = new THREE.MeshStandardMaterial({
             	map: loadTexture('rock/albedo.png'),
             	roughnessMap: loadTexture('rock/roughness.png'),
             	normalMap: loadTexture('rock/normal.png'),
             	displacementMap: loadTexture('rock/height.png'),
             	aoMap: loadTexture('rock/ao.png')
             })
             ```

         - 除了屋里材质外，Threejs还提供了ToonMaterial用于实现卡通画效果的材质

           - MeshToonMaterial
           - 所有的材质均以Material结尾，可以在文档搜索，他们的参数和用法也都可以在文档中找到

      4. 几何变换Transformations

         - 创建了Mesh对象之后，自然可以修改它在空间中的位置

         - 可以用它的posotion属性来设置物体的三维坐标

         - 用scale来设置物体的缩放

         - 用rotation来旋转该物体

           - 比如下面会将立方体沿着Y轴旋转45度
           - 需要注意的是Threejs中所有的旋转角都是使用弧度制的

         - 可以使用Group对象将多个Mesh组合起来，这样可以通过修改Group对象的坐标，来对它下面的所有子对象同时进行坐标变换

           ```
           const geometry = new THREE.BoxGeometry()
           const material = new THREE.MeshBasicMaterial({color: 0x00ff00})
           const cube = new THREE.Mesh(geometry, material)
           scene.add(cube)
           
           cube.position.set(4, 0, 0);
           cube.scale.multiplyScalar(3);
           cube.rotation.y = Math.PI / 4;
           
           // create a group
           const group = new THREE.Group();
           group.add(sphere);
           group.add(cone)
           
           const animate = () => {
           	requestAnimationFrame(animate);
           	
           	// Rotate group of object
           	group.rotation.x += 0.01;
           	group.rotation.y += 0.01;
           	
           	renderer.render(scene, camera);
           }
           animate()
           ```

2. 摄像机Camera

   > 摄像机Camera决定了我们应当从什么角度去观察这个三维场景
   >
   > threejs中提供了两种不同类型的摄像机

   - 第一种最常用到的PerspectiveCamera 透视摄像机

     - 它和人眼成像的原理类似，看到的物体会呈现近大远小的透视效果，所以被广发应用在三维渲染中

     - 创建PerspectiveCamera时需要定义一个视锥Frustum

     - 随后在渲染的时候计算机会将视锥中的所有三维物体投影到我们的二维屏幕上

     - 视锥有四个参数决定

       ```
       const camera = new THREE.PerspectiveCamera(
       	60, // fov 垂直方向上视角的大小，视角越大物体越远
       	1920 / 1080, // aspect ratio 表示投影平面的纵宽比，类似拉伸二位平面，取决于分辨率的纵宽比
       	0.1, // near 代表相机能够看到的最近的平面，超出这个平面的场景会被自动剪切掉
       	5 // far 代表相机能够看到的最远的平面，超出这个平面的场景会被自动剪切掉
       );
       ```

   - 第二种相机时orthographicCamera正交投影摄像机

     - 它会将空间中的所有物体平行投射到投影面上，因此不会像透视相机那样呈现近大远小的效果，正交相机组要被用在像CAD这类需要精准测量物体尺寸的应用场景中，比如用正交投影可以轻易渲染出物体的三视图等等

     - 正交相机的视锥Frustum是一个长方体，

     - 因此要定义一个正交相机，我们需要指定它前后左右上下这六个面的位置

     - 创建相机之后同样可以使用position属性来修改相机的位置

       - 三种修改position的方法
         - camera.position.set(0, 1, 5)
         - camera.position.y += 2
         - camera.translateZ(2)
       - 修改完位置之后千万不要忘记调用updateMatrix()更新相机的变换矩阵
         - camera.updateMatrix()

     - 也可以使用rotation来调整相机的旋转角

       - camera.rotation.set(0, Math.PI / 4, 0)
       - camera.rotateX(Math.PI / 4)
       - camera.updateMatrix()

     - 还有个更方便的函数lookAt让相机看向空间中的某一个点

       - camera.lookAt(1, 0.4, 1)
       - camera.updateMatrix()

     - ```
       const camera = new THREE.OrthographicCamera(
       	-1, // left
       	1, // right
       	1, // top
       	1, // bottom
       	1, // near
       	100, // far
       );
       
       // 三种修改position的方法
       camera.position.set(0, 1, 5);
       camera.position.y += 2;
       camera.translateZ(2);
       ```

       

