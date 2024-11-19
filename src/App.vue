<template>
  <div
    ref="container"
    @dblclick="fullscreenSwich"
    style="
      width: 100%;
      height: 518px;
      min-height: 200px;
      background-color: #000000;
      margin: 0 auto;
      position: relative;
    "
  >
    <div class="buttons-box" id="buttonsBox">
      <div class="buttons-box-left">
        <i v-if="!playing" class="iconfont icon-play jessibuca-btn" @click="playBtnClick"></i>
        <i v-if="playing" class="iconfont icon-pause jessibuca-btn" @click="pause"></i>
        <i class="iconfont icon-stop jessibuca-btn" @click="destroy"></i>
        <i v-if="isNotMute" class="iconfont icon-audio-high jessibuca-btn" @click="mute()"></i>
        <i
          v-if="!isNotMute"
          class="iconfont icon-audio-mute jessibuca-btn"
          @click="cancelMute()"
        ></i>
      </div>
      <div class="buttons-box-right">
        <span class="jessibuca-btn">{{ kBps }} kb/s</span>
        <i
          class="iconfont icon-camera1196054easyiconnet jessibuca-btn"
          @click="screenshot"
          style="font-size: 1rem !important"
        ></i>
        <i class="iconfont icon-shuaxin11 jessibuca-btn" @click="playBtnClick"></i>
        <i
          v-if="!fullscreen"
          class="iconfont icon-weibiaoti10 jessibuca-btn"
          @click="fullscreenSwich"
        ></i>
        <i
          v-if="fullscreen"
          class="iconfont icon-weibiaoti11 jessibuca-btn"
          @click="fullscreenSwich"
        ></i>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, defineProps } from 'vue'

// Define props
const props = defineProps({
  videoUrl: String,
})

// Component state
const playing = ref(false)
const isNotMute = ref(true)
const fullscreen = ref(false)
const kBps = ref(0)
const btnDom = ref<HTMLElement | null>(null)
const videoInfo = ref(null)
const jessibucaPlayer: Record<string, any> = {}

// Get video URL from route params or props
const paramUrl = decodeURIComponent(props.videoUrl || '' || '')

// Create player instance
const create = () => {
  const options = {
    container: document.querySelector('#container') as HTMLElement,
    autoWasm: true,
    background: '',
    controlAutoHide: false,
    debug: false,
    decoder: '/js/jessibuca/decoder.js',
    forceNoOffscreen: false,
    hasAudio: true,
    heartTimeout: 5,
    heartTimeoutReplay: true,
    heartTimeoutReplayTimes: 3,
    hiddenAutoPause: false,
    hotKey: true,
    isFlv: false,
    isFullResize: false,
    isNotMute: isNotMute.value,
    isResize: false,
    keepScreenOn: true,
    loadingText: '请稍等, 视频加载中......',
    loadingTimeout: 10,
    loadingTimeoutReplay: true,
    loadingTimeoutReplayTimes: 3,
    openWebglAlignment: false,
    operateBtns: {
      fullscreen: false,
      screenshot: false,
      play: false,
      audio: false,
      record: false,
    },
    recordType: 'mp4',
    rotate: 0,
    showBandwidth: false,
    supportDblclickFullscreen: false,
    timeout: 10,
    useMSE: true,
    useWCS: location.hostname === 'localhost' || location.protocol === 'https:',
    useWebFullScreen: true,
    videoBuffer: 0.1,
    wasmDecodeErrorReplay: true,
    wcsUseVideoRender: true,
  }

  jessibucaPlayer[props.videoUrl!] = new window.Jessibuca(options)

  const player = jessibucaPlayer[props.videoUrl!]

  player.on('pause', () => (playing.value = false))
  player.on('play', () => (playing.value = true))
  player.on('fullscreen', (msg: boolean) => (fullscreen.value = msg))
  player.on('mute', (msg: boolean) => (isNotMute.value = !msg))
  player.on('performance', (performance: number) => {
    let show = '卡顿'
    if (performance === 2) show = '非常流畅'
    else if (performance === 1) show = '流畅'
    console.log('Performance:', show)
  })
  player.on('kBps', (kBpsValue: number) => (kBps.value = Math.round(kBpsValue)))
}

// Play video
const play = (url: string) => {
  if (url) {
    if (jessibucaPlayer[props.videoUrl!]) {
      destroy()
    }
    create()
    const player = jessibucaPlayer[props.videoUrl!]
    player.on('play', () => {
      playing.value = true
    })
    if (player.hasLoaded()) {
      player.play(url)
    } else {
      player.on('load', () => player.play(url))
    }
  }
}

// Pause video
const pause = () => {
  if (jessibucaPlayer[props.videoUrl!]) {
    jessibucaPlayer[props.videoUrl!].pause()
  }
  playing.value = false
}

// Take screenshot
const screenshot = () => {
  if (jessibucaPlayer[props.videoUrl!]) {
    jessibucaPlayer[props.videoUrl!].screenshot()
  }
}

// Mute video
const mute = () => {
  if (jessibucaPlayer[props.videoUrl!]) {
    jessibucaPlayer[props.videoUrl!].mute()
  }
}

// Cancel mute
const cancelMute = () => {
  if (jessibucaPlayer[props.videoUrl!]) {
    jessibucaPlayer[props.videoUrl!].cancelMute()
  }
}

// Destroy player instance
const destroy = () => {
  const player = jessibucaPlayer[props.videoUrl!]
  if (player) {
    player.destroy()
  }
  playing.value = false
  fullscreen.value = false
}

// Fullscreen switch
const fullscreenSwich = () => {
  const isFull = isFullscreen()
  jessibucaPlayer[props.videoUrl!].setFullscreen(!isFull)
  fullscreen.value = !isFull
}

// Check if in fullscreen mode
const isFullscreen = (): boolean => {
  return (
    document.fullscreenElement ||
    document.msFullscreenElement ||
    document.mozFullScreenElement ||
    document.webkitFullscreenElement ||
    false
  )
}

// Watch videoUrl change
watch(
  () => props.videoUrl,
  (newVal) => {
    if (newVal) play(newVal)
  },
  { immediate: true },
)

onMounted(() => {
  play('')
})
</script>

<style>
.buttons-box {
  width: 100%;
  height: 28px;
  background-color: rgba(43, 51, 63, 0.7);
  position: absolute;
  display: flex;
  left: 0;
  bottom: 0;
  user-select: none;
  z-index: 10;
}

.jessibuca-btn {
  width: 20px;
  color: rgb(255, 255, 255);
  line-height: 27px;
  margin: 0px 10px;
  padding: 0px 2px;
  cursor: pointer;
  text-align: center;
  font-size: 0.8rem !important;
}

.buttons-box-right {
  position: absolute;
  right: 0;
}
</style>
