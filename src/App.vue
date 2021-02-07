<template>
  <UploadFile/>
</template>

<script>
import UploadFile from "@/components/Sources/Kaggle/UploadFile";

export default {
  name: 'App',
  components: {
    UploadFile
  }
}
</script>

<style>
#app {
  margin: 0;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
