<template>
  <div>
    <!--
    <input v-model="videoUrl" placeholder="视频URL" required=""/>
    <button @click="fetchVideoInfo">确认</button>-->

    <!--
    <div style="display:flex">
      <div class="form">
        <input v-model="videoUrl" class="input" placeholder="视频URL" required="" type="text">
        <span class="input-border"></span>
      </div>
      <button @click="fetchVideoInfo">确认</button>
    </div>

    <div v-if="videoInfo">
      <p>Name: {{ videoInfo.fileName }}</p>
      <p>Size: {{ videoInfo.fileSize }} Bytes</p>
      <p>MD5: {{ videoInfo.md5Hash }}</p>
      <p v-if="episodeId">EpisodeID: {{ episodeId }}</p>
      <p v-if="animeTitle">AnimeTitle: {{ animeTitle }}</p>
      <p v-if="episodeTitle">EpisodeTitle: {{ episodeTitle }}</p>
    </div>
    <p v-if="error" style="color: red;">错误: {{ error }}</p>
    -->
    <ArtPlayer v-if="videoUrl && episodeId" :videoUrl="videoUrl" :episodeId="episodeId" />
    <p v-if="error" style="color: red;">{{ error }}</p>
    <p v-if="!(videoUrl && episodeId)">Loading...</p>
  </div>
</template>

<script>
import CryptoJS from 'crypto-js';
import ArtPlayer from './ArtPlayer.vue';

export default {
  name: "ddPlay",
  props: {
    videoUrl: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      episodeId: null,
      animeTitle: null,
      episodeTitle: null,
      error: null
    };
  },
  components: {
    ArtPlayer
  },
  watch: {
    videoUrl: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.fetchVideoInfo();
        }
      }
    }
  },
  methods: {
    async fetchVideoInfo() {
      this.error = null;
      this.episodeId = null;
      this.animeTitle = null;
      this.episodeTitle = null;
      try {
        // 文件名
        const fileName = this.getFileName(this.videoUrl);

        // 总长度
        const response = await fetch(this.videoUrl, { method: 'HEAD' });
        if (!response.ok) {
          throw new Error(`无法获取文件信息: ${response.statusText}`);
        }

        const contentLength = response.headers.get('content-length');
        const fileSize = parseInt(contentLength, 10);

        // 前16MB
        const rangeResponse = await fetch(this.videoUrl, {
          method: 'GET',
          headers: {
            'Range': 'bytes=0-16777215'
          }
        });

        if (!rangeResponse.ok) {
          throw new Error(`无法获取文件范围: ${rangeResponse.statusText}`);
        }

        const arrayBuffer = await rangeResponse.arrayBuffer();

        // MD5
        const wordArray = CryptoJS.lib.WordArray.create(arrayBuffer);
        const md5Hash = CryptoJS.MD5(wordArray).toString(CryptoJS.enc.Hex);

        this.videoInfo = {
          fileName,
          fileSize,
          md5Hash
        };

        await this.fetchEpisodeId(fileName, md5Hash, fileSize);

      } catch (err) {
        this.error = err.message;
      }
    },
    getFileName(url) {
      const path = new URL(url).pathname;
      const fullName = path.substring(path.lastIndexOf('/') + 1);
      return fullName.substring(0, fullName.lastIndexOf('.')) || fullName;
    },
    async fetchEpisodeId(fileName, fileHash, fileSize) {
      try {
        const response = await fetch('https://dandanplay.scc.moe/api/v2/match', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            'fileName': fileName,
            'fileHash': fileHash,
            'fileSize': fileSize,
            'matchMode': 'hashAndFileName'
          })
        });

        const data = await response.json();

        if (data.success && data.isMatched && data.matches.length > 0) {
          this.episodeId = data.matches[0].episodeId;
          this.animeTitle = data.matches[0].animeTitle;
          this.episodeTitle = data.matches[0].episodeTitle;
        } else {
          this.error = "未找到匹配的EpisodeID";
        }
      } catch (err) {
        this.error = `API Error: ${err.message}`;
      }
    }
  }
};
</script>

<style scoped>
/*
.form {
  --width-of-input: 90%;
  --border-height: 1px;
  --border-before-color: rgba(221, 221, 221, 0.39);
  --border-after-color: #5891ff;
  --input-hovered-color: #4985e01f;
  position: relative;
  width: var(--width-of-input);
}

.input {
  color: #fff;
  font-size: 0.9rem;
  background-color: transparent;
  width: 100%;
  box-sizing: border-box;
  padding-inline: 0.5em;
  padding-block: 0.7em;
  border: none;
  border-bottom: var(--border-height) solid var(--border-before-color);
}

.input-border {
  position: absolute;
  background: var(--border-after-color);
  width: 0%;
  height: 2px;
  bottom: 0;
  left: 0;
  transition: 0.3s;
}

input:hover {
  background: var(--input-hovered-color);
}

input:focus {
  outline: none;
}

input:focus~.input-border {
  width: 100%;
}

button {
  display: inline-block;
  width: 10%;
  margin-left: 10px;
  border-radius: 5px;
  background-color: var(--theme-color);
  border: none;
  color: white;

}

button:hover {
  background-color: var(--theme-color-mask);
}
*/
</style>
