<template>
<div></div>
</template>

<script>
import * as THREE from "three"
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import * as d3 from "d3"
import Stats from "three/examples/jsm/libs/stats.module.js";
// import { operation } from "retry";

export default {
  name: 'HelloWorld',
  data(){
    return {
      scene:null,
      camera:null,
      renderer:null,
      map:null,
      // axesHelper:null,
      directionalLight:null,
      light:null,
      controls:null,
      stats:null,
      // textureLoader:null,
      loader:null,
      jsonData:null,
      deltaTime:null,
    }
  },
  mounted(){
    this.initFPS()
    this.initTHREE()
    this.addJsonData()
    this.animate()
  },
  methods:{
    // 显示帧率的窗口
    initFPS(){
      this.stats = new Stats();
      document.body.appendChild(this.stats.dom)
    },
    // 初始化场景，{相机、场景、渲染器}
    initTHREE(){
      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(90,window.innerHeight/window.innerWidth,0.1,1000)
      this.camera.position.set(0,0,30)
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.scene.add(this.camera)
      // 加入坐标轴
      // this.axesHelper = new THREE.AxesHelper(5);
      // this.scene.add(this.axesHelper)
      // 加载纹理
      this.map = new THREE.Object3D();
      this.directionalLight = new THREE.DirectionalLight(0xffffff,0.5)
      this.scene.add(this.directionalLight)
      this.light = new THREE.AmbientLight(0xffffff,0.5)
      this.scene.add(this.light)
      // 加载渲染器
      this.renderer = new THREE.WebGLRenderer({alpha:true})
      this.renderer.setSize(window.innerWidth,window.innerHeight)
      // 将渲染器添加到body
      document.body.appendChild(this.renderer.domElement);
      // 初始化控制器 可以旋转
      this.controls = new OrbitControls(this.camera,this.renderer.domElement)
      // 创建纹理加载器对象
      // this.textureLoader = new THREE.TextureLoader();
    },
    animate(){
      this.controls.update()
      this.stats.update()
      const clock = new THREE.Clock();
      this.deltaTime = clock.getDelta()
      requestAnimationFrame(this.animate)
      this.renderer.render(this.scene,this.camera)
    },
    addJsonData(){
      this.loader = new THREE.FileLoader();
      this.loader.load('https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json',(data)=>{
        this.jsonData = JSON.parse(data)
        // this.operationData(this.jsonData)
        const projection1 = d3.geoMercator().center([116, 39]).translate([0, 0]);
        const features = this.jsonData.features;
        features.forEach((feature) => {
          // 单个省份 对象
          const province = new THREE.Object3D();
          // 地址
          province.properties = feature.properties.name;
          const coordinates = feature.geometry.coordinates;
          const color = "#99ff99";
      
          if (feature.geometry.type === "MultiPolygon") {
            // 多个，多边形
            coordinates.forEach((coordinate) => {
              // console.log(coordinate);
              // coordinate 多边形数据
              coordinate.forEach((rows) => {
                const mesh = this.drawExtrudeMesh(rows, color, projection1);
                // const line = this.lineDraw(rows, color, projection1);
                // 唯一标识
                mesh.properties = feature.properties.name;
                // province.add(line);
                province.add(mesh);
              });
            });
          }
      
          if (feature.geometry.type === "Polygon") {
            // 多边形
            coordinates.forEach((coordinate) => {
              const mesh = this.drawExtrudeMesh(coordinate, color, projection1);
              // const line = this.lineDraw(coordinate, color, projection1);
              // 唯一标识
              mesh.properties = feature.properties.name;
      
              // province.add(line);
              province.add(mesh);
            });
          }
          this.map.add(province);
        });
        this.scene.add(this.map)
      })
    },
    // operationData(jsondata){
    //   // 以经纬度116，39为中心，进行投影的函数转换函数
    //   const projection1 = d3.geoMercator().center([116, 39]).translate([0, 0, 0]);
    //   const features = jsondata.features;
    //   features.forEach((feature) => {
    //     // 单个省份 对象
    //     const province = new THREE.Object3D();
    //     // 地址
    //     province.properties = feature.properties.name;
    //     const coordinates = feature.geometry.coordinates;
    //     const color = "#99ff99";
    
    //     if (feature.geometry.type === "MultiPolygon") {
    //       // 多个，多边形
    //       coordinates.forEach((coordinate) => {
    //         // console.log(coordinate);
    //         // coordinate 多边形数据
    //         coordinate.forEach((rows) => {
    //           const mesh = this.drawExtrudeMesh(rows, color, projection1);
    //           const line = this.lineDraw(rows, color, projection1);
    //           // 唯一标识
    //           mesh.properties = feature.properties.name;
    //           province.add(line);
    //           province.add(mesh);
    //         });
    //       });
    //     }
    
    //     if (feature.geometry.type === "Polygon") {
    //       // 多边形
    //       coordinates.forEach((coordinate) => {
    //         const mesh = this.drawExtrudeMesh(coordinate, color, projection1);
    //         const line = this.lineDraw(coordinate, color, projection1);
    //         // 唯一标识
    //         mesh.properties = feature.properties.name;
    
    //         province.add(line);
    //         province.add(mesh);
    //       });
    //     }
    //     this.map.add(province);
    //   });
    //   this.scene.add(this.map)
    // },
    lineDraw(polygon, color, projection){
      const lineGeometry = new THREE.BufferGeometry();
      const pointsArray = new Array();
      polygon.forEach((row) => {
        const [x, y] = projection(row);
        // 创建三维点
        pointsArray.push(new THREE.Vector3(x, -y, 9));
      });
      // 放入多个点
      lineGeometry.setFromPoints(pointsArray);
      // 生成随机颜色
      const lineColor = new THREE.Color(
        Math.random() * 0.5 + 0.5,
        Math.random() * 0.5 + 0.5,
        Math.random() * 0.5 + 0.5
      );
    
      const lineMaterial = new THREE.LineBasicMaterial({
        color: lineColor,
      });
      return new THREE.Line(lineGeometry, lineMaterial);
    },
    drawExtrudeMesh(polygon, color, projection){
      const shape = new THREE.Shape();
      // console.log(polygon, projection);
      polygon.forEach((row, i) => {
        const [x, y] = projection(row);
        // console.log(row, [x, y]);
        if (i === 0) {
          // 创建起点,使用moveTo方法
          // 因为计算出来的y是反过来，所以要进行颠倒
          shape.moveTo(x, -y);
        }
        shape.lineTo(x, -y);
      });
    
      // 拉伸
      const geometry = new THREE.ExtrudeGeometry(shape, {
        depth: 5,
        bevelEnabled: true,
      });
    
      // 随机颜色
      const randomColor = (0.5 + Math.random() * 0.5) * 0xffffff;
      const material = new THREE.MeshBasicMaterial({
        color: randomColor,
        transparent: true,
        opacity: 0.5,
      });
      return new THREE.Mesh(geometry, material);
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
