<template>
  <div id="editor">
    <div class="controls" ref="controlsRef">
      <input type="file" ref="elInputFile" style="display: none" @change="onFileSelect"
             :accept="allowedTypes.toString()" multiple/>
      <button ref="addImageButtonRef" @click="onAddImageButtonClick">Add img</button>
      <button @click="onSaveClick">Save</button>
      <ContextMenu v-show="showContextRef" ref="contextRef"/>
    </div>
    <editor-content :editor="editor"/>
  </div>
  <img v-if="showPreloader" class="preloader" src="/preloader.svg" alt="preloader"/>
  <AlertDialog v-show="showDialogRef" ref="alertRef"/>
  <URLDialog v-show="showURLDialogRef" ref="urlDialogRef"/>
</template>

<script setup lang="ts">
import {EditorContent, Editor} from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';
import Image from '@tiptap/extension-image';
import Focus from '@tiptap/extension-focus';
import {defineComponent, ref} from "vue";
import AlertDialog from "@/components/tip-tap-editor/AlertDialog.vue";
import ContextMenu from "@/components/tip-tap-editor/ContextMenu.vue";
import type {IContextData} from "@/components/tip-tap-editor/interfaces";
import URLDialog from "@/components/tip-tap-editor/URLDialog.vue";

defineComponent({
  components: {
    EditorContent
  }
});

const allowedTypes: string[] = [
  "image/gif",
  "image/jpeg",
  "image/png",
  "image/tiff",
  "image/webp"
];
const MAX_FILE_SIZE_IN_MB: number = 15;
const MAX_IMG_WIDTH: number = 2500;
const MAX_IMG_HEIGHT: number = 2500;
const URL_PATTERN:RegExp = new RegExp("^https?\\:\\/\\/[^/]+fantlab\\.(ru|org)\\/");
const elInputFile = ref<HTMLInputElement>();
const showPreloader = ref<boolean>(false);
const alertRef = ref<typeof AlertDialog>();
const showDialogRef = ref<boolean>(false);
const showContextRef = ref<boolean>(false);
const contextRef = ref<typeof ContextMenu>();
const addImageButtonRef = ref<HTMLButtonElement>()
const showURLDialogRef = ref<boolean>();
const urlDialogRef = ref<typeof URLDialog>();
const controlsRef = ref<HTMLElement>();
let id = 0;

const onAddImageButtonClick = (event: Event): void => {
  (event.target as HTMLButtonElement).disabled = true;
  contextRef.value?.show(addImageContext, onAddImageContextClosed);
  showContextRef.value = true;
}

const onAddImageContextClosed = (): void => {
  const button = addImageButtonRef.value;
  if (button) button.disabled = false;
  showContextRef.value = false;
}


const pickFile = (): void => {
  elInputFile.value?.click();
}

const pickURL = (): void => {
  showURLDialogRef.value = true;
  urlDialogRef.value?.show(onImageURL);
}


const addImageContext: IContextData = {
  "Загрузить с компьютера": pickFile,
  "Вставить URL": pickURL,
}


const showAlert = (message: string): void => {
  showDialogRef.value = true;
  alertRef.value?.show(message, onAlertClose);
}


const onAlertClose = (): void => {
  showDialogRef.value = false;
}


const onImageURL = (url: string): void => {
  showURLDialogRef.value = false;
  if (url) {
    appendImage(url);
  }
}


