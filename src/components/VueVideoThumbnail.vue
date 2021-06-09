<template>
  <div
    :class="[
        'snapshot-generator',
        (showPlayButton && displayedSnapshot) ? 'snapshot-generator__play-button' : ''
    ]"
  >
    <video
      v-if="toolsActive"
      ref="videoEl"
      class="snapshot-generator__hidden"
      muted
      :src="videoSrc"
      @loadeddata="onLoadedData"
      @loadedmetadata="onLoadedMetadata"
      @suspend="onSuspend"
      @seeked="onSeeked"
    />
    <canvas
      v-if="toolsActive"
      ref="canvas"
      class="snapshot-generator__hidden"
    />
    <!--    slot for custom snapshot handling-->
      <slot
      name="snapshot"
      :snapshot="displayedSnapshot"
      :renderThumbnail="renderThumbnail"
    >
        <img
          v-if="displayedSnapshot && renderThumbnail"
          :src="displayedSnapshot"
          alt="snapshot"
        >
    </slot>
  </div>
</template>

<script>

export default {
  name: 'VueVideoThumbnail',
  props: {
    videoSrc: {
      type: String,
      required: true,
    },

    renderThumbnail: {
      type: Boolean,
      default: true,
    },

    width: {
      type: Number,
      default: 250,
      validator: (value) => value > 0,
    },

    height: {
      type: Number,
      default: 250,
      validator: (value) => value > 0,
    },

    snapshotAtTime: {
      type: Number,
      default: 2,
      validator: (value) => value > 0,
    },

    snapshotScaleType: {
      type: String,
      default: 'contain',
      validator: (value) => ['contain', 'cover'].indexOf(value) !== -1,
    },

    backgroundFillColor: {
      type: String,
      default: '#000000',
    },

    snapshotAtDurationPercent: {
      type: Number,
      default: 50,
      validator: (value) => !(value > 100 || value < 0),
    },

    snapshotQuality: {
      type: Number,
      default: 0.8,
      validator: (value) => !(value > 1 || value < 0),
    },

    snapshotFormat: {
      type: String,
      default: 'image/jpeg',
    },

    cors: {
      type: Boolean,
      default: false,
    },

    percentagesArray: {
      type: Array,
      default: null,
    },

    // number of snapshots, that will be created, evenly distributing through the video duration.
    chunksQuantity: {
      type: Number,
      default: null,
      validator: (value) => value > 0,
    },

    // if set to true, the component destroys video and canvas elements to clear the DOM after creating first snapshot
    // or snapshotsArray, if its required, so the creation of new snapshots become impossible after that.
    once: {
      type: Boolean,
      default: false,
    },

    showPlayButton: {
      type: Boolean,
      default: false,
    },
  },

  data: () => ({
    displayedSnapshot: null,
    metadataLoaded: false,
    dataLoaded: false,
    suspended: false,
    seeked: false,
    duration: null,
    snapshots: [],
    awaitForMultiSnapshots: false,
    awaitForSingleSnapshot: true,
    timestamps: [],
    displayedSnapshotTime: null,
    toolsActive: true,
  }),

  computed: {
    video() {
      return (this.$refs.videoEl) ? this.$refs.videoEl : null;
    },

    canvas() {
      return (this.$refs.canvas) ? this.$refs.canvas : null;
    },

    multiSnapshotsIsNeeded() {
      return !this.snapshots.length && (!!this.percentagesArray || !!this.chunksQuantity);
    },
  },

  watch: {
    snapshotAtDurationPercent(newVal) {
      this.getSnapshotByPercentage(newVal);
    },

    width() {
      this.refreshPropUpdated();
    },

    height() {
      this.refreshPropUpdated();
    },

    snapshotQuality() {
      this.refreshPropUpdated();
    },

    snapshotFormat() {
      this.refreshPropUpdated();
    },

    videoSrc(newVal, oldVal) {
      if (newVal && newVal !== oldVal) this.refresh();
    },

  },

  mounted() {
    if (!this.cors) this.$refs.videoEl.setAttribute('crossOrigin', 'Anonymous');
  },

  beforeDestroy() {
    this.clear();
  },

  methods: {
    // Video element condition's handlers.

    onLoadedData() {
      this.dataLoaded = true;
      this.videoConditionUpdateHandler();
    },

    onLoadedMetadata() {
      this.metadataLoaded = true;
      this.videoConditionUpdateHandler();
    },

    onSuspend() {
      this.suspended = true;
      this.videoConditionUpdateHandler();
    },

    onSeeked() {
      this.seeked = true;
      if (!this.displayedSnapshot || this.awaitForSingleSnapshot) this.createSingleSnapshot();
      else if (this.awaitForMultiSnapshots) this.multiSnapshotsSeekHandler();
    },

    videoConditionUpdateHandler() {
      if (!this.video || !this.canvas) return;
      if (this.metadataLoaded && this.dataLoaded && this.suspended) {
        this.duration = this.video.duration;
        if (this.awaitForSingleSnapshot) {
          let snapshotTime = this.snapshotAtTime;
          if (this.snapshotAtDurationPercent) snapshotTime = this.percentageToTime(this.snapshotAtDurationPercent);
          if (!this.video.currentTime || this.video.currentTime < snapshotTime) {
            this.setVideoTime(snapshotTime);
          }
        }
      }
    },

    // Props condition handlers

    refreshPropUpdated() {
      this.awaitForSingleSnapshot = true;
      this.videoConditionUpdateHandler();
    },

    handleArrayProps() {
      if (!this.multiSnapshotsIsNeeded) return;
      if (this.percentagesArray) {
        this.timestamps = this.getTimestampsByPercentagesArray(this.percentagesArray);
        this.createMultiSnapshots();
      } else if (this.chunksQuantity) {
        this.timestamps = this.getTimestampsByChunksQuantity(this.chunksQuantity);
        this.createMultiSnapshots();
      }
    },

    // Snapshot creation handlers

    createSingleSnapshot() {
      this.displayedSnapshot = this.createSnapshot();
      this.displayedSnapshotTime = this.video.currentTime;
      this.$emit('snapshotCreated', this.displayedSnapshot);
      this.awaitForSingleSnapshot = false;
      if (this.multiSnapshotsIsNeeded) this.handleArrayProps();
      else if (this.once) this.clear();
    },

    createMultiSnapshots() {
      this.awaitForMultiSnapshots = (this.timestamps.length !== this.snapshots.length);
      if (this.awaitForMultiSnapshots) {
        this.setVideoTime(this.timestamps[this.snapshots.length]);
      } else {
        this.$emit('snapshotsArrayCreated', this.snapshots);
        this.setVideoTime(this.displayedSnapshotTime);
        if (this.once) this.clear();
      }
    },

    multiSnapshotsSeekHandler() {
      const snapshot = this.createSnapshot();
      this.snapshots.push(snapshot);
      this.createMultiSnapshots();
    },

    getSnapshotByTime(time) {
      if (!this.video || !this.canvas) return;
      this.awaitForSingleSnapshot = true;
      this.setVideoTime(time);
    },

    getSnapshotByPercentage(percentage) {
      this.getSnapshotByTime(this.percentageToTime(percentage));
    },

    // Snapshot creation function

    createSnapshot() {
      this.canvas.height = this.height;
      this.canvas.width = this.width;
      const context = this.canvas.getContext('2d');

      // Coloring canvas background

      context.rect(0, 0, this.width, this.height);
      context.fillStyle = this.backgroundFillColor;
      context.fill();

      // Image scaling and placing it to canvas

      if (this.snapshotScaleType === 'contain') {
        const scale = Math.min(this.canvas.width / this.video.videoWidth, this.canvas.height / this.video.videoHeight);
        const x = (this.width / 2) - (this.video.videoWidth / 2) * scale;
        const y = (this.height / 2) - (this.video.videoHeight / 2) * scale;
        context.drawImage(this.video, x, y, this.video.videoWidth * scale, this.video.videoHeight * scale);
      } else if (this.snapshotScaleType === 'cover') {
        const scale = this.height / this.width;
        let width = this.video.videoWidth;
        let height = this.video.videoHeight;
        const imgRatio = height / width;

        if (imgRatio > scale) {
          height = width * scale;
        } else {
          width = height / scale;
        }

        context.drawImage(this.video,
          (this.video.videoWidth - width) * 0.5, (this.video.videoHeight - height) * 0.5, width, height,
          0, 0, this.width, this.height);
      }

      // Converting canvas to data url

      const snapshot = this.canvas.toDataURL(this.snapshotFormat, this.snapshotQuality);
      this.$emit('sizeChanged', snapshot.length);
      return snapshot;
    },

    // Utils

    setVideoTime(time) {
      if (this.video && time) this.video.currentTime = time;
    },

    clear() {
      this.toolsActive = false;
    },

    refresh() {
      this.toolsActive = true;
      this.metadataLoaded = false;
      this.dataLoaded = false;
      this.suspended = false;
      this.seeked = false;
      this.duration = null;
      this.snapshots = [];
      this.awaitForMultiSnapshots = false;
      this.awaitForSingleSnapshot = true;
      this.timestamps = [];
      this.displayedSnapshotTime = null;
      this.$nextTick(() => {
        this.video.currentTime = 0;
        this.video.src = '';
        this.video.src = this.videoSrc;
        const context = this.canvas.getContext('2d');
        context.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.$nextTick(() => {
          this.videoConditionUpdateHandler();
        });
      });
    },

    percentageToTime(percentage) {
      return this.duration * (percentage / 100);
    },

    getTimestampsByChunksQuantity(chunksQuantity) {
      const chunkLength = this.duration / chunksQuantity;
      const timestamps = [];
      for (let i = 0; i < chunksQuantity; i++) {
        timestamps.push(((i + 1) * chunkLength).toFixed(2));
      }
      return timestamps;
    },

    getTimestampsByPercentagesArray(percentagesArray) {
      return percentagesArray.map((percentage) => this.percentageToTime(percentage));
    },

  },
};
</script>

