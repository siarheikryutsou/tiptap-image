<template>
  <dialog ref="dialogRef">
    <h3>Вставка изображения</h3>
    <input type="url" ref="inputRef" placeholder="https://example.com/image.png" required @input="onInput" />
    <div class="controls">
      <button @click="close()">ОТМЕНА</button>
      <button ref="submitRef" @click="submit()" disabled>ВСТАВИТЬ ИЗОБРАЖЕНИЕ</button>
    </div>
  </dialog>
</template>

<script setup lang="ts">
import {ref, defineExpose} from "vue";

const dialogRef = ref<HTMLDialogElement>();
const inputRef = ref<HTMLInputElement>();
const submitRef = ref<HTMLButtonElement>();
let onSubmitCallbackHandler:(value:string) => void;

const show = (onSubmitCallback:(value:string) => void): void => {
  onSubmitCallbackHandler = onSubmitCallback;
  dialogRef.value?.showModal();
  inputRef.value?.focus();
}

const close = ():void => {
  dialogRef.value?.close();
  onSubmitCallbackHandler("");
  if(inputRef.value?.value) inputRef.value.value = "";
}


const submit = ():void => {
  onSubmitCallbackHandler(inputRef.value?.value || "");
  close();
}

const onInput = (event:Event):void => {
  const value = inputRef.value?.value;
  const button = submitRef.value;
  if(button) button.disabled = !(value && value.startsWith("http"));
}


defineExpose({
  show
});

</script>

<style scoped>
  dialog {
    width: 320px;
    height: 130px;
  }

  h3 {
    margin-bottom: 10px;
  }

  input {
    width: 100%;
  }

  .controls {
    display: flex;
    justify-content: flex-end;
  }

  .controls button:not(:last-child) {
    margin-right: 10px;
  }

</style>