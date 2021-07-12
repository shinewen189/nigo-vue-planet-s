## 前言
笔者认为Three.js是一个伟大的框架，为什么这样说，因为它可以让我们轻易创造三维世界，甚至好像笔者写这遍教程，可以创造一个太阳系，在这个三维世界里你就是创世主。哈哈！好像说得有点夸！！

[三维太阳系完整效果](https://shinewen189.github.io/nigo-vue-planet/)

## 了解一些基本天文知识

学习创造这个三维太阳系之前先了解一下基本的天文知识：太阳系有“八大行星”，按照离太阳的距离从近到远，它们依次为水星、金星、地球、火星、木星、土星、天王星、海王星。八大行星自转方向多数也和公转方向一致。只有金星和天王星两个例外。金星自转方向与公转方向相反。而天王星则是在轨道上“横滚”的。例如地球自转一天是23.9小时，公转一年有365.2天 ，而相邻的火星自转一天是24.6小时 公转一年则有687天，其他行星也有不同的公转和自转信息，有了这些信息就可以定义一些基本规则

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a8e2f80f2be471cadef468084b78b2d~tplv-k3u1fbpfcp-watermark.image)
## 了解Three框架
Three的一些基本概念我在[用最简单方式打造Three.js 3D汽车展示厅](https://juejin.cn/post/6982435184838705159)一文也粗略介绍一下，为了让同学们加深理解，笔者就相对于太阳系来比如一下
1. 场景 `Sence` 相当于太阳系，宇宙中有无数星系，比如现在说的太阳系，后续还可以增加其他星系，那不是永远都加不完的呀 o(╥﹏╥)o
2. 相机 `Carma` 相当一枚哈勃天文望远镜
3. 几何体 `Geometry` 相当于太阳和八大行星
4. 控制 `Controls` 相当”创世者“的你

有了这几个概念我们就创建一些函数一一对应

## 完整效果


![屏幕录制2021-07-12 下午2.34.26.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae8d2023fdf648d1a341af0717c2be41~tplv-k3u1fbpfcp-watermark.image)


## 进入教程




先引入Three 要用到的对象 

```js
  import {
        Group,
        Mesh,
        MeshBasicMaterial,
        PerspectiveCamera,
        PointCloud,
        PointCloudMaterial,
        Scene,
        SphereGeometry,
        TextureLoader,
        Vector3,
        WebGLRenderer
    } from 'three'
     import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js'

```




### 场景(太阳系)，相机(哈勃天文望远镜)，控制(创世主)

```js
//场景
const setScene = () => {
        scene = new Scene()
        renderer = new WebGLRenderer({
            antialias: true,
        })
        renderer.setSize(innerWidth, innerHeight)
        document.querySelector('#planet').appendChild(renderer.domElement)
    }
//相机
const setCamera = () => {
        camera = new PerspectiveCamera(60, innerWidth / innerHeight, 1, 100000)
        camera.position.set(0, 500, 2000)
        camera.lookAt(scene.position)
    }
//控制    
 const setControls = () => {
        controls = new OrbitControls(camera, renderer.domElement)
    }        
```
### 创建一个太阳系的背景(繁星背景)
这个满天星效果是太阳系的背景，运用到Three的粒子系统，行星密度可自行调整

```js
 const starForge = () => {
        const starQty = 10000
        const geometry = new SphereGeometry(10000, 100, 50)
        const materialOptions = {}
        const starStuff = new PointCloudMaterial(materialOptions)
        geometry.vertices = []
        for (let i = 0; i < starQty; i++) {
            let starVertex = new Vector3()
            starVertex.x = Math.random() * 20000 - 10000
            starVertex.y = Math.random() * 20000 - 10000
            starVertex.z = Math.random() * 20000 - 10000
            geometry.vertices.push(starVertex)
        }
        const stars = new PointCloud(geometry, starStuff)
        scene.add(stars)
    }
```
效果如下图:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f38407fff984a2aaa2642548d4d7be4~tplv-k3u1fbpfcp-watermark.image)


### 创建太阳和行星前先说说行星自转同时公转的规律

![屏幕录制2021-07-12 上午11.23.20.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da23575894064bd2a5fe42ba80a4c796~tplv-k3u1fbpfcp-watermark.image)


旋转方式：实现旋转功能有三种方式
1. 旋转照相机 
2. 旋转整个场景(Scene)
3. 旋转单个元素

因为我们这里每个行星的自转速度，公转速度都不一样。所以设置整体旋转并不可行，所以要给每个元素设置不同的旋转属性。

行星需要让它们围绕着太阳转，就要先给它们自身设置一个位置偏移。以水星为例：`mercury.position.x -= 300`，而此时设置`mercury.rotation.y`属性，它就会实现自转。因为它的Y轴位置已经改变了。

