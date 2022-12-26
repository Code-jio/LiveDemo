<template>
  <div class="video-box">
    <video
      controls="controls"
      class="demo-video"
      ref="player"
      muted
      @dblclick="fullScreen"
    ></video>
  </div>
</template>

<script>
import flvjs from "flv.js";
export default {
  components: {},
  data() {
    return {
      player: null,
      loading: true,
      rtspURl:
        "rtsp://172.16.100.110:554/ user=admin_password=tlJwpbo6_channel=1_stream=0&amp;onvif=0.sdp?real_st",
    };
  },
  mounted() {
    this.playVideo();
  },
  watch: {
    rtsp: {
      handler: function (val) {
        if (val) {
          if (this.player) {
            this.player.unload();
            this.player.destroy();
            this.player = null;
            this.loading = true;
          }
          this.$nextTick(() => {
            this.playVideo();
          });
        }
      },
      immediate: true,
    },
  },
  methods: {
    fullScreen() {
      if (this.$refs.player.requestFullScreen) {
        this.$refs.player.requestFullScreen();
      } else if (this.$refs.player.mozRequestFullScreen) {
        this.$refs.player.mozRequestFullScreen();
      } else if (this.$refs.player.webkitRequestFullScreen) {
        this.$refs.player.webkitRequestFullScreen();
      }
    },
    playVideo() {
      const time1 = new Date().getTime();
      if (flvjs.isSupported()) {
        let video = this.$refs.player;
        if (video) {
          this.player = flvjs.createPlayer(
            {
              type: "flv",
              isLive: true,
              url: `ws://172.16.100.90:8888/rtsp/1?url=rtsp://172.16.100.110:554/user=admin_password=tlJwpbo6_channel=1_stream=0&amp;onvif=0.sdp?real_st`,
              // url: `http://www.xmeye.net:8899/rtsp/1?url=rtsp://172.16.100.110:554/user=admin_password=tlJwpbo6_channel=1_stream=0&amp;onvif=0.sdp?real_st`,
            },
            {
              deferLoadAfterSourceOpen: false,
            }
          );

          this.player.attachMediaElement(video);
          console.log(this.player);

          try {
            this.player.load();

            var controller = this.player._transmuxer._controller;
            var wsLoader = controller._ioctl._loader;
            var oldWsOnCompleteFunc = wsLoader._onComplete;

            wsLoader._onComplete = function () {
              if (!controller._remuxer) {
                controller._remuxer = {
                  flushStashedSamples: function () {
                    this.loadingVisiable = false;
                    console.log("flushStashedSamples");
                  },
                };
              }
              oldWsOnCompleteFunc();
            };

            this.player.play().then(() => {
              console.log(new Date().getTime() - time1);
              this.loading = false;
            });
          } catch (error) {
            console.log(error);
          }
        }
      }
    },
  },
  beforeDestroy() {
    if (this.player) {
      this.player.unload();
      this.player.destroy();
      this.player = null;
    }
  },
};
</script>
<style>
.video-box {
  width: 70%;
  height: 70%;
}
video {
  width: 70%;
  height: 70%;
  object-fit: fill;
}
video::-webkit-media-controls-play-button {
  display: none;
}
video::-webkit-media-controls-current-time-display {
  display: none;
}
video::-webkit-media-controls-timeline {
  display: none;
}
</style>
