<template>
  <div class="flex flex-col h-full w-full mt-2.5 box-border">
    <div
      class="h-[3.25rem] w-full justify-start p-0 m-0 border rounded-t-md border-b-0 box-border flex"
    >
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
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleStrike().run()"
          :disabled="!editor.can().chain().focus().toggleStrike().run()"
          :class="{ activeToggle: editor.isActive('strike') }"
          ><Strikethrough class="h-4 w-4"></Strikethrough
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleHighlight().run()"
          :disabled="!editor.can().chain().focus().toggleHighlight().run()"
          :class="{ activeToggle: editor.isActive('highlight') }"
          ><Highlighter class="h-4 w-4"></Highlighter
        ></Button>
      </div>
      <Separator orientation="vertical" />
      <div type="multiple" class="box-border p-1.5">
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().setParagraph().run()"
          :class="{ activeToggle: editor.isActive('paragraph') }"
          ><Type class="h-4 w-4"></Type
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleHeading({ level: 1 }).run()"
          :class="{ activeToggle: editor.isActive('heading', { level: 1 }) }"
          ><Heading1 class="h-4 w-4"></Heading1
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleHeading({ level: 2 }).run()"
          :class="{ activeToggle: editor.isActive('heading', { level: 2 }) }"
          ><Heading2 class="h-4 w-4"></Heading2
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleHeading({ level: 3 }).run()"
          :class="{ activeToggle: editor.isActive('heading', { level: 3 }) }"
          ><Heading3 class="h-4 w-4"></Heading3
        ></Button>
      </div>
      <Separator orientation="vertical" />
      <div type="multiple" class="box-border p-1.5">
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleBulletList().run()"
          :class="{ activeToggle: editor.isActive('bulletList') }"
          ><List class="h-4 w-4"></List
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleOrderedList().run()"
          :class="{ activeToggle: editor.isActive('orderedList') }"
          ><ListOrdered class="h-4 w-4"></ListOrdered
        ></Button>
        <Button
          variant="ghost"
          class="h-10 mr-1.5"
          @click="editor.chain().focus().toggleCodeBlock().run()"
          :disabled="!editor.can().chain().focus().toggleCodeBlock().run()"
          :class="{ activeToggle: editor.isActive('codeBlock') }"
          ><Code class="h-4 w-4"></Code
        ></Button>
      </div>
    </div>
    <editor-content class="h-full w-full prose max-w-none" :editor="editor" />
  </div>
</template>

<script>
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent } from '@tiptap/vue-3'
import Highlight from '@tiptap/extension-highlight'
import CodeBlock from '@tiptap/extension-code-block'
import '@tailwindcss/typography'
import { Button } from '@/components/ui/button'
import {
  Bold,
  Italic,
  Strikethrough,
  Code,
  Type,
  Heading1,
  Heading2,
  Heading3,
  List,
  ListOrdered,
  Highlighter
} from 'lucide-vue-next'
import { Separator } from '@/components/ui/separator'

export default {
  components: {
    EditorContent,
    Button,
    Bold,
    Italic,
    Strikethrough,
    Code,
    Separator,
    Type,
    Heading1,
    Heading2,
    Heading3,
    List,
    ListOrdered,
    Highlight,
    Highlighter
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
      extensions: [StarterKit, Highlight, CodeBlock],
      content: this.modelValue,
      onUpdate: () => {
        // HTML
        this.$emit('update:modelValue', this.editor.getHTML())
      }
    })
    Highlight.configure({
      HTMLAttributes: {
        class: 'my-custom-class'
      },
      multicolor: true
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