当我们移动了`mercury`时，`mercuryParent`的位置是没有变的，自然它的Y轴也不会变，又因为`mercuryParent`包含了`mercury`，所以旋转`mercuryParent`时，`mercury`也会绕着初始的默认Y轴旋转。所以设置那么多组，是为了实现每颗行星不同的速度和公转的同时自转。至于设置以下代码数值就根据 行星自转一天、公转一年用多少时间来大概定义一下。


```js
//设置公转函数
const revolution = () => {
        mercuryParent.rotation.y += 0.015
        venusParent.rotation.y += 0.0065
        earthParent.rotation.y += 0.05
        marsParent.rotation.y += 0.03
        jupiterParent.rotation.y += 0.001
        saturnParent.rotation.y += 0.02
        uranusParent.rotation.y += 0.09
        neptuneParent.rotation.y += 0.001
    }
    //设置自转函数
    const selfRotation = () => {
        sun.rotation.y += 0.004
        mercury.rotation.y += 0.002
        venus.rotation.y += 0.005
        earth.rotation.y += 0.01
        mars.rotation.y += 0.01
        jupiter.rotation.y += 0.08
        saturn.rotation.y += 1.5
        uranus.rotation.y += 1
        neptune.rotation.y += 0.1
    }
```

### 创建太阳和八大行星
创建星系用到几何球体+纹理贴图

首先介绍一下太阳如何创造，利用 `SphereGeometry`创建球体，利用`MeshBasicMaterial`添加纹理，太阳是质量是最大的，所以设置球体的时候数值是最大。 下图是太阳的纹理贴图

![sun.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76f8857b0a174f0e95f0f96bfed21cdc~tplv-k3u1fbpfcp-watermark.image)

```js
  // 添加设置太阳
    let sun, sunParent
    const setSun = () => {
        sun = new Group()//建立一个组
        sunParent = new Group()
        scene.add(sunParent) //把组都添加到场景里
        loader.load('src/assets/universe/sun.jpg', (texture) => {
            const geometry = new SphereGeometry(500, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            sun.add(mesh)//添加到组里
            sunParent.add(sun)
        })
    }

```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95447ada34044e9ca44bd051aadd3363~tplv-k3u1fbpfcp-watermark.image)

按照离太阳最近一个接一个创建 
### 创建水星 
水星离太阳最近，质量是所有行星中最小，所以球体数值也给一个最小的数值。 下图水星纹理贴图
![mercury.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d01d320b56c54c7ea61298c813f8dadc~tplv-k3u1fbpfcp-watermark.image)

