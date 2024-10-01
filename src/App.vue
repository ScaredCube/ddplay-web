<template>
  <div>
    <ddPlay :videoUrl="videoUrl" />
  </div>
</template>

<script>
import ddPlay from './components/ddPlay.vue';

export default {
  components: {
    ddPlay
  },
  data() {
    return {
      videoUrl: null
    };
  },
  created() {
    // 从 iframe URL 中获取参数
    const params = new URLSearchParams(window.location.search);
    const videoUrl = params.get('videoUrl');
    
    // 检查是否有 videoUrl 参数
    if (videoUrl) {
      this.videoUrl = videoUrl;
    } else {
      console.error('未提供 videoUrl 参数');
    }
  }
};
</script>

<style scoped>
</style>
