<template>
  <dialog ref="dialogRef">
    <p>{{ messageRef }}</p>
    <button @click="close()">Ok</button>
  </dialog>
</template>

<script setup lang="ts">
import {ref, defineExpose} from "vue";

const dialogRef = ref<HTMLDialogElement | null>(null);
let onCloseCallbackHandler:Function | undefined;
let messageRef = ref<string>("");
const show = (message:string, onCloseCallback:Function): void => {
  messageRef.value = message;
  onCloseCallbackHandler = onCloseCallback;
  dialogRef.value?.showModal();
}

const close = ():void => {
  dialogRef.value?.close();
  messageRef.value = "";
  onCloseCallbackHandler?.();
}

defineExpose({
  show
});

</script>

<style scoped>

  dialog {
    min-width: 250px;
    max-width: 400px;
  }

</style>