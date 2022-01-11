<template>
  <div ref="box"></div>
</template>
<script>
import dat from 'dat.gui'
import * as THREE from 'three'
import {BoxGeometry} from 'three'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"

let scene;
export default {
  data() {
    return {
      renderer: '',
      camera: '',
      // scene: '',
      cube: '',
      control: '',
      gui: '',
      guiData: ''
    }
  },
  mounted() {
    this.initDraw()
  },
  methods: {
    initDraw() {
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
      this.renderer = new THREE.WebGLRenderer({antialias: true})
      this.renderer.setSize( window.innerWidth, window.innerHeight)
      this.renderer.shadowMap.enabled = true
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
      this.$refs.box.appendChild(this.renderer.domElement)
    },
    initScene() {
      scene = new THREE.Scene()
    },
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
      this.camera.position.set(0, 20, 50)
      this.camera.lookAt(new THREE.Vector3(0, 0, 0))
    },
    initGui() {
      this.guiData = {
        influence1:0.01,
        influence2:0.01,
        update : function () {
          this.cube.morphTargetInfluences[0] = this.guiData.influence1;
          this.cube.morphTargetInfluences[1] = this.guiData.influence2;
        }
      };
      this.gui = new dat.GUI()
      this.gui.add(this.guiData, 'influence1', 0, 1).onChange(this.guiData.update)
      this.gui.add(this.guiData, 'influence2', 0, 1).onChange(this.guiData.update)
    },
    initLight() {
      let ambientLight = new THREE.AmbientLight('#111111')
      scene.add(ambientLight)

      let directionalLight = new THREE.DirectionalLight('#ffffff')
      directionalLight.position.set(40, 60, 10)

      directionalLight.shadow.camera.near = 1
      directionalLight.shadow.camera.far = 400
      directionalLight.shadow.camera.left = -50
      directionalLight.shadow.camera.right = 50
      directionalLight.shadow.camera.top = 50
      directionalLight.shadow.camera.bottom = -50

      directionalLight.shadow.mapSize.height = 1024
      directionalLight.shadow.mapSize.width = 1024

      directionalLight.castShadow = true
      scene.add(directionalLight)
    },
    initModel() {
      let planeGeometry = new THREE.PlaneGeometry(100, 100)
      let planeMaterial = new THREE.MeshLambertMaterial({color: 0xaaaaaa, side: THREE.DoubleSide})
      let plane = new THREE.Mesh(planeGeometry, planeMaterial)
      plane.rotation.x = -0.5 * Math.PI
      plane.position.y = -0.1
      plane.receiveShadow = true
      scene.add(plane)

      let cubeGeometry = new BoxGeometry(10, 10, 10)
      let cubeMaterial = new THREE.MeshLambertMaterial({morphTargets: true, color: 0x00fffff})
      
      let cubeTarget1 = new BoxGeometry(2, 10, 2)
      let cubeTarget2 = new BoxGeometry(8, 2, 8)

      cubeGeometry.morphTargets[0] = {name: 'target1', vertices: cubeTarget1.vertices}
      cubeGeometry.morphTargets[1] = {name: 'target2', vertices: cubeTarget2.vertices}
      cubeGeometry.computeMorphNormals()

      this.cube = new THREE.Mesh(cubeGeometry, cubeMaterial)
      this.cube.position.set(0, 10, 0)
      scene.add(this.cube)
      
    },
    initControl() {
      this.control = new OrbitControls(this.camera, this.renderer.domElement)
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