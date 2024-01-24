<script setup lang="ts">
import { recognize } from 'tesseract.js';
import { computed, ref } from 'vue';
import { onBeforeRouteLeave } from 'vue-router';

const imgElement = ref();
const canvasElement = ref<HTMLCanvasElement>();
const fileUrl = ref('');
const progress = ref(0);
const status = ref('');
const result = ref<string>();
const cvReady = ref(false);

const openCvReadyHandler = () => {
  console.log('opencvIsReady');
  cvReady.value = true;
};

document.addEventListener('opencvIsReady', openCvReadyHandler);

const getOpencvReady = computed(() => cvReady.value ? 'opencv ready!' : 'opencv not ready!');
const isDisabled = computed(() => !cvReady.value);

const onFileChange = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) {
    return;
  }

  fileUrl.value = URL.createObjectURL(file);
};

const onLoadImage = () => {
  if (!imgElement) {
    return;
  }

  let src = window.cv.imread(imgElement.value);
  let dst = new window.cv.Mat();
  window.cv.cvtColor(src, dst, window.cv.COLOR_RGBA2GRAY);
  window.cv.imshow('canvasOutput', dst);

  const canvasFileUrl = canvasElement.value?.toDataURL('image/png');
  if (!fileUrl) {
    src.delete();
    dst.delete();
    return;
  }

  recognize(
    canvasFileUrl,
    'kor',
    {
      logger: m => {
        console.log(m);
        status.value = m.status;
        progress.value = m.progress;
      },
    }
  )
    .catch(err => console.error(err))
    .then((res) => {
      if (!res) {
        return;
      }

      result.value = res.data.text;
    }).finally(() => {
      src.delete();
      dst.delete();
    });
};

onBeforeRouteLeave(() => {
  URL.revokeObjectURL(fileUrl.value);
  document.removeEventListener('opencvIsReady', openCvReadyHandler);
});
</script>

<template>
  <div class="flex-column">
    <h2>{{getOpencvReady}}</h2>
    <input
      type="file"
      accept="image/*"
      @change="onFileChange"
      :disabled="isDisabled"
    />
    <div>
      <progress
        :value="progress"
        max="1"
      />
      status: {{status}}
    </div>
    <template v-if="fileUrl">
      <h3>원본 이미지</h3>
      <img
        ref="imgElement"
        alt="원본 이미지"
        :src="fileUrl"
        @load="onLoadImage"
      />
      <h3>그레이스케일 이미지</h3>
      <canvas
        ref="canvasElement"
        id="canvasOutput"
      />
      result: {{result}}
    </template>
  </div>
</template>

<style scoped lang="scss">
.flex-column {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  gap: 0.6rem;
}
</style>
