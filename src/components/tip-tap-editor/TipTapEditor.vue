<template>
  <div id="editor">
    <editor-content :editor="editor"/>
    <input type="file" ref="elInputFile" style="display: none" @change="onFileSelect"
           :accept="allowedTypes.toString()"/>
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


const allowedTypes: string[] = [
  "image/bmp",
  "image/gif",
  "image/jpeg",
  "image/jpg",
  "image/png",
  "image/tiff",
  "image/webp",
  "image/x-icon",
];
const MAX_FILE_SIZE_IN_MB:number = 15;
const MAX_IMG_WIDTH:number = 2500;
const MAX_IMG_HEIGHT:number = 2500;
const elInputFile = ref<HTMLInputElement>();

const openFileInput = (): void => {
  elInputFile.value?.click();
}

const onFileSelect = (event:Event): void => {
  const input:HTMLInputElement = event.target as HTMLInputElement;
  if(input && input.files) {
    const [file] = input.files;
    if (file) {
      checkFile(file).then((url:string) => {
        appendImage(url);
      });
    }
  }
}

const onPaste = ():void => {

}

const checkFile = async (file:File):Promise<string> => {
  return await new Promise((resolve, reject) => {
    const fileSizeInMb = file.size / 1024 / 1024;
    if(fileSizeInMb > MAX_FILE_SIZE_IN_MB) {
      showFileSizeAlert(fileSizeInMb);
      reject();
    }
    
    if(!allowedTypes.includes(file.type)) {
      showFileWrongTypeAlert(file.type);
      reject();
    }

    const img = document.createElement("img");
    img.src = URL.createObjectURL(file);
    img.onload = () => {
      if(img.width > MAX_IMG_WIDTH || img.height > MAX_IMG_HEIGHT) {
        const url = resizeImage(img);
        resolve(url);
      } else {
        resolve(img.src);
      }
    }
  })
}


const appendImage = (url:string):void => {
  editor.chain().focus().setImage({src: url}).run();
}


const resizeImage = (img:HTMLImageElement):string => {
  const canvas:HTMLCanvasElement | undefined = document.createElement("canvas");
  const ctx:CanvasRenderingContext2D | null = canvas.getContext("2d");
  const scaleRatio:number = Math.min(MAX_IMG_WIDTH / img.width, MAX_IMG_HEIGHT / img.height);
  const newWidth:number = Math.floor(img.width * scaleRatio);
  const newHeight:number = Math.floor(img.height * scaleRatio);
  canvas.width = newWidth;
  canvas.height = newHeight;
  ctx?.drawImage(img, 0, 0, newWidth, newHeight);
  return canvas.toDataURL();
}


const showFileSizeAlert = (size:number):void => {
  alert(`Image need to be less than ${MAX_FILE_SIZE_IN_MB}mb in size. Your file is ${size.toFixed(2)}Mb`);
}


const showFileWrongTypeAlert = (type:string):void => {
  alert(`Image need to be${allowedTypes.toString().split("image/").join(" ")}. Yur image is ${type.split("/")[1]}`);
}


const editor: Editor = new Editor({
  content: '<p>Hello world</p>',
  extensions: [
    StarterKit,
    Image
  ],
  editorProps: {
    handlePaste: onPaste
  }
});

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
  outline: solid 2px red;
  outline-offset: -2px;
}

</style>