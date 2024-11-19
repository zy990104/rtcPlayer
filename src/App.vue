<template>
  <div
    ref="container"
    @dblclick="fullscreenSwitch"
    style="
      width: 100%;
      height: 518px;
      min-height: 200px;
      background-color: #000000;
      margin: 0 auto;
      position: relative;
    "
  >
    <div class="buttons-box">
      <i v-if="!playing" class="iconfont icon-play jessibuca-btn" @click="playBtnClick"></i>
      <i v-if="playing" class="iconfont icon-pause jessibuca-btn" @click="pause"></i>
      <i
        v-if="!fullscreen"
        class="iconfont icon-weibiaoti10 jessibuca-btn"
        @click="fullscreenSwitch"
      ></i>
      <i
        v-if="fullscreen"
        class="iconfont icon-weibiaoti11 jessibuca-btn"
        @click="fullscreenSwitch"
      ></i>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, defineProps } from 'vue'

const props = defineProps({
  videoUrl: String,
})

const container = ref<HTMLDivElement | null>(null)
const playing = ref(false)
const fullscreen = ref(false)

let jessibucaPlayer: Record<string, any> = {}

onMounted(() => {
  nextTick(() => {
    if (typeof props.videoUrl !== 'undefined') {
      play(props.videoUrl)
    }
    play('http://192.168.1.39:8899/rtp/34020000001320000001_34020000001320000001.live.mp4')
  })
})

const createPlayer = (url: string) => {
  const options = {
    container: container.value,
    autoWasm: true,
    hasAudio: true,
    useMSE: true,
    useWebFullScreen: true,
    videoBuffer: 0.1,
  }

  jessibucaPlayer[container.value?.id ?? ''] = new window.Jessibuca({ ...options })

  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  jessibuca.on('play', () => {
    playing.value = true
  })
  jessibuca.on('pause', () => {
    playing.value = false
  })

  if (jessibuca.hasLoaded()) {
    jessibuca.play(url)
  } else {
    jessibuca.on('load', () => {
      jessibuca.play(url)
    })
  }
}

const playBtnClick = () => {
  if (props.videoUrl) {
    play(props.videoUrl)
  }
}

const play = (url: string) => {
  if (jessibucaPlayer[container.value?.id ?? '']) {
    destroy()
  }
  createPlayer(url)
}

const pause = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.pause()
  }
  playing.value = false
}

const fullscreenSwitch = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.setFullscreen(!fullscreen.value)
  }
  fullscreen.value = !fullscreen.value
}

const destroy = () => {
  const jessibuca = jessibucaPlayer[container.value?.id ?? '']
  if (jessibuca) {
    jessibuca.destroy()
  }
  jessibucaPlayer[container.value?.id ?? ''] = null
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