```js
 let mercury, mercuryParent
    const setMercury = () => {
        mercury = new Group()
        mercuryParent = new Group()
        scene.add(mercuryParent)
        loader.load('src/assets/universe/mercury.jpg', (texture) => {
            const geometry = new SphereGeometry(25, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            mercury.position.x -= 600
            mercury.add(mesh)//添加到组里
            mercuryParent.add(mercury)
        })
    }
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/413eed35d8e148918a4726acdebd3893~tplv-k3u1fbpfcp-watermark.image)

### 创建金星

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98dda416cf0e460faf4945402e903c2b~tplv-k3u1fbpfcp-watermark.image)
O(∩_∩)O哈哈~ 应该是下图，这张才是金星行星的纹理贴图，千万不要用错哟！！
    
![venus.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e5cfaeee859497eb07c35267eb8e5eb~tplv-k3u1fbpfcp-watermark.image)


```js
  let venus, venusParent
    const setVenus = () => {
        venus = new Group()//建立一个组
        venusParent = new Group()
        scene.add(venusParent)
        loader.load('src/assets/universe/venus.jpg', (texture) => {
            const geometry = new SphereGeometry(100, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            venus.position.x -= 700
            venus.add(mesh)//添加到组里
            venusParent.add(venus)
        })
    }

```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37123027fac44c89b1091f5ddb0e935f~tplv-k3u1fbpfcp-watermark.image)

### 地球
怎可以没有我们的家园呢，这么美丽的家园要好好保护它啊！！

![earth.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dffd90ba5e2c4515830eac662be0f7f5~tplv-k3u1fbpfcp-watermark.image)

```js
let earth, earthParent
    const setEarth = () => {
        earth = new Group()//建立一个组
        earthParent = new Group()
        scene.add(earthParent)
        loader.load('src/assets/universe/earth.jpg', (texture) => {
            const geometry = new SphereGeometry(100, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            earth.position.x -= 900
            earth.add(mesh)//添加到组里
            earthParent.add(earth)
        })
    }
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/835db2879236498e93576e98e38bec82~tplv-k3u1fbpfcp-watermark.image)

### 火星、木星、土星、天王星、海王星
接下来的行星设置都是大同小异、只是公转、自转、和行星大小的设置不同。

接着对应行星的纹理贴图也一一发给大家

火星的纹理贴图

![mars.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/421f1c2284374c05873f3a15560edc3c~tplv-k3u1fbpfcp-watermark.image)

木星的纹理贴图 

![jupiter.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ffd78d079524bb1a67a59a5382dfb1c~tplv-k3u1fbpfcp-watermark.image)

土星的纹理贴图


![saturn.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba259c8d1ed843bf97b51c57b26627bf~tplv-k3u1fbpfcp-watermark.image)

天王星的纹理贴图

![uranus.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03a6a5565ab6405ba80aeab0c53ef6f2~tplv-k3u1fbpfcp-watermark.image)

海王星的纹理贴图
![neptune.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6b1d0fa5d00468b96865d4df9871ac5~tplv-k3u1fbpfcp-watermark.image)


## 最后
一个三维太阳系就创造出来啦，这个例子也是很适合刚入门three.js的同学，目的也是提高对三维的兴趣，提高自身成就感。当然在这列子上我们还可以增加一些功能，比如定位标注一些行星的信息，点击行星可以进入星球内部，利用天空盒子做一个VR全景效果，等等。另外小弟找这些行星纹理贴图也不易，特别找金星的时候😏，希望大家如果喜欢这篇文章能给个赞小弟，当鼓励一下。以后小弟必定为大家创作更多好文，谢谢啦！！^_^

![屏幕录制2021-07-12 下午2.34.26.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae8d2023fdf648d1a341af0717c2be41~tplv-k3u1fbpfcp-watermark.image)


## 上完整代码

```js
<template>
    <div id="planet">

    </div>
</template>
<script setup>
    import {onMounted} from 'vue'
    import {
        Group,
        Mesh,
        MeshBasicMaterial,
        PerspectiveCamera,
        PointCloud,
        PointCloudMaterial,
        Scene,
        SphereGeometry,
        TextureLoader,
        Vector3,
        WebGLRenderer
    } from 'three'
    import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js'

    const loader = new TextureLoader() //引入模型的loader实例
    let scene, camera, renderer, group, controls // 定义所有three实例变量

    // 创建场景
    const setScene = () => {
        scene = new Scene()
        renderer = new WebGLRenderer({
            antialias: true,
        })
        renderer.setSize(innerWidth, innerHeight)
        document.querySelector('#planet').appendChild(renderer.domElement)
    }

    // 创建相机
    const setCamera = () => {
        camera = new PerspectiveCamera(60, innerWidth / innerHeight, 1, 100000)
        camera.position.set(0, 500, 2000)
        camera.lookAt(scene.position)
    }

    // 设置模型控制
    const setControls = () => {
        controls = new OrbitControls(camera, renderer.domElement)
    }
    
    // 添加设置太阳
    let sun, sunParent
    const setSun = () => {
        sun = new Group()//建立一个组
        sunParent = new Group()
        scene.add(sunParent) //把组都添加到场景里
        loader.load('src/assets/universe/sun.jpg', (texture) => {
            const geometry = new SphereGeometry(500, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            sun.add(mesh)//添加到组里
            sunParent.add(sun)
        })
    }

    // 设置水星
    let mercury, mercuryParent
    const setMercury = () => {
        mercury = new Group()//建立一个组
        mercuryParent = new Group()
        scene.add(mercuryParent)
        loader.load('src/assets/universe/mercury.jpg', (texture) => {
            const geometry = new SphereGeometry(25, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            mercury.position.x -= 600
            mercury.add(mesh)//添加到组里
            mercuryParent.add(mercury)
        })
    }

    //设置金星
    let venus, venusParent
    const setVenus = () => {
        venus = new Group()//建立一个组
        venusParent = new Group()
        scene.add(venusParent)
        loader.load('src/assets/universe/venus.jpg', (texture) => {
            const geometry = new SphereGeometry(100, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            venus.position.x -= 700
            venus.add(mesh)//添加到组里
            venusParent.add(venus)
        })
    }
    //设置地球
    let earth, earthParent
    const setEarth = () => {
        earth = new Group()//建立一个组
        earthParent = new Group()
        scene.add(earthParent)
        loader.load('src/assets/universe/earth.jpg', (texture) => {
            const geometry = new SphereGeometry(100, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            earth.position.x -= 900
            earth.add(mesh)//添加到组里
            earthParent.add(earth)
        })
    }
//设置火星
    let mars, marsParent
    const setMars = () => {
        mars = new Group()//建立一个组
        marsParent = new Group()
        scene.add(marsParent)
        loader.load('src/assets/universe/mars.jpg', (texture) => {
            const geometry = new SphereGeometry(85, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            mars.position.x -= 1200
            mars.add(mesh)//添加到组里
            marsParent.add(mars)
        })
    }

    // 设置木星
    let jupiter, jupiterParent
    const setJupiter = () => {
        jupiter = new Group()//建立一个组
        jupiterParent = new Group()
        scene.add(jupiterParent)
        loader.load('src/assets/universe/jupiter.jpg', (texture) => {
            const geometry = new SphereGeometry(150, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            jupiter.position.x -= 1500
            jupiter.add(mesh)//添加到组里
            jupiterParent.add(jupiter)
        })
    }

    // 设置土星
    let saturn, saturnParent
    const setSaturn = () => {
        saturn = new Group()//建立一个组
        saturnParent = new Group()
        scene.add(saturnParent)
        loader.load('src/assets/universe/saturn.jpg', (texture) => {
            const geometry = new SphereGeometry(120, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            saturn.position.x -= 1800
            saturn.add(mesh)//添加到组里
            saturnParent.add(saturn)
        })
    }

    //设置天王星
    let uranus, uranusParent
    const setUranus = () => {
        uranus = new Group()
        uranusParent = new Group()
        scene.add(uranusParent)
        loader.load('src/assets/universe/uranus.jpg', (texture) => {
            const geometry = new SphereGeometry(50, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            uranus.position.x -= 2100
            uranus.add(mesh)//添加到组里
            saturnParent.add(uranus)
        })
    }

    //设置海王星
    let neptune, neptuneParent
    const setNeptune = () => {
        neptune = new Group()
        neptuneParent = new Group()
        scene.add(neptuneParent)
        loader.load('src/assets/universe/neptune.jpg', (texture) => {
            const geometry = new SphereGeometry(50, 20, 20) //球体模型
            const material = new MeshBasicMaterial({map: texture}) //材质 将图片解构成THREE能理解的材质
            const mesh = new Mesh(geometry, material)  //网孔对象 第一个参数是几何模型(结构),第二参数是材料(外观)
            neptune.position.x -= 2300
            neptune.add(mesh)//添加到组里
            neptuneParent.add(neptune)
        })
    }

//监听浏览器改变大小时重新渲染
    function onWindowResize() {
        const WIDTH = window.innerWidth,
            HEIGHT = window.innerHeight
        camera.aspect = WIDTH / HEIGHT
        camera.updateProjectionMatrix()
        renderer.setSize(WIDTH, HEIGHT)
    }

//设置公转函数
    const revolution = () => {
        mercuryParent.rotation.y += 0.015
        venusParent.rotation.y += 0.0065
        earthParent.rotation.y += 0.05
        marsParent.rotation.y += 0.03
        jupiterParent.rotation.y += 0.01
        saturnParent.rotation.y += 0.02
        uranusParent.rotation.y += 0.09
        neptuneParent.rotation.y += 0.01
    }

    //设置自转
    const selfRotation = () => {
        sun.rotation.y += 0.004
        mercury.rotation.y += 0.002
        venus.rotation.y += 0.005
        earth.rotation.y += 0.01
        mars.rotation.y += 0.01
        jupiter.rotation.y += 0.08
        saturn.rotation.y += 1.5
        uranus.rotation.y += 1
        neptune.rotation.y += 0.1
    }

    // 设置太阳系背景
    const starForge = () => {
        const starQty = 10000
        const geometry = new SphereGeometry(10000, 100, 50)
        const materialOptions = {}
        const starStuff = new PointCloudMaterial(materialOptions)
        geometry.vertices = []
        for (let i = 0; i < starQty; i++) {
            let starVertex = new Vector3()
            starVertex.x = Math.random() * 20000 - 10000
            starVertex.y = Math.random() * 20000 - 10000
            starVertex.z = Math.random() * 20000 - 10000
            geometry.vertices.push(starVertex)
        }
        const stars = new PointCloud(geometry, starStuff)
        scene.add(stars)
    }


    // 循环场景 、相机、 位置更新
    const loop = () => {
        requestAnimationFrame(loop)
        revolution()
        selfRotation()
        renderer.render(scene, camera)
        camera.lookAt(scene.position)
    }

  //初始化所有函数
    const init = () => {
        setScene() //设置场景
        setCamera() //设置相机
        setSun() // 设置太阳
        setMercury() //设置水星
        setVenus() //设置金星
        setEarth() // 地球
        setMars() //火星
        setJupiter() // 木星
        setSaturn() // 土星
        setUranus()// 天王星
        setNeptune()//海王星
        starForge()//设置满天星背景
        setControls() //设置可旋转控制
        loop() // 循环动画
    }

    onMounted(init)
    window.addEventListener('resize', onWindowResize)

</script>



```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/885047ce19d045228e779ca67b98bbff~tplv-k3u1fbpfcp-watermark.image)
