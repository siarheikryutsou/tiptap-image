<template>
  <div id="editor">
    <editor-content :editor="editor"/>
  </div>
</template>

<script>
import {Editor, EditorContent} from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'

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
        handlePaste: function(view, event, slice) {
          const items = Array.from(event.clipboardData?.items || []);
          console.log(items);
          for (const item of items) {
            if (item.type.indexOf("image") === 0) {
              const file = item.getAsFile();
              let filesize = ((file.size/1024)/1024).toFixed(4); // get the filesize in MB
              console.log(filesize);
              if (filesize < 10) { // check image under 10MB
                // check the dimensions
                let _URL = window.URL || window.webkitURL;
                let img = new Image(); /* global Image */
                img.src = _URL.createObjectURL(item);
                img.onload = function () {
                  if (this.width > 5000 || this.height > 5000) {
                    window.alert("Your images need to be less than 5000 pixels in height and width."); // display alert
                  } else {
                    // valid image so upload to server
                    // uploadImage will be your function to upload the image to the server or s3 bucket somewhere
                    uploadImage(file).then(function(response) { // response is the image url for where it has been saved
                      // do something with the response
                    }).catch(function(error) {
                      if (error) {
                        window.alert("There was a problem uploading your image, please try again.");
                      }
                    });
                  }
                };
              } else {
                window.alert("Images need to be in jpg or png format and less than 10mb in size.");
              }
              return true; // handled
            }
          }
          return false; // not handled use default behaviour
        }
      }
    })
  },

  beforeUnmount() {
    this.editor.destroy()
  },
}
</script>

<style scoped>
  #editor {
    width: 50%;
    border: 1px solid var(--color-border);
  }
  #editor :deep(.ProseMirror) {
    padding: 15px;
    border-radius: unset;
    min-height: 320px;
  }
</style>