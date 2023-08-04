<template>
  <div class="context-menu" ref="containerRef">

  </div>
</template>

<script setup lang="ts">
import {ref, defineExpose} from "vue";
import type {IContextData} from "@/components/tip-tap-editor/interfaces";

const containerRef = ref<HTMLElement>();
let closeCallbackHandler:Function | undefined;
let onWindowBlurHandler:EventListenerOrEventListenerObject;
let onWindowMouseDownHandler:EventListenerOrEventListenerObject;


const show = (data:IContextData, closeCallback:Function): void => {
  const el = containerRef.value;
  closeCallbackHandler = closeCallback;
  if(el?.innerHTML !== undefined) {
    el.innerHTML = "";
    Object.keys(data).forEach((label:string) => {
      const button:HTMLElement = document.createElement("button");
      button.innerHTML = label;
      button.onclick = () => {
        data[label]();
        closeCallback();
      }
      el.append(button);
    });
  }
  onWindowBlurHandler = () => {
    close();
  }

  onWindowMouseDownHandler = (event:Event) => {
    onWindowMouseDown(event);
  }

  window.addEventListener("blur", onWindowBlurHandler);
  window.addEventListener("mousedown", onWindowMouseDownHandler);
}


const close = ():void => {
  if(onWindowBlurHandler) window.removeEventListener("blur", onWindowBlurHandler);
  if(onWindowMouseDownHandler) window.removeEventListener("mousedown", onWindowMouseDownHandler);
  closeCallbackHandler?.();
}


const onWindowMouseDown = (event:Event):void => {
  const target = event.target;
  const menu = containerRef.value;
  if(menu && target && !menu?.contains(target as HTMLElement)) {
    close();
  }
}


defineExpose({
  show
});

</script>

<style scoped>
  .context-menu {
    background: white;
    border: 1px solid black;
    position: absolute;
    z-index: 11111;
  }

  .context-menu :deep(button) {
    background: transparent;
    width: 100%;
    border: none;
    text-align: left;
  }

  .context-menu :deep(button):hover {
    background: #d2d2d2;
  }

</style>