<style>
.snapshot-generator {
  display: block;
  position: relative;
}
.snapshot-generator__hidden {
  height: 1px;
  width: 1px;
  overflow: hidden;
  position: absolute;
  bottom: 0;
  right: 0;
  z-index: -1;
}
.snapshot-generator__play-button {
  width: min-content;
  height: min-content;
  position: relative;
  cursor: pointer;
}
.snapshot-generator__play-button:after {
  content: '';
  position: absolute;
  top: calc(50% - 50px / 2);
  left: calc(50% - 50px / 2);
  height: 50px;
  width: 50px;
  border-radius: 50%;
  background-color: #4F4F4F;
  opacity: 0.5;
}
.snapshot-generator__play-button:before {
  content: '';
  position: absolute;
  top: calc(50% - 50px  / 4);
  left: calc(50% - 50px / 8);
  height: 0;
  width: calc(50px / 2);
  border: calc(50px / 4) solid transparent;
  border-left: calc(50px / 2.75) solid white;
  z-index: 2;
  opacity: 0.75;
}
.snapshot-generator__play-button:hover:before {
  opacity: 1;
}
.snapshot-generator__play-button:active:after {
  transform: scale(0.95);
  transition: 0.1s;
}
.snapshot-generator__play-button:active:before {
  transform: scale(0.95);
  transition: 0.1s;
}
</style>

