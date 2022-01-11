<template>
  <div ref="container"></div>
</template>

<script>
import dat from 'dat.gui'
import * as THREE from 'three'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
let scene;
export default {
  data() {
    return {
      // scene: null,
      light: null,
      camera: null,
      renderer: null,
      cube: null,
      control: null,
    }
  },
  mounted() {
    this.draw()
  },
  methods: {
    draw() {
      this.initGui()
      this.initRenderer()
      this.initScene()
      this.initCamera()
      this.initLight()
      this.initModel()
      this.initControl()
      this.animate()
    },
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({
        antialias: true
      })
      // this.renderer.setClearColor(0xffffff)
      // this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize( window.innerWidth, window.innerHeight)
      this.renderer.shadowMap.enabled = true
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
      this.$refs.container.appendChild( this.renderer.domElement );
    },
    initScene() {
      scene = new THREE.Scene()
    },
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 1000)
      this.camera.position.set(0, 100, 200)
      this.camera.lookAt(new THREE.Vector3(0, 0, 0))
    },
    initGui() {
       const control = {
        positionX: 0,
        positionY: 0,
        positionZ: 0
      }
      let gui = new dat.GUI()
      gui.add(control, 'positionX').onChange(updatePosition)
      gui.add(control, 'positionY', -1, 1).onChange(updatePosition)
      gui.add(control, 'positionZ', -1, 1).onChange(updatePosition)
      function updatePosition() {
        // this.cube.position.set(control.positionX, control.positionY, control.positionZ)
      }
    },
    initLight() {
      scene.add(new THREE.AmbientLight(0x404040 ))
      this.light = new THREE.DirectionalLight(0xffffff, 0.5)
      // light.color.set(0x000000)
      // light.intensity = 2.0
      this.light.position.set(40,60,10)
      // this.light.position.multiplyScalar(0.3)
      this.light.shadow.camera.near = 1; //产生阴影的最近距离
      this.light.shadow.camera.far = 400; //产生阴影的最远距离
      this.light.shadow.camera.left = -50; //产生阴影距离位置的最左边位置
      this.light.shadow.camera.right = 50; //最右边
      this.light.shadow.camera.top = 50; //最上边
      this.light.shadow.camera.bottom = -50; //最下面

        //这两个值决定生成阴影密度 默认512
      this.light.shadow.mapSize.height = 1024;
      this.light.shadow.mapSize.width = 1024;
      this.light.castShadow = true;
      scene.add(this.light)

    },
    initModel() {
      const planeGeometry = new THREE.PlaneGeometry(100, 100)
      const planeMeterial = new THREE.MeshLambertMaterial({color: 0xaaaaaa, side: THREE.DoubleSide})
      const plane = new THREE.Mesh(planeGeometry, planeMeterial)
      plane.rotation.x = -0.5 * Math.PI
      plane.position.y = -0.1
      plane.receiveShadow = true
      scene.add(plane)

      let loader = new GLTFLoader()
      loader.load('static/model/scene.gltf', (gltf)=> {
        let mesh = gltf.scene
        mesh.scale.set(.1, .1, .1)
        mesh.position.set(0, 0, 0)
        scene.add(mesh)
      }, () => {

      }, (err) => {
        console.log(err)
      })
    },
    initControl(){
      this.control = new OrbitControls(this.camera, this.renderer.domElement)
      console.log(this.$refs.container)

    },
    render() {
      this.control.update()
      this.renderer.render(scene, this.camera)
    },
    animate() {
      this.render()
      requestAnimationFrame(this.animate)
    }
  }
}
</script>