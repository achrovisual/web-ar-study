<template>
        <a-scene embedded arjs vr-mode-ui="enabled: false">
        <a-marker preset="hiro">
            <!-- we use cors proxy to avoid cross-origin problems ATTENTION! you need to set up your server -->
            <a-entity
            position="0 0 0"
            scale="0.05 0.05 0.05"
            gltf-model="scene.gltf"
            ></a-entity>
        </a-marker>
        <a-entity camera></a-entity>
        </a-scene>
</template>

<script>

export default {
  name: "App",
}
</script>

<style>

</style>
