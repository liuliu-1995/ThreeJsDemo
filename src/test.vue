<template>
    <div id="app">
      <a id="modelChangeBtn" @click="changeModel">切换模型</a>
    </div>
  </template>
  
  <script>
  import * as THREE from 'three';
  import * as TWEEN from "@tweenjs/tween.js"
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  
  export default {
    name: 'App',
    data() {
      return {
        scene: null,
        camera: null,
        renderer: null,
        controls: null,
        pointsList: [],
        objIndex: 0,
        particalNum: 50000, // 设置粒子数量
        particalCompleteStep: 0,
        positions: [],
        tween: null,
        modeSrcList: [
          '../static/gltfModel/shiba.glb',
          '../static/gltfModel/phoenix_bird.glb',
        ],
      }
    },
    mounted() {
      this.init();
      this.render();
    },
    methods: {
      init() {
        // 创建场景
        this.scene = new THREE.Scene();
        //设置相机
        this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 10000);
        this.camera.position.z = 300;
        // 初始化渲染器
        this.renderer = new THREE.WebGLRenderer();
        this.renderer.setPixelRatio(window.devicePixelRatio);
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.controls = new OrbitControls(this.camera, this.renderer.domElement);
        this.controls.update();
        document.body.appendChild(this.renderer.domElement);
         // 创建环境光
        const ambientLight = new THREE.AmbientLight(0xff0000, 0.1);
        this.scene.add(ambientLight);
        // 初始化粒子
        this.positions = new Float32Array(this.particalNum * 3);
        const scales = new Float32Array(this.particalNum);
        for(let i=0; i<this.particalNum; i++){
          //弄一堆坐标在-400~400之间的随时数字;
          let x = THREE.MathUtils.randFloatSpread(800);
          let y = THREE.MathUtils.randFloatSpread(800);
          let z = THREE.MathUtils.randFloatSpread(800);
          this.positions[i] = x;
          this.positions[i+1] = y;
          this.positions[i+2] = z;
          const size = THREE.MathUtils.randFloatSpread(12);                 
          scales[i] = size;
        }
        this.bufferGeometry = new THREE.BufferGeometry();               
        this.bufferGeometry.setAttribute( 'position', new THREE.BufferAttribute(this.positions, 3));
        this.bufferGeometry.setAttribute( 'scale', new THREE.BufferAttribute(scales, 1));
        // 初始化贴图
        const textureLoader = new THREE.TextureLoader();
        textureLoader.crossOrigin = '';
        const mapDot = textureLoader.load('../static/textures/gradient.png');  // 圆点
        const material = new THREE.PointsMaterial({
          size: 1,
          sizeAttenuation: true,
          color: 0xffffff,
          transparent: true,
          opacity: 1,
          map: mapDot
        });
        const points = new THREE.Points(this.bufferGeometry, material);
        this.scene.add(points);
        // 加载3D模型
        this.loadObj(0);
        // this.controls.enablePan = false;
        // this.controls.enableZoom = false;
        // this.controls.maxPolarAngle = Math.PI / 3;
        //事件监听
        window.addEventListener('resize', this.onWindowResize, false);
      },
      loadObj(index){
        const objSrc = this.modeSrcList[index]
        this.particalCompleteStep = 0;
        const loader = new GLTFLoader();
        loader.load(objSrc, object => {
          console.log('object==================', object);
          const startPositions = this.bufferGeometry.getAttribute('position');                            
          const destPosition = this.combineBuffer(object.scene, 'position');    
          this.tweenObj(startPositions, destPosition)
        });
      },
      loadObj2() {
        const loader = new GLTFLoader();
        loader.load('../static/gltfModel/shiba.glb', gltf => {
           // 获取模型顶点数据，并将其转换成 Points 类型
           const positions = this.combineBuffer(gltf.scene, 'position')
           const geometry = new THREE.BufferGeometry();
           geometry.setAttribute('position', positions);
            const points = new THREE.Points(geometry, material);
            geometry.scale(250, 250, 250);
            geometry.center();
            geometry.rotateX(Math.PI*1.5);
            geometry.rotateY(Math.PI/1.8);
            geometry.translate(100, 0, 0);
            this.pointsList.push(points);
        }, undefined, function ( error ) {
          console.error( error );
        });
        loader.load('../static/gltfModel/phoenix_bird.glb', gltf => {
           // 获取模型顶点数据，并将其转换成 Points 类型
           const positions = this.combineBuffer(gltf.scene, 'position')
           const geometry = new THREE.BufferGeometry();
           geometry.setAttribute('position', positions);
           const material = new THREE.PointsMaterial({
              size: 1,
              sizeAttenuation: true,
              color: 0xffffff,
              transparent: true,
              opacity: 1,
              map: mapDot
            });
            const points = new THREE.Points(geometry, material);
            geometry.center();
            geometry.rotateY(3);
            this.pointsList.push(points);
        }, undefined, function ( error ) {
          console.error( error );
        });
      },
      combineBuffer(model, bufferName) {
        let count = 0;
        model.traverse(function (child) {
          if (child.isMesh) {
            const buffer = child.geometry.attributes[bufferName];
            count += buffer.array.length;
          }
        });
        const combined = new Float32Array(count);
        let offset = 0;
        model.traverse(function (child) {
          if (child.isMesh) {
            const buffer = child.geometry.attributes[bufferName];
            combined.set(buffer.array, offset);
            offset += buffer.array.length;
          }
        });
        return new THREE.BufferAttribute(combined, 3);
      },
      changeModel() {
        if (this.objIndex < this.pointsList.length - 1) {
          this.objIndex++;
        } else {
          this.objIndex = 0;
        }
        this.loadObj(this.objIndex);
      },
      tweenObj(startPositions, destPosition) {
        for(let i = 0; i< startPositions.count; i++) {
            this.tween = new TWEEN.Tween(this.positions);
            const cur = i % destPosition.count;
            this.tween.to({
                [i * 3]: destPosition.array[cur * 3],
                [i * 3 + 1]: destPosition.array[cur * 3 + 1],
                [i * 3 + 2]: destPosition.array[cur * 3 + 2]
            }, 1000 * Math.random());
            this.tween.easing(TWEEN.Easing.Exponential.InOut);
            this.tween.delay(3000);
            this.tween.onUpdate(() => {
                startPositions.needsUpdate = true;                    
            });              
            this.tween.start();
        }
    },
    onWindowResize() {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    render() {
        TWEEN.update();
        this.scene.rotation.y += 0.0014;
        this.renderer.render(this.scene, this.camera);
    },
    animate() {
        this.render();
        this.controls.enablePan = false;
        this.controls.enableZoom = false;
        this.controls.maxPolarAngle = Math.PI / 3;
        this.controls.update();
        requestAnimationFrame(this.animate);
    },
},
  }
  </script>
  
  <style>
  body {
    background-color: #000000;
    margin: 0px;
    overflow: hidden;
    font-family: Monospace;
    font-size: 13px;
    text-align: center;
    font-weight: bold;
    text-align: center;
  }
  #modelChangeBtn{
    position: fixed;
    left: 50%;
    bottom: 20px;
    font-size:18px;
    color: #fff;
    cursor: pointer;
  }
  </style>
  