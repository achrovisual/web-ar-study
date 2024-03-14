<template>
  <video id="video" playsinline webkit-playsinline muted loop autoplay width="320" height="240" src="video/sintel.mp4" style="display: none"></video>
</template>

<script>
import * as THREE from 'three';
import { ArToolkitProfile, ArToolkitSource, ArToolkitContext, ArMarkerControls} from '@ar-js-org/ar.js/three.js/build/ar-threex.js';

export default {
  /* eslint-disable */ 
  name: 'App',
  mounted() {
    console.log(ArToolkitContext.baseURL)
    ArToolkitContext.baseURL = './'
    console.log("ARScene mounted");
    
    // init renderer
    var renderer	= new THREE.WebGLRenderer({
      antialias	: true,
      alpha: true
    });
    
    renderer.setClearColor(new THREE.Color('lightgrey'), 0)
    // renderer.setPixelRatio( 2 );
    renderer.setSize( window.innerWidth, window.innerHeight );
    renderer.domElement.style.position = 'absolute'
    renderer.domElement.style.top = '0px'
    renderer.domElement.style.left = '0px'
    document.body.appendChild( renderer.domElement ); // We should be able to specify an html element to append AR.js related elements to.
    
    // array of functions for the rendering loop
    var onRenderFcts= [];
    
    // init scene and camera
    var scene	= new THREE.Scene();
    
    let ambientLight = new THREE.AmbientLight( 0xcccccc, 0.5 );
    scene.add( ambientLight );
    
    //////////////////////////////////////////////////////////////////////////////////
    //		Initialize a basic camera
    //////////////////////////////////////////////////////////////////////////////////
    
    // Create a camera
    var camera = new THREE.Camera();
    scene.add(camera);
    const artoolkitProfile = new ArToolkitProfile();
    artoolkitProfile.sourceWebcam(); // Is there good reason for having a function to set the sourceWebcam but not the displayWidth/Height etc?
    
    // add existing parameters, this is not well documented
    let additionalParameters = {
      // Device id of the camera to use (optional)
      deviceId: null,
      // resolution of at which we initialize in the source image
      sourceWidth: 640,
      sourceHeight: 480,
      // resolution displayed for the source
      displayWidth: 640,
      displayHeight: 480,
    }
    
    Object.assign(artoolkitProfile.sourceParameters, additionalParameters);
    console.log(artoolkitProfile.sourceParameters); // now includes the additionalParameters
    
    const arToolkitSource = new ArToolkitSource(artoolkitProfile.sourceParameters);
    
    arToolkitSource.init(function onReady(){
      onResize();
    })
    
    // handle resize
    window.addEventListener('resize', function(){
      onResize(); 
    })
    
    // resize is not called for the canvas on init. The canvas with the cube seems to be resized correctly at start.
    // Is that maybe a vue-specific problem?
    function onResize(){
      arToolkitSource.onResizeElement()
      arToolkitSource.copyElementSizeTo(renderer.domElement)
      if(arToolkitContext.arController !== null){
        arToolkitSource.copyElementSizeTo(arToolkitContext.arController.canvas)
      }
    }
    
    
    ////////////////////////////////////////////////////////////////////////////////
    //          initialize arToolkitContext
    ////////////////////////////////////////////////////////////////////////////////
    
    // create atToolkitContext
    var arToolkitContext = new ArToolkitContext({
      debug: false,
      cameraParametersUrl: ArToolkitContext.baseURL + 'data/camera_para.dat',
      detectionMode: 'mono',
      canvasWidth: 640,
      canvasHeight: 490,
      imageSmoothingEnabled : true, // There is still a warning about mozImageSmoothingEnabled when using Firefox
    })
    
    // initialize it
    arToolkitContext.init(function onCompleted(){
      // copy projection matrix to camera
      camera.projectionMatrix.copy( arToolkitContext.getProjectionMatrix() );
    })
    
    // update artoolkit on every frame
    onRenderFcts.push(function(){
      if( arToolkitSource.ready === false )	return
      
      arToolkitContext.update( arToolkitSource.domElement )
    })
    
    ////////////////////////////////////////////////////////////////////////////////
    //          Create a ArMarkerControls
    ////////////////////////////////////////////////////////////////////////////////
    
    var markerGroup = new THREE.Group()
    scene.add(markerGroup)
    
    var markerControls = new ArMarkerControls(arToolkitContext, markerGroup, {
      type: 'pattern',
      patternUrl: ArToolkitContext.baseURL + 'data/patt.hiro',
      smooth: true,
      smoothCount: 5,
      smoothTolerance: 0.01,
      smoothThreshold: 2
    })
    
    var markerScene = new THREE.Scene()
    markerGroup.add(markerScene)
    
    // Auto play video on load
    const video = document.getElementById("video");
    video.onloadeddata = function () {
      video.play();
    };
    
    //Create your video texture:
    const texture = new THREE.VideoTexture(video);
    const material1 =  new THREE.MeshBasicMaterial( {map: texture, side: THREE.FrontSide, toneMapped: false} );
    //Create screen
    const geometry1 = new THREE.PlaneGeometry(2,2,4,4);
    const mesh1 = new THREE.Mesh(geometry1, material1);
    markerScene.add(mesh1);
    
    //////////////////////////////////////////////////////////////////////////////////
    //		render the whole thing on the page
    //////////////////////////////////////////////////////////////////////////////////
    onRenderFcts.push(function(){
      renderer.render(scene, camera);
    })
    
    // run the rendering loop
    var lastTimeMsec = null;
    requestAnimationFrame(function animate(nowMsec){
      // keep looping
      requestAnimationFrame(animate);
      // measure time
      lastTimeMsec	= lastTimeMsec || nowMsec-1000/60
      var deltaMsec	= Math.min(200, nowMsec - lastTimeMsec)
      lastTimeMsec	= nowMsec
      // call each update function
      onRenderFcts.forEach(function(onRenderFct){
        onRenderFct(deltaMsec/1000, nowMsec/1000)
      })
    })
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
