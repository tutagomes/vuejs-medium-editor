<template>
    <div>
        <!-- Editor Mode -->
        <div class="medium-editor-container" v-if="!readOnly">
            <insert-embed v-if="editor" 
                :uploadUrl="options.uploadUrl"
                :uploadUrlHeader="options.uploadUrlHeader"
                :file_input_name="options.file_input_name"
                :imgur_bool="options.imgur"
                :onChange="triggerChange"
                :editorRef="$refs.editor"
                :editor="editor"
                v-on:uploaded="uploadedCallback"
                :hideImage="options.hideImage"
                :hideGist="options.hideGist"
                ></insert-embed>
            <list-handler v-if="editor"
                :editor="editor"
                :onChange="triggerChange"></list-handler>
            <div class="editor" 
                v-bind:class="editorClass"
                v-html="prefill"
                ref="editor">
            </div>
        </div>
        <!-- Read Only Mode -->
        <read-mode v-if="readOnly" :content="prefill"></read-mode>
    </div>
</template>

<script>
import MediumEditor from 'medium-editor';
import InsertEmbed from './libs/InsertEmbed';
import ListHandler from './libs/ListHandler';
import ReadMode from './libs/ReadMode';
import _ from 'underscore';
export default {
  name: "medium-editor",
  data() {
    return {
      editor: null,
      defaultOptions: {
        forcePlainText: false,
        placeholder: {
          text: "Write something great!!"
        },
        uploadUrl: "https://api.imgur.com/3/image",
        uploadUrlHeader: {'Authorization': 'Client-ID db856b43cc7f441'},
        file_input_name: "image",
        imgur: true,
        youtubeFrame: '<iframe width="%w%" height="%h%" src="https://www.youtube.com/embed/%s%" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>',
        toolbar: {
          buttons: ["bold", "italic", "quote", "h1", "h2", "h3", "h4", "h5"]
        }
      },
      hasContent: false
    };
  },
  props: ["options", "onChange", "prefill", "readOnly"],
  computed: {
    editorOptions() {
      return _.extend(this.defaultOptions, this.options);
    },
    editorClass() {
        return {
            'has-content': this.hasContent
        }
    }
  },
  components: {
    InsertEmbed,
    ListHandler,
    ReadMode
  },
  mounted() {
    if (!this.readOnly) {
      this.createElm();
    }
  },
  methods: {
    createElm() {
      this.editor = new MediumEditor(this.$refs.editor, this.editorOptions);
      if (this.prefill) {
        if (/<[a-z][\s\S]*>/i.test(this.prefill)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
        this.$refs.editor.focus();
      }
      this.editor.subscribe("editableInput", this.triggerChange);
    },
    destroyElm() {
      this.editor.destroy();
    },
    addEmbededVideo (url, height = '300px', width = '100%') {
      let video_id = url.split('v=')[1];
      let ampersandPosition = video_id.indexOf('&');
      if(ampersandPosition != -1) {
        video_id = video_id.substring(0, ampersandPosition);
      }
      let content = this.editor.getContent() + this.defaultOptions.youtubeFrame.replace('%s%', video_id).replace('%h%', height).replace('%w%', width)
      this.editor.setContent(content)
    },
    triggerChange() {
      const content = this.editor.getContent();
      setTimeout(() => {
        if (/<[a-z][\s\S]*>/i.test(content)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
      }, 0);
      this.$emit("input", content);
      if (this.onChange) {
        this.onChange(content);
      }
    },
    uploadedCallback(url) {
      console.log("callback")
      this.$emit("uploaded", url);
    }
  },
  destroyed() {
    this.destroyElm();
  }
};
</script>