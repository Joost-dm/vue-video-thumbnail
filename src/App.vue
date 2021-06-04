<template>
  <div class="app-grid">
    <div class="app-grid__controllers">
      <label for="src">Video src:</label>
      <input disabled id="src" type="text" v-model="videoSrc" placeholder="video src">
      <label for="percentage">Snapshot at video duration percent:</label>
      <input id="percentage" type="number" min="0" max="100" v-model="snapshotPercentage" placeholder="percentage">
      <label for="height">Height:</label>
      <input id="height"  type="number" min="0" v-model="snapshotHeight" placeholder="height">
      <label for="width">Height:</label>
      <input id="width" type="number" min="0" v-model="snapshotWidth" placeholder="width">
      <label for="quality">Quality:</label>
      <input id="quality" type="number" step="0.1" min="0" max="1" v-model="snapshotQuality" placeholder="quality">
      <label for="format">Format:</label>
      <input id="format" v-model="snapshotFormat" placeholder="format">
    </div>
    <div class="app-grid__preview">
      <VueVideoThumbnail
      :video-src="videoSrc"
      :snapshot-at-duration-percent="parseInt(snapshotPercentage, 10)"
      snapshot-scale-type="contain"
      :snapshot-format="snapshotFormat"
      background-fill-color="black"
      :snapshot-quality="parseFloat(snapshotQuality, 10)"
      :width="parseInt(snapshotWidth, 10)"
      :height="parseInt(snapshotHeight, 10)"
      :chunks-quantity="10"
      @sizeChanged="(value) => snapshotSize = value"
      @snapshotCreated="test"
      @snapshotsArrayCreated="test"
    >
      <template #snapshot="{snapshot}">
        <img
          v-if="snapshot"
          :src="snapshot"
          alt="snapshot"
        >
      </template>
    </VueVideoThumbnail>
    </div>

  </div>
</template>

<script>
import VueVideoThumbnail from '@/components/VueVideoThumbnail';

export default {
  name: 'TestPage',
  components: { VueVideoThumbnail },
  data: () => ({
    videoSrc: 'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4',
    snapShotSrc: null,
    snapshotHeight: 400,
    snapshotWidth: 400,
    snapshotPercentage: 50,
    snapshotQuality: 0.8,
    snapshotFormat: 'image/jpeg',
    snapshotSize: 0,
  }),

  methods: {
    test(image) {
      console.log(image);
    },
  },
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.app-grid {
  display: grid;
  gap: 10px;
  grid-template-columns: 500px 1fr;

}
.app-grid__controllers {
  grid-column: 1/2;
  display: flex;
  flex-direction: column;
  color: white;
  background-color: #2c3e50;
}
.app-grid__controllers input {
  height: 30px;
  font-size: 20px;
}
.app-grid__preview {
  grid-column: 2/3;
}

</style>
