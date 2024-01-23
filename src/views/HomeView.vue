<script setup lang="ts">
import {createWorker, recognize} from 'tesseract.js';
import { onMounted, ref } from 'vue';
import { onBeforeRouteLeave } from 'vue-router';

const fileUrl = ref('');
const progress = ref(0);
const status = ref('');
const result = ref<string>();

const setup = async () => {
  const worker = await createWorker('eng');


};

const onFileChange = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) {
    return;
  }

  fileUrl.value = URL.createObjectURL(file);

  recognize(
    fileUrl.value,
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
    })
};

onMounted(() => {
  setTimeout(() => {
    console.log(cv);
    console.log(window.cv_ready);
  }, 2000);
})

onBeforeRouteLeave(() => {
  URL.revokeObjectURL(fileUrl.value);
});
</script>

<template>
  <div class="flex-column">
    <input
      type="file"
      accept="image/*"
      @change="onFileChange"
    />
    <div>
      <progress
        :value="progress"
        max="1"
      />
      status: {{status}}
    </div>
    result: {{result}}
  </div>
</template>

<style scoped lang="scss">
.flex-column {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}
</style>
