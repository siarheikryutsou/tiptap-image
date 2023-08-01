<template>
  <div id="editor">
    <editor-content :editor="editor"/>
    <input type="file" ref="elInputFile" style="display: none" @change="onFileChange"
           :accept="ALLOWED_TYPES.toString()"/>
    <button @click="openFileInput">Add img</button>
  </div>
</template>

<script setup lang="ts">
import {EditorContent, Editor} from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Image from '@tiptap/extension-image'
import {defineComponent, ref} from "vue";

const {} = defineComponent({
  components: {
    EditorContent,
  }
});

const editor: Editor = new Editor({
  content: '<p>Hello world</p>',
  extensions: [
    StarterKit,
    Image
  ],
});

const ALLOWED_TYPES: string[] = [
  "image/bmp",
  "image/gif",
  "image/jpeg",
  "image/jpg",
  "image/png",
  "image/tiff",
  "image/webp",
  "image/x-icon",
];

const elInputFile = ref<HTMLInputElement>();

const openFileInput = (): void => {
  elInputFile.value?.click();
}

const onFileChange = (): void => {

}


</script>

<style scoped>
#editor {
  width: 50%;
}

#editor :deep(.ProseMirror) {
  padding: 15px;
  border-radius: unset;
  min-height: 320px;
  border: 1px solid black;
}

#editor :deep(img) {
  max-width: 100%;
}

#editor :deep(img.ProseMirror-selectednode) {
  border: 1px solid red;
}

</style>