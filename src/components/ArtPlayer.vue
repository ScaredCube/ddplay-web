<template>
  <div ref="artplayerContainer" :style="{ width: '100%', height: '500px' }"></div>
</template>

<script>
import Artplayer from 'artplayer';
import ArtplayerPluginDanmuku from 'artplayer-plugin-danmuku';


export default {
  props: {
    videoUrl: {
      type: String,
      required: true,
    },
    danmukuUrlTemplate: {
      type: String,
      default: 'https://danmuku.scc.moe/',
    },
    episodeId: {
      type: String,
      required: true,
    },
  },
  mounted() {
    const danmukuUrl = `${this.danmukuUrlTemplate}${this.episodeId}`;

    this.art = new Artplayer({
      container: this.$refs.artplayerContainer,
      url: this.videoUrl,
      autoplay: false,
      autoSize: true,
      fullscreen: true,
      fullscreenWeb: true,
      autoOrientation: true,
      plugins: [
        ArtplayerPluginDanmuku({
          danmuku: danmukuUrl,
          // 以下为非必填
          speed: 7.5, // 弹幕持续时间，范围在[1 ~ 10]
          opacity: 0.5, // 弹幕透明度，范围在[0 ~ 1]
          antiOverlap: true, // 弹幕是否防重叠
          synchronousPlayback: true, // 是否同步播放速度
          heatmap: false, // 是否开启热力图
          points: [], // 热力图数据
          visible: true, // 弹幕层是否可见
          emitter: false, // 是否开启弹幕发射器
        }),
      ],
    });
  },
  beforeDestroy() {
    if (this.art) {
      this.art.destroy();
    }
  },
};
</script>

<style scoped></style>
