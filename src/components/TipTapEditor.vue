<template>
  <div id="editor">
    <editor-content :editor="editor"/>
    <input type="file" ref="fileInput" style="display: none" @change="onFileChange" :accept="ALLOWED_TYPES().toString()"/>
    <button @click="openFileInput">Add img</button>
  </div>
</template>

<script>
import {Editor, EditorContent} from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Image from "@tiptap/extension-image";
import {ALLOWED_TYPES, MAX_HEIGHT, MAX_SIZE_MB, MAX_WIDTH} from "@/shared/lib/ImgConsts";

export default {
  components: {
    EditorContent,
  },

  data() {
    return {
      editor: null,
    }
  },

  mounted() {
    this.editor = new Editor({
      content: '<p>I’m running Tiptap with Vue.js. 🎉</p>',
      extensions: [
        StarterKit,
        Image
      ],
      editorProps: {
        handlePaste: this.handlePaste
      }
    })
  },

  methods: {
    ALLOWED_TYPES() {
      return ALLOWED_TYPES
    },
    openFileInput() {
      this.$refs.fileInput.click();
    },

    onFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.editor.chain().focus().setImage({src: URL.createObjectURL(file)}).run();
      }
    },

    handlePaste(view, event, slice) {
      const items = Array.from(event.clipboardData?.items || []);
      for (const item of items) {
        if (ALLOWED_TYPES.includes(item.type)) {
          const file = item.getAsFile();
          let filesize = ((file.size / 1024) / 1024).toFixed(4);
          console.log(filesize);
          if (filesize < MAX_SIZE_MB) { // check image under 10MB
            let img = document.createElement("img");
            img.src = URL.createObjectURL(file);
            img.onload = () => {
              if (this.width > MAX_WIDTH || this.height > MAX_HEIGHT) {
                window.alert(`Your images need to be less than ${MAX_HEIGHT} pixels in height ${MAX_WIDTH} pixels and width.`); // display alert
              } else {
                this.uploadImage(file);
                this.editor.chain().focus().setImage({src: img.src}).run();
              }
            };
          } else {
            window.alert(`Images need to be in jpg or png format and less than ${MAX_SIZE_MB}mb in size.`);
          }
          return true;
        }
      }
      return false;
    },

    async uploadImage(file) {

    }

  },

  beforeUnmount() {
    this.editor.destroy();
  },
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
  border: 1px solid var(--color-border);
}

#editor :deep(img) {
  max-width: 100%;
}

#editor :deep(img.ProseMirror-selectednode) {
  border: 1px solid red;
}

</style>