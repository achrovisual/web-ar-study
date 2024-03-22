<template>
  <video id="video" playsinline webkit-playsinline muted loop autoplay width="320" height="240" src="video/taco.mp4" style="display: none"></video>
</template>

<script>
import * as THREE from 'three';
import { ArToolkitSource, ArToolkitContext, ArMarkerControls } from '@ar-js-org/ar.js/three.js/build/ar-threex.js';

export default {
  /* eslint-disable */ 
  name: 'App',
  mounted() {
    var scene, camera, renderer, clock, deltaTime, totalTime;
    
    var arToolkitSource, arToolkitContext;
    
    var markerRoot1;
    
    var mesh1;
    
    initialize();
    animate();
    
    function initialize() {
      scene = new THREE.Scene();
      
      let ambientLight = new THREE.AmbientLight( 0xcccccc, 0.5 );
      scene.add( ambientLight );
      
      camera = new THREE.Camera();
      scene.add(camera);
      
      renderer = new THREE.WebGLRenderer({
        antialias : true,
        alpha: true
      });
      renderer.setClearColor(new THREE.Color('lightgrey'), 0)
      renderer.setSize( 640, 480 );
      renderer.domElement.style.position = 'absolute'
      renderer.domElement.style.top = '0px'
      renderer.domElement.style.left = '0px'
      document.body.appendChild( renderer.domElement );
      
      clock = new THREE.Clock();
      deltaTime = 0;
      totalTime = 0;
      
      ////////////////////////////////////////////////////////////
      // setup arToolkitSource
      ////////////////////////////////////////////////////////////
      
      arToolkitSource = new ArToolkitSource({
        sourceType : 'webcam',
      });
      
      function onResize() {
        arToolkitSource.onResizeElement()
        arToolkitSource.copyElementSizeTo(renderer.domElement)
        if(arToolkitContext.arController !== null){
          arToolkitSource.copyElementSizeTo(arToolkitContext.arController.canvas)
        }
      }
      
      arToolkitSource.init(function onReady(){
        onResize()
      });
      
      // handle resize event
      window.addEventListener('resize', function(){
        onResize()
      });
      
      ////////////////////////////////////////////////////////////
      // setup arToolkitContext
      ////////////////////////////////////////////////////////////	
      
      // create atToolkitContext
      arToolkitContext = new ArToolkitContext({
        cameraParametersUrl: 'data/camera_para.dat',
        detectionMode: 'mono',
        // canvasWidth: document.documentElement.clientWidth,
        // canvasHeight: document.documentElement.clientHeight,
      });
      
      // copy projection matrix to camera when initialization complete
      arToolkitContext.init( function onCompleted(){
        camera.projectionMatrix.copy( arToolkitContext.getProjectionMatrix() );
        
        // arToolkitContext.arController.orientation = getSourceOrientation(); 
        // arToolkitContext.arController.options.orientation = getSourceOrientation(); 
        arToolkitContext.arController.orientation = "portrait"
        arToolkitContext.arController.options.orientation = "portrait"
        arToolkitContext.arController.setPattRatio(0.9);
        
        console.log('arToolkitContext', arToolkitContext); 
        window.arToolkitContext = arToolkitContext; 
      });
      
      function getSourceOrientation() { 
        if (!arToolkitSource) { 
          return null; 
        } 
        
        console.log( 
        'actual source dimensions', 
        arToolkitSource.domElement.videoWidth, 
        arToolkitSource.domElement.videoHeight 
        ); 
        
        if (arToolkitSource.domElement.videoWidth > arToolkitSource.domElement.videoHeight) { 
          console.log('source orientation', 'landscape'); 
          return 'landscape'; 
        } else { 
          console.log('source orientation', 'portrait'); 
          return 'portrait'; 
        } 
      }
      
      // arToolkitContext.arController.orientation = "portrait"
      
      ////////////////////////////////////////////////////////////
      // setup markerRoots
      ////////////////////////////////////////////////////////////
      
      // build markerControls
      markerRoot1 = new THREE.Group();
      scene.add(markerRoot1);
      let markerControls1 = new ArMarkerControls(arToolkitContext, markerRoot1, {
        type: 'pattern', patternUrl: 'data/taco.patt',
      })
      
      // Auto play video on load
      let video = document.getElementById("video");
      video.onloadeddata = function () {
        video.play();
      };
      
      //Create your video texture:
      let texture = new THREE.VideoTexture(video);
      let material1 =  new THREE.MeshBasicMaterial( {map: texture, side: THREE.FrontSide, toneMapped: false} );
      //Create screen
      let geometry1 = new THREE.PlaneGeometry(0.89,1,4,4);
      
      mesh1 = new THREE.Mesh( geometry1, material1 );
      mesh1.rotation.x = -Math.PI/2;
      mesh1.scale.normalize().multiplyScalar(1.55);
      
      markerRoot1.add( mesh1 );
      
      const axesHelper = new THREE.AxesHelper( 5 );
      markerRoot1.add( axesHelper );
      
      console.log(arToolkitContext)
    }
    
    
    function update() {
      // update artoolkit on every frame
      if ( arToolkitSource.ready !== false )
      arToolkitContext.update( arToolkitSource.domElement );
    }
    
    
    function render() {
      renderer.render( scene, camera );
    }
    
    
    function animate() {
      requestAnimationFrame(animate);
      deltaTime = clock.getDelta();
      totalTime += deltaTime;
      update();
      render();
    }
    
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
