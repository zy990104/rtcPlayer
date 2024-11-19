<template>
  <div id="rtcPlayer">
    <video id="webRtcPlayerBox" controls autoPlay style="text-align:left;">
      Your browser is too old which doesn't support HTML5 video.
    </video>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, defineProps, defineEmits, onBeforeUnmount } from 'vue';

// Define props
const props = defineProps({
  videoUrl: String,
  error: String,
  hasaudio: Boolean
});

// Define emits
const emit = defineEmits<{
  (event: 'eventcallbacK', type: string, message: string): void;
}>();

// Declare the RTC player and timer using refs
const webrtcPlayer = ref<ZLMRTCClient.Endpoint | null>(null);
const timer = ref<NodeJS.Timeout | null>(null);

// Function to initialize and play the video
const play = (url: string) => {
  if (webrtcPlayer.value) webrtcPlayer.value.close(); // Close any existing player

  webrtcPlayer.value = new ZLMRTCClient.Endpoint({
    element: document.getElementById('webRtcPlayerBox'), // video 标签
    debug: true, // 是否打印日志
    zlmsdpUrl: url, // 流地址
    simulecast: false,
    useCamera: true,
    audioEnable: true,
    videoEnable: true,
    recvOnly: true,
  });

  webrtcPlayer.value.on(ZLMRTCClient.Events.WEBRTC_ICE_CANDIDATE_ERROR, () => {
    console.error('ICE 协商出错');
    emit('eventcallbacK', 'ICE ERROR', 'ICE 协商出错');
  });

  webrtcPlayer.value.on(ZLMRTCClient.Events.WEBRTC_ON_REMOTE_STREAMS, (e: any) => {
    console.log('播放成功', e.streams);
    emit('eventcallbacK', 'playing', '播放成功');
  });

  webrtcPlayer.value.on(ZLMRTCClient.Events.WEBRTC_OFFER_ANWSER_EXCHANGE_FAILED, (e: any) => {
    console.error('offer anwser 交换失败', e);
    emit('eventcallbacK', 'OFFER ANSWER ERROR', 'offer anwser 交换失败');
    if (e.code === -400 && e.msg === '流不存在') {
      console.log('流不存在');
      timer.value = setTimeout(() => {
        webrtcPlayer.value?.close();
        play(url);
      }, 100);
    }
  });

  webrtcPlayer.value.on(ZLMRTCClient.Events.WEBRTC_ON_LOCAL_STREAM, () => {
    emit('eventcallbacK', 'LOCAL STREAM', '获取到了本地流');
  });
};

// Watch for changes to videoUrl prop
watch(() => props.videoUrl, (newData) => {
  if (webrtcPlayer.value) {
    webrtcPlayer.value.close();
  }
  play(newData);
}, { immediate: true });

// onMounted lifecycle hook
onMounted(() => {
  const paramUrl = decodeURIComponent(props.videoUrl || '');
  console.log('初始化时的地址为: ', paramUrl);
  play(paramUrl);
});

// Cleanup when component is destroyed
onBeforeUnmount(() => {
  if (timer.value) {
    clearTimeout(timer.value);
  }
  if (webrtcPlayer.value) {
    webrtcPlayer.value.close();
  }
});
</script>

<style scoped>
#rtcPlayer {
  width: 100%;
}

#webRtcPlayerBox {
  width: 100%;
  max-height: 56vh;
  background-color: #000;
}
</style>
