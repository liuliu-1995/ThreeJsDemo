<template>
  <div id="app">
   
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
      glist: [],
      objIndex: 0,
    }
  },
  mounted() {
    this.init();
    this.animate();
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
      const ambientLight = new THREE.AmbientLight(0xff0000, 0.1)
      this.scene.add(ambientLight)
      // 加载3D模型
      this.loadObj();
      // 初始化首图
      this.tweenObj(this.objIndex);
      //事件监听
      document.addEventListener("mousewheel", this.onDocumentMouseWheel, false);
      document.addEventListener("keydown", this.onDocumentKeyDown, false);
      window.addEventListener('resize', this.onWindowResize, false);
    },
    loadObj() {
      const loader = new GLTFLoader();
      // 初始化贴图
      const textureLoader = new THREE.TextureLoader();
      textureLoader.crossOrigin = '';
      const mapDot = textureLoader.load('../static/textures/gradient.png');  // 圆点
      loader.load('../static/gltfModel/shiba.glb', gltf => {
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
          geometry.scale(200, 200, 200);
          geometry.center();
          geometry.rotateX(0.2);
          geometry.rotateY(1.5);
          geometry.rotateZ(1.5);
          this.glist.push(points)
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
          // geometry.rotateX(0.2);
          geometry.rotateY(3);
          // geometry.rotateZ(1.5);
          this.glist.push(points)
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
    tweenObj(index) {
      new TWEEN.Tween().onUpdate((obj) => { 
        this.scene.clear();
        this.scene.add(this.glist[this.objIndex]);
      })
      .start();
    },
    onDocumentKeyDown(event) {
      if (event.which == 40 && this.objIndex < 1) {
        this.objIndex++;
        this.tweenObj(this.objIndex);
      } else if (event.which == 38 && this.objIndex > 0) {
        this.objIndex--;
        this.tweenObj(this.objIndex);
      }
    },
    onDocumentMouseWheel(event) {
      this.camera.position.z += event.deltaY;
    },
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    render() {
      TWEEN.update();
      this.camera.lookAt(this.scene.position);
      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      this.controls.update();
      this.render();
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
</style>
