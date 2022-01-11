<template>
  <div ref="box"></div>
</template>
<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import dat from 'dat.gui'
let scene;
const state = {
  animateBones: false
};
export default {
  data() {
    return {
      renderer: null,
      camera: null,
      light: null,
      controls: null,
      mesh: null,
      gui: null,
      state: null,
    }
  },
  mounted() {
    this.draw()
  },
  methods: {
    draw() {
      this.initScene()
      this.initRender()
      this.initScene()
      this.initCamera()
      this.initLight()
      this.initModel()
    },
    initGui() {

    },
    initScene() {
      scene = new THREE.Scene()
    },
    initRender() {
      this.renderer = new THREE.WebGLRenderer({antialias: true})
      this.renderer.setSize( window.innerWidth, window.innerHeight)
      this.renderer.shadowMap.enabled = true
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
      this.$refs.box.appendChild(this.renderer.domElement)
    },
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
      this.camera.position.set(0, 20, 50)
      this.camera.lookAt(new THREE.Vector3(0, 0, 0))
    },
    initLight() {
      scene.add(new THREE.AmbientLight(0x444444))
      this.light = new THREE.PointLight(0xffffff)
      light.position.set(0, 50, 0)
      this.light.castShadow = true
      scene.add(this.light)
    },
    initModel() {
      let helper = new THREE.AxesHelper(50)
      scene.add(helper)

      const segmentHeight = 6
      const segmentCount = 4
      const height = segmentHeight * segmentCount
      const halfHeight = height * 0.5
      const sizing = {
        segmentHeight,
        segmentCount,
        height,
        halfHeight
      }
      this.createGeometry(sizing)
    },
    createGeometry(sizing) {
      let geometry = new THREE.CylinderGeometry(5, 5, sizing.height, 8, sizing.segmentCount * 3, true)
      const len = geometry.faces.length;
      for (let i = 0; i < len; i++) {
        let face = geometry.faces[i].clone()
        face.materialIndex = 1
        geometry.faces.push(face)
      }
      geometry.faceVertexUvs = []
      for (let i = 0; i < geometry.vertices.length; i++) {
        let vertex = geometry.vertices[i]
        let y = (vertex.y + sizing.halfHeight)
        let skinIndex = Math.floor(y / sizing.segmentHeight)
        let skinWeight = (y % sizing.segmentHeight) / sizing.segmentHeight
        geometry.skinIndices.push(new THREE.Vector4(skinIndex, skinIndex + 1, 0, 0))
        geometry.skinIndices.push(new THREE.Vector4(1 - skinWeight, skinWeight, 0, 0))
      }
      return geometry
    },
    createBones(sizing) {
      let bones = []
      let prevBone = new THREE.Bone()
      bones.push(prevBone)
      prevBone.position.y = -sizing.halfHeight
      for (let i = 0; i < sizing.segmentCount; i++) {
        let bone = new THREE.Bone()
        bone.position.y = sizing.segmentHeight
        bones.push(bone)
        prevBone.add(bone)
        prevBone = bone
      }
      return bones
    },
    createMesh(geometry, bones) {
      let material = new THREE.MeshPhongMaterial({
        skinning: true,
        color: 0x156289,
        emissive: 0x072534,
        side: THREE.DoubleSide
      })
      let lineMaterial = new THREE.MeshBasicMaterial({
        skinning: true,
        wireframe: true
      })
      let mesh = new THREE.SkinnedMesh(geometry, [material, lineMaterial])
      let skeleton = new THREE.Skeleton(bones)
      mesh.add(bones[0])
      mesh.bind(skeleton)
      
      let skeletonHelper = new THREE.SkeletonHelper(mesh)
      skeletonHelper.material.linewidth = 2
      scene.add(skeletonHelper)
      return mesh
    },
    initControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.enableDamping = true
      this.controls.enableZoom = true
      this.controls.autoRotate = false
      this.controls.autoRotateSpeed = 0.5
      this.controls.minDistance = 2000
      this.controls.enablePan = true
    },
    render() {
      const time = Date.now() * 0.001
      if(state.animateBones) {
        for(let i = 0; i < this.mesh.skeleton.bones.length; i++) {
          this.mesh.skeleton.bones[i].rotation.z = Math.sin(time) * 2 / this.mesh.skeleton.bones.length
        }
      }
      this.controls.update()
    },
    animate() {
      this.render()
      this.renderer.render(scene, this.camera)
      requestAnimationFrame(this.animate)
    }
  }
}
</script>