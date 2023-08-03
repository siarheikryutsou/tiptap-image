<template>
  <dialog ref="dialogRef">
    <p>{{ messageValue }}</p>
    <button class="btn-ok" @click="close()">Ok</button>
  </dialog>
</template>

<script setup lang="ts">
import {ref, defineExpose} from "vue";

const dialogRef = ref<HTMLDialogElement | null>(null);
let messageValue:string = "";
const show = (message:string): void => {
  messageValue = message;
  dialogRef.value?.showModal();
}

const close = ():void => {
  dialogRef.value?.close();
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

  .btn-ok {
    margin-top: 20px;
    float: right;
    min-width: 50px;
  }

</style>