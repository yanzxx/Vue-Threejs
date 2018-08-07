<template :style="{textAlign:'center'}">
  <div id="container" v-renderScene :style="{border:'1px solid red',height:'600px'}"></div>
</template>

<script>
 import GlobalVariable from '@/components/Global'
 import THREE from 'three.js'
 import $ from 'jquery'
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  directives:{
    renderScene:{
      inserted:function(el){
        var scene = GlobalVariable.scene;
        var camera = GlobalVariable.camera;
        var fov = GlobalVariable.fov;
        var renderer = GlobalVariable.renderer;
        var shouldrender = true;
        var isUserInteracting = false,
          isUserPanning = false,
          lon = 0,
          lat = 0,
          onPointerDownPointerX,
          onPointerDownPointerY,
          onPointerDownLon,
          onPointerDownLat;
        var mouseMoveDelta = {dx: 0, dy: 0};
        var currentMousePosition = {x: 0, y: 0};
        var previousMousePosition = {x: currentMousePosition.x, y: currentMousePosition.y};
        var mouseMoveVelocity = {x: 0, y: 0};
        scene = new THREE.Scene();

        fov = 45;

        camera = new THREE.PerspectiveCamera(

          fov,800/600, 0.1, 1000

        );
        camera.target = new THREE.Vector3(0, 0, 0);


        renderer = new THREE.WebGLRenderer();



        var clearColor = new THREE.Color(0xEEEEEE);



        renderer.setClearColor(clearColor);



        renderer.setSize(800,600);


        var planeGeometry  = new THREE.PlaneGeometry(60,20);

        var planeMaterial = new THREE.MeshBasicMaterial({

          color:0xcccccc

        });



        var plane = new THREE.Mesh(planeGeometry,planeMaterial);



        plane.rotation.x = -.5 * Math.PI;

        plane.position.x = 15;

        plane.position.y = 0;

        plane.position.z = 0;



        scene.add(plane);



        // 立方体

        var cubeGeometry = new THREE.BoxGeometry(4,4,4);

        var cubeMaterial = new THREE.MeshBasicMaterial({

          color:0xff0000, wireframe:true

        });

        var cube = new THREE.Mesh(cubeGeometry,cubeMaterial);



        cube.position.x = -4;

        cube.position.y = 3;

        cube.position.z = 0;

        var geometryChild = new THREE.BoxGeometry(2,2,2);

        var cube2 = new THREE.Mesh(geometryChild,cubeMaterial)

        //获取cube2相对于整个坐标系的位置；

        getWorldPosition(cube.position.clone(),cube);

        cube.add(cube2);





        scene.add(cube);



        // 球面体

        var sphereGeometry = new THREE.SphereGeometry(4,20,20);

        var sphereMaterial = new THREE.MeshBasicMaterial({color:0x7777ff,wireframe:true});

        var sphere = new THREE.Mesh(sphereGeometry,sphereMaterial);



        sphere.position.x = 20;

        sphere.position.y = 4;

        sphere.position.z = 2;



        scene.add(sphere);



        // 相机设置

        camera.position.x = -30;

        camera.position.y = 40;

        camera.position.z = 30;



        camera.lookAt(scene.position);



        $(el).append(renderer.domElement);

        el.addEventListener('mousemove', onContainerMouseMove, false);

        el.addEventListener('mousedown', onContainerMouseDown, false);

        el.addEventListener('mouseup', onContainerMouseUp, false);

        //renderer.domElement.addEventListener('mouseout', onContainerMouseOut, false);

        el.addEventListener('mousewheel', onContainerMouseWheel, false);

        //renderer.domElement.addEventListener('DOMMouseScroll', onContainerMouseWheel, false);




        function onContainerMouseDown(){
          isUserInteracting =true;
          isUserPanning = false;
          mouseMoveVelocity = {x: 0, y: 0};

          onPointerDownPointerX = event.clientX;
          onPointerDownPointerY = event.clientY;

          onPointerDownLon = lon;
          onPointerDownLat = lat;
          shouldrender =true;
        }

        function onContainerMouseMove(){
          if(isUserInteracting){
            var dx = event.clientX - onPointerDownPointerX;
            var dy = event.clientY - onPointerDownPointerY;
            mouseMoveDelta.x = dx;
            mouseMoveDelta.y = dy;
            //是否拖动
            if (Math.abs(dx) > 2 || Math.abs(dy) > 2) {
              isUserPanning = true;
              shouldrender =true;
            } else {
              currentMousePosition.x = dx;
              currentMousePosition.y = dy;
            }
          }


        }

        function  onContainerMouseUp(){
          isUserInteracting =false;
          isUserPanning = false;

        }

        animate();

        function animate(){
          requestAnimationFrame(animate);
          if (isUserPanning && isUserInteracting) {
            updateMoveVelocityWhenUserPanning();
          }
          if(shouldrender){
            render();
            shouldrender = false;
          }
        }

        function render(){
          console.log(lon,lat)
          var phi = THREE.Math.degToRad(90 - lat);
          var theta = THREE.Math.degToRad(lon);

          camera.target.x = camera.position.x + 450 * Math.sin(phi) * Math.cos(theta);
          camera.target.z = camera.position.z +450 * Math.cos(phi);
          camera.target.y = camera.position.y + 450 * Math.sin(phi) * Math.sin(theta);
          renderer.render(scene,camera);
        }


        function updateMoveVelocityWhenUserPanning() {
          var scale = fov / 100;

          lon = mouseMoveDelta.x * scale * 0.1 + onPointerDownLon;
          lat = mouseMoveDelta.y * scale * 0.1 + onPointerDownLat;
          // if (lat < 0) {
          //   lat = Math.max(-LAT_RANGE_MIN, lat);
          // } else {
          //   lat = Math.min(LAT_RANGE_MAX, lat);
          // }
          previousMousePosition.x = currentMousePosition.x;
          previousMousePosition.y = currentMousePosition.y;

          currentMousePosition.x = mouseMoveDelta.x;
          currentMousePosition.y = mouseMoveDelta.y;

          mouseMoveVelocity.x = ( currentMousePosition.x - previousMousePosition.x );
          mouseMoveVelocity.y = ( currentMousePosition.y - previousMousePosition.y );
          shouldrender = true;
        }
//获取元素在世界坐标系中的位置
        function getWorldPosition(position,cube){
          var vector = position.applyMatrix4(cube.matrixWorld);
         // console.log(vector)
        }

        function onContainerMouseWheel(){
          if (event.wheelDeltaY) {

            camera.fov -= event.wheelDeltaY * 0.05;

            // Opera / Explorer 9

          } else if (event.wheelDelta) {

            camera.fov -= event.wheelDelta * 0.05;

            // Firefox

          } else if (event.detail) {

            camera.fov += event.detail * 1.0;

          }

          if (camera.fov < 10) camera.fov = 10;

          else if (camera.fov > 150) camera.fov = 150;

          camera.updateProjectionMatrix();

          render()

        }
      }
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
