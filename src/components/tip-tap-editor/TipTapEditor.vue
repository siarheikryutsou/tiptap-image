<template>
  <div id="editor">
    <editor-content :editor="editor"/>
    <div class="controls">
      <input type="file" ref="elInputFile" style="display: none" @change="onFileSelect"
             :accept="allowedTypes.toString()"/>
      <button @click="onAddImageButtonClick">Add img</button>
      <button @click="onSaveClick">Save</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import {EditorContent, Editor} from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';
import Image from '@tiptap/extension-image';
import Focus from '@tiptap/extension-focus';
import {defineComponent, ref} from "vue";
import {EditorView} from "prosemirror-view";

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
const MAX_FILE_SIZE_IN_MB: number = 15;
const MAX_IMG_WIDTH: number = 2500;
const MAX_IMG_HEIGHT: number = 2500;
const elInputFile = ref<HTMLInputElement>();

const onAddImageButtonClick = (): void => {
  elInputFile.value?.click();
}

const onFileSelect = (event: Event): void => {
  const input: HTMLInputElement = event.target as HTMLInputElement;
  if (input && input.files) {
    const [file] = input.files;
    checkFile(file)
        .then(onFileValid)
        .catch((reason:string) => {
          console.error(reason);
        });
  }
}

const onPaste = (view:EditorView, event:ClipboardEvent):boolean => {
  console.log("onPaste")
  const items:DataTransferItem[] = Array.from(event.clipboardData?.items || []);
  for (const item of items) {
    if (allowedTypes.includes(item.type)) {
      const file = item.getAsFile();
      checkFile(file)
          .then(onFileValid)
          .catch((reason:string) => {
            console.error(reason);
          });
      return true;
    }
  }
  return false;
}

const onDrop = (view: EditorView, event: DragEvent, slice: unknown, moved: boolean): boolean => {
  console.log("onDrop");
  if (!moved && event.dataTransfer?.files?.length) {
    const [file] = event.dataTransfer.files;
    checkFile(file)
        .then(onFileValid)
        .catch((reason:string) => {
          console.error(reason);
        });
    return true;
  }
  return false;
}


const checkFile = async (file: File | null): Promise<string> => {
  return await new Promise((resolve, reject) => {
    if(!file) {
      reject("File not exists");
      return;
    }

    const fileSizeInMb = file.size / 1024 / 1024;
    if (fileSizeInMb > MAX_FILE_SIZE_IN_MB) {
      showFileSizeAlert(fileSizeInMb);
      reject("File size limit");
      return;
    }

    if (!allowedTypes.includes(file.type)) {
      showFileWrongTypeAlert(file.type);
      reject("Wrong file type");
      return;
    }

    const img = document.createElement("img");
    img.src = URL.createObjectURL(file);
    img.onload = () => {
      if (img.width > MAX_IMG_WIDTH || img.height > MAX_IMG_HEIGHT) {
        const url = resizeImage(img);
        resolve(url);
        return;
      } else {
        resolve(img.src);
        return;
      }
    }
  })
}


const onFileValid = (url: string) => {
  appendImage(url);
}


const appendImage = (url: string): void => {
  editor.chain().focus().setImage({src: url}).run();
}


const resizeImage = (img: HTMLImageElement): string => {
  const canvas: HTMLCanvasElement = document.createElement("canvas");
  const ctx: CanvasRenderingContext2D | null = canvas.getContext("2d");
  const scaleRatio: number = Math.min(MAX_IMG_WIDTH / img.width, MAX_IMG_HEIGHT / img.height);
  const newWidth: number = Math.floor(img.width * scaleRatio);
  const newHeight: number = Math.floor(img.height * scaleRatio);
  canvas.width = newWidth;
  canvas.height = newHeight;
  ctx?.drawImage(img, 0, 0, newWidth, newHeight);
  return canvas.toDataURL();
}


const showFileSizeAlert = (size: number): void => {
  alert(`Image need to be less than ${MAX_FILE_SIZE_IN_MB}mb in size. Your file is ${size.toFixed(2)}Mb`);
}


const showFileWrongTypeAlert = (type: string): void => {
  alert(`Image need to be${allowedTypes.toString().split("image/").join(" ")}. Yur image is ${type.split("/")[1]}`);
}


const onSaveClick = ():void => {
  save();
}


const save = ():void => {
  console.log(editor.getJSON());
}

console.log("setup")

const editor: Editor = new Editor({
  content: '<p>Hello world</p>',
  extensions: [
      StarterKit,
      Image.configure({
        HTMLAttributes: {
          class: "user-image"
        }
      }),
      Focus.configure({
        className: 'focus',
      })
  ],
  editorProps: {
    handlePaste: onPaste,
    handleDrop: onDrop
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

#editor :deep(.user-image) {
  max-width: 100%;
}

#editor :deep(.user-image.focus) {
  outline: solid 2px red;
  outline-offset: -2px;
}

#editor .controls {
  margin-top: 10px;
  & button {
    margin-right: 5px;
  }
}

</style>