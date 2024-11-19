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
import { ref, onMounted, watch, nextTick, defineProps, defineEmits } from 'vue'

const props = defineProps({
  videoUrl: String,
})

const emit = defineEmits()

const container = ref<HTMLDivElement | null>(null)
const btnDom = ref<HTMLElement | null>(null)
const playing = ref(false)
const isNotMute = ref(false)
const fullscreen = ref(false)
const kBps = ref(0)
const performance = ref('')
const videoInfo = ref(null)

let jessibucaPlayer: Record<string, any> = {}

onMounted(() => {
  // let paramUrl = decodeURIComponent(window.location.href.split('?')[1]?.split('=')[1] || '');
  let paramUrl = 'http://192.168.1.39:8899/rtp/34020000001320000001_34020000001320000001.live.mp4'
  nextTick(() => {
    updatePlayerDomSize()
    window.onresize = () => {
      updatePlayerDomSize()
    }
    if (typeof props.videoUrl === 'undefined') {
      props.videoUrl = paramUrl
    }
    btnDom.value = document.getElementById('buttonsBox')
  })
})

watch(
  () => props.videoUrl,
  (val) => {
    nextTick(() => {
      play(val)
    })
  },
  { immediate: true },
)

const updatePlayerDomSize = () => {
  if (container.value) {
    container.value.width = '700px'
    container.value.height = '393.75px'
  }
}

const create = () => {
  const options = {
    container: container.value,
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

  console.log('Jessibuca -> options: ', options)
  jessibucaPlayer[container.value?.id ?? ''] = new window.Jessibuca({ ...options })

  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  jessibuca.on('pause', () => {
    playing.value = false
  })
  jessibuca.on('play', () => {
    playing.value = true
  })
  jessibuca.on('fullscreen', (msg: boolean) => {
    fullscreen.value = msg
  })
  jessibuca.on('mute', (msg: boolean) => {
    isNotMute.value = !msg
  })
  jessibuca.on('performance', (performanceValue: number) => {
    performance.value =
      performanceValue === 2 ? '非常流畅' : performanceValue === 1 ? '流畅' : '卡顿'
  })
  jessibuca.on('kBps', (kBpsValue: number) => {
    kBps.value = Math.round(kBpsValue)
  })
  jessibuca.on('videoInfo', (msg: any) => {
    console.log('Jessibuca -> videoInfo: ', msg)
  })
}

const playBtnClick = () => {
  play(props.videoUrl)
}

const play = (url: string) => {
  url = 'http://192.168.1.39:8899/rtp/34020000001320000001_34020000001320000001.live.mp4'
  if (url) {
    if (jessibucaPlayer[container.value?.id ?? '']) {
      destroy()
    }
    create()
    const jessibuca = jessibucaPlayer[container.value?.id ?? '']
    jessibuca.on('play', () => {
      playing.value = true
    })
    if (jessibuca.hasLoaded()) {
      jessibuca.play(url)
    } else {
      jessibuca.on('load', () => {
        jessibuca.play(url)
      })
    }
  }
  console.log('Jessibuca -> url: ', url)
}

const pause = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.pause()
  }
  playing.value = false
}

const screenshot = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.screenshot()
  }
}

const mute = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.mute()
  }
}

const cancelMute = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.cancelMute()
  }
}

const destroy = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.destroy()
  }
  if (btnDom.value && container.value) {
    container.value.appendChild(btnDom.value)
  }
  jessibucaPlayer[container.value?.id ?? ''] = null
  playing.value = false
}

const fullscreenSwich = () => {
  const isFull = isFullscreen()
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.setFullscreen(!isFull)
  }
  fullscreen.value = !isFull
}

const isFullscreen = () => {
  return (
    document.fullscreenElement ||
    document.msFullscreenElement ||
    document.mozFullScreenElement ||
    document.webkitFullscreenElement ||
    false
  )
}
</script>

<style scoped>
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
  right: 10px;
}

.buttons-box-left {
  position: absolute;
  left: 10px;
}
</style>