const onFileSelect = (event: Event): void => {
  const input: HTMLInputElement = event.target as HTMLInputElement;
  if (input && input.files) {
    onFilesInput(input.files);
    input.value = "";
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
        .catch((reason: string) => showAlert(reason));
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

  togglePreloader(true);

  return await processFile(file);
}


const processFile = async (file: File): Promise<string> => {
  const img = document.createElement("img");
  img.src = URL.createObjectURL(file);
  return await new Promise((resolve) => {
    img.onload = async () => {
      if (img.width > MAX_IMG_WIDTH || img.height > MAX_IMG_HEIGHT) {
        const blob = await resizeImage(img);
        resolve(URL.createObjectURL(blob));
      } else {
        resolve(img.src);
      }
    }
  })
}


const appendImage = (url: string): void => {
  editor.commands.setImage({src: url, alt: `editor-image-${(++id).toString()}`});
  editor.commands.createParagraphNear();
  togglePreloader(false);
}


const resizeImage = async (img: HTMLImageElement): Promise<Blob> => {
  const scaleRatio: number = Math.min(MAX_IMG_WIDTH / img.width, MAX_IMG_HEIGHT / img.height);
  const newWidth: number = Math.floor(img.width * scaleRatio);
  const newHeight: number = Math.floor(img.height * scaleRatio);
  //@ts-ignore
  const canvas: OffscreenCanvas = getImageCanvas(img, newWidth, newHeight);
  return await canvas.convertToBlob()
}


//@ts-ignore
const getImageCanvas = (img: HTMLImageElement, width?: number, height?: number): OffscreenCanvas | null => {
  width = width || img.width;
  height = height || img.height;

  //@ts-ignore
  const canvas: OffscreenCanvas = new OffscreenCanvas(width, height);
  const ctx: CanvasRenderingContext2D | null = canvas.getContext("2d");
  ctx?.drawImage(img, 0, 0, width, height);
  return canvas;
}


const togglePreloader = (force?: boolean): boolean => {
  if (force !== undefined) {
    showPreloader.value = force;
    return force;
  }
  showPreloader.value = !showPreloader.value;
  return showPreloader.value;
};


const onSaveClick = (): void => {
  save();
}


const imageToBase64 = async (img: HTMLImageElement): Promise<string> => {
  // @ts-ignore
  const canvas: OffscreenCanvas = getImageCanvas(img);
  const blob: Blob = await canvas.convertToBlob();

  return new Promise((resolve, reject) => {
    const reader: FileReader = new FileReader();
    reader.onloadend = () => {
      const base64 = reader.result?.toString();
      if (base64) {
        resolve(base64);
      } else {
        const error: Error = new Error("Не удалось получить Base64 изображения.");
        showAlert(error.message);
        reject(error);
      }
    }

    reader.onerror = () => {
      const error: Error = new Error("Ошибка при чтении Blob данных.");
      showAlert(error.message);
      reject(error);
    }

    reader.readAsDataURL(blob);
  })
}


const setEditable = (editable:boolean):void => {
  editor.setEditable(editable);
  if(controlsRef.value?.children) {
    const controlsList:Element[] = Array.from(controlsRef.value?.children);
    controlsList.forEach((el:Element) => {
      if(editable) {
        el.removeAttribute("disabled");
      } else {
        el.setAttribute("disabled", "true");
      }
    });
  }
}


const confirmLeave = (event:BeforeUnloadEvent):void => {
  event.preventDefault();
  event.returnValue = "";
}


const save = async (): Promise<void> => {
  togglePreloader(true);
  setEditable(false);
  window.addEventListener("beforeunload", confirmLeave);
  const json = {...editor.getJSON()};

  if (json.content) {
    for (let i = 0; i < json.content.length; i++) {
      const item = json.content[i];
      if (item.type === "image") {
        const currentSrc: string = item.attrs?.src;
        const imgID: string = item.attrs?.alt;
        const img = document.querySelector(`img[alt=${imgID}]`);
        if (img) {
          if (currentSrc.startsWith("blob:") || (currentSrc.startsWith("http") && !URL_PATTERN.test(currentSrc))) {
            item.attrs = {
              src: await imageToBase64(img as HTMLImageElement)
            }
          }
        }
      }
    }
  }

  fetch("/api/upload_article/", {
    method: "POST",
    body: JSON.stringify(json),
    headers: {
      "Content-Type": "application/json"
    }
  })
      .then(response => response.json())
      .then(data => {
        showAlert("Статья успешно загружена");
      })
      .catch((error) => {
        showAlert(`Ошибка при отправке изображения: ${error}`);
      })
      .finally(() => {
        togglePreloader(false);
        setEditable(true);
        window.removeEventListener("beforeunload", confirmLeave);
      });
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
  margin-bottom: 10px;
}

#editor .controls button {
  margin-right: 5px;
}

.preloader {
  position: fixed;
  top: 50%;
  left: 50%;
  margin: -100px;
}

</style>