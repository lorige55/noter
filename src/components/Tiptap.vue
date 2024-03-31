<template>
  <div class="flex flex-col h-full mt-2.5 w-full box-border">
    <div class="h-[3.25rem] justify-start p-0 m-0 border rounded-t-md border-b-0 box-border flex">
      <div type="multiple" class="box-border p-1.5">
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleBold().run()"
          :disabled="!editor.can().chain().focus().toggleBold().run()"
          :class="{ activeToggle: editor.isActive('bold') }"
          ><Bold class="h-4 w-4"></Bold
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleItalic().run()"
          :disabled="!editor.can().chain().focus().toggleItalic().run()"
          :class="{ activeToggle: editor.isActive('italic') }"
          ><Italic class="h-4 w-4"></Italic
        ></Button>
      </div>
    </div>
    <editor-content class="h-full" :editor="editor" />
  </div>
</template>

<script>
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent } from '@tiptap/vue-3'
import { Button } from '@/components/ui/button'
import { Bold, Italic } from 'lucide-vue-next'

export default {
  components: {
    EditorContent,
    Button,
    Bold,
    Italic
  },

  props: {
    modelValue: {
      type: String,
      default: ''
    }
  },

  emits: ['update:modelValue'],

  data() {
    return {
      editor: null
    }
  },

  watch: {
    modelValue(value) {
      // HTML
      const isSame = this.editor.getHTML() === value
      if (isSame) {
        return
      }
      this.editor.commands.setContent(value, false)
    }
  },

  mounted() {
    this.editor = new Editor({
      editorProps: {
        attributes: {
          class:
            'h-full justify-between flex-grow mb-2.5 w-full rounded-b-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50'
        }
      },
      extensions: [StarterKit],
      content: this.modelValue,
      onUpdate: () => {
        // HTML
        this.$emit('update:modelValue', this.editor.getHTML())
      }
    })
  },

  beforeUnmount() {
    this.editor.destroy()
  }
}
</script>
<style scoped>
.activeToggle {
  background-color: #f4f4f5;
}
</style>
