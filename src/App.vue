<template>
  <div ref="app" id="app">
    <div class="timecode">{{timecode}}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{{frame}}</div>
    <VGPlayer class="hires"
              ref="hires"
              :url="`${url1}`"
              :subtitles="`${subtitles}`"
              v-bind:startFrame="startFrame"
              v-bind:initialFrame="initialFrame"
              v-bind:endFrame="endFrame"
              v-bind:hotkeys="true"
              v-on:frameChange="onFrameChange"
              v-on:videoSize="onVideoSize"
              v-on:play="onPlay"/>
    <div class="divider"
         v-bind:style="{left: slider + `%`}"></div>
    <div class="lowres_container"
         v-bind:style="{webkitClipPath: `inset(0 0 0 ` + slider + `%)`, clipPath: `inset(0 0 0 ` + slider + `%)`}">
      <VGPlayer ref="lores"
                class="lowres"
                :url="`${url2}`"
                v-bind:startFrame="startFrame"
                v-bind:endFrame="endFrame"
                v-bind:style="{width: videoWidth + `px`, height: videoHeight + `px`}"
                v-on:frameChange="onLowresFrameChange"
                v-bind:controls="false"/>
    </div>
  </div>
</template>

<script>
import VGPlayer from "@/components/VGPlayer";
const getParams = new URLSearchParams(location.search);

export default {
  name: 'app',
  components: {
    VGPlayer,
  },
  data: () => {
    return {
      publicPath: process.env.BASE_URL,
      url1: getParams.get("url1") || "video1.mp4",
      url2: getParams.get("url2") || "video2.mp4",
      subtitles: getParams.get("srt") || "./subtitles.srt",
      slider: !isNaN(parseFloat(getParams.get("slider"))) ? parseFloat(getParams.get("slider")) : 50,
      frame: 0,
      frameLowres: 0,
      timecode: "",
      videoWidth: 0,
      videoHeight: 0,
      startFrame: 0,
      endFrame: -1,
      initialFrame: parseInt(getParams.get("fn")) || 0
    }
  },
  methods: {
    onPlay: function(isPlaying) {
      if (isPlaying) {
        this.$refs.lores.play();
      } else {
        this.$refs.lores.pause();
      }
      this.onFrameChange(this.$refs.hires.getFrame());
    },
    onLowresFrameChange: function(frame) {
      this.frameLowres = frame;
    },
    onFrameChange: function(frame) {
      this.frame = frame;
      this.timecode = this.$refs.hires.getTapeTimecode();
      if (this.$refs.hires.isPlaying()) {
        return;
      }
      this.$refs.lores.seekFrame(frame);
    },
    updateSlider: function(e) {
      this.slider = (e.pageX - this.$refs.app.getBoundingClientRect().left) / this.$refs.app.clientWidth * 100;
    },
    onVideoSize: function(width, height) {
      this.videoWidth = width;
      this.videoHeight = height;
    }
  },
  mounted: function() {
    setInterval(() => {
      const query = new URLSearchParams(location.search);
      const oldQuery = query.toString();
      query.set("fn", this.frame);
      query.set("slider", "" + Math.round(this.slider * 10000) / 10000);
      if (query.toString() !== oldQuery) {
        history.pushState(null, null, "?" + query.toString());
      }
    }, 400);
    this.$refs.hires.video.addEventListener("click", (e) => {
      e.preventDefault();
    });
    const onSlide = function(e) {
      if (this.sliding) {
        this.updateSlider(e);
      }
      e.preventDefault();
    }.bind(this);
    this.$refs.app.addEventListener("mousemove", onSlide);
    this.$refs.app.addEventListener("touchmove", onSlide);
    const stopSliding = function (e) {
      if (this.sliding) {
        this.updateSlider(e);
      }
      this.sliding = false;
    }.bind(this);
    window.addEventListener("mouseup", stopSliding);
    window.addEventListener("touchend", stopSliding);
    const startSliding = (e) => {
      this.sliding = true;
      this.updateSlider(e);
    };
    this.$refs.app.addEventListener("mousedown", startSliding);
    this.$refs.app.addEventListener("touchstart", startSliding);
    window.addEventListener("keydown", function (e) {
      if(e.key.toLowerCase().indexOf("arrow") >= 0) {
        e.preventDefault();
      }
    }.bind(this));
  }
}
</script>

<style>
  body, html {
    padding: 0;
    margin: 0;
    background-color: black;
  }

  .hires video {
    width: auto !important;
    height: auto !important;
  }

  #app {
    position: absolute;
  }

  .divider {
    pointer-events: none;
    position: absolute;
    width: 1px;
    border-left: 1px solid black;
    height: 100%;
    z-index: 1;
    top: 0;
  }

  .timecode {
    transform: translateX(-50%);
    position: absolute;
    left: 50%;
    top: 5%;
    padding: 5px;
    background: black;
    color: white;
    z-index: 2;
    font-family: monospace;
    vertical-align: middle;
    min-width: 50px;
  }

  .timecode span {
    display: block;
    font-size: 8px;
    vertical-align: middle;
  }

  .hires video {
    object-fit: none;
  }

  .lowres_container {
    position: absolute;
    width: 100%;
    height: 100%;
    pointer-events: none;
    top: 0;
    left: 0;
  }
</style>
