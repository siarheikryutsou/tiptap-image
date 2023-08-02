<template>
  <div id="editor">
    <editor-content :editor="editor"/>
    <div class="controls">
      <input type="file" ref="elInputFile" style="display: none" @change="onFileSelect"
             :accept="allowedTypes.toString()" multiple/>
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

defineComponent({
  components: {
    EditorContent,
  }
});

const allowedTypes: string[] = [
  "image/gif",
  "image/jpeg",
  "image/jpg",
  "image/png",
  "image/tiff",
  "image/webp"
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
    onFilesInput(input.files);
  }
}

const onPaste = (view: unknown, event: ClipboardEvent): boolean => {
  const items: DataTransferItem[] = Array.from(event.clipboardData?.items || []);
  const files: File[] = [];

  for (const item of items) {
    if (isAllowedType(item.type)) {
      const file = item.getAsFile();
      if (file) files.push(file);
    }
  }

  if (files.length) {
    onFilesInput(files);
    return true;
  }

  return false;
}

const onDrop = (view: unknown, event: DragEvent, slice: unknown, moved: boolean): boolean => {
  console.log("onDrop");
  if (!moved && event.dataTransfer?.files?.length) {
    onFilesInput(event.dataTransfer.files);
    return true;
  }
  return false;
}


const onFilesInput = async (files: FileList | File[] | null): Promise<boolean> => {
  if (!files?.length) return false;
  const filesList: File[] = files instanceof FileList ? Array.from(files) : files;

  for (const file of filesList) {
    await checkFile(file)
        .then((url: string) => appendImage(url))
        .catch((reason: string) => alert(reason));
  }
  return false;
}


const isAllowedType = (type: string): boolean => {
  return allowedTypes.includes(type);
}


const checkFile = async (file: File | null): Promise<string> => {
  if (!file) {
    return Promise.reject(`File not exists`);
  }

  if (!isAllowedType(file.type)) {
    return Promise.reject(`Image ${file.name} need to be${allowedTypes.toString().split("image/").join(" ")}. Yur image is ${file.type.split("/")[1]}`);
  }

  const fileSizeInMb = file.size / 1024 / 1024;
  if (fileSizeInMb > MAX_FILE_SIZE_IN_MB) {
    return Promise.reject(`Image ${file.name} need to be less than ${MAX_FILE_SIZE_IN_MB}mb in size. Your file is ${fileSizeInMb.toFixed(2)}Mb`);
  }

  return await processFile(file);
}


const processFile = async (file: File): Promise<string> => {
  const img = document.createElement("img");
  img.src = URL.createObjectURL(file);
  return await new Promise((resolve) => {
    img.onload = () => {
      if (img.width > MAX_IMG_WIDTH || img.height > MAX_IMG_HEIGHT) {
        const url = resizeImage(img);
        resolve(url);
      } else {
        resolve(img.src);
      }
    }
  })
}


const appendImage = (url: string): void => {
    editor.commands.setImage({src: url});
    editor.commands.createParagraphNear()
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


const onSaveClick = (): void => {
  save();
}


const save = (): void => {
  console.log(editor.getJSON());
}


const editor: Editor = new Editor({
  content: "",
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