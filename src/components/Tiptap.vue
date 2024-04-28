<template>
  <div class="flex flex-col h-full w-full mt-2.5 box-border">
    <div
      class="h-[3.25rem] w-full justify-start p-0 m-0 border rounded-t-md border-b-0 box-border flex"
    >
      <div type="multiple" class="box-border p-1.5">
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5">
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                :value="editor.getAttributes('textStyle').color"
                @click.prevent
              />
            </Button>
          </PopoverTrigger>
          <PopoverContent>
            <!--Black-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#000000' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#000000"
                @click.prevent="editor.chain().focus().setColor('#000000').run()"
              />
            </Button>
            <!--Red-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#fc0707' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#fc0707"
                @click.prevent="editor.chain().focus().setColor('#fc0707').run()"
              />
            </Button>
            <!--Orange-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#fc8a07' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#fc8a07"
                @click.prevent="editor.chain().focus().setColor('#fc8a07').run()"
              />
            </Button>
            <!--Yellow-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#fcf007' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#fcf007"
                @click.prevent="editor.chain().focus().setColor('#fcf007').run()"
              />
            </Button>
            <!--Green-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#13fc07' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#13fc07"
                @click.prevent="editor.chain().focus().setColor('#13fc07').run()"
              />
            </Button>
            <!--Blue-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#0e02f7' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#0e02f7"
                @click.prevent="editor.chain().focus().setColor('#0e02f7').run()"
              />
            </Button>
            <!--Purple-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#a207fc' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#a207fc"
                @click.prevent="editor.chain().focus().setColor('#a207fc').run()"
              />
            </Button>
            <!--Pink-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#f807fc' }) }"
            >
              <input
                type="color"
                class="h-[25px] w-[25px] bg-transparent cursor-pointer"
                value="#f807fc"
                @click.prevent="editor.chain().focus().setColor('#f807fc').run()"
              />
            </Button>
          </PopoverContent>
        </Popover>
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5"><Type class="h-4 w-4"></Type></Button>
          </PopoverTrigger>
          <PopoverContent class="w-50 p-2.5">
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
              @click="editor.chain().focus().toggleUnderline().run()"
              :disabled="!editor.can().chain().focus().toggleUnderline().run()"
              :class="{ activeToggle: editor.isActive('underline') }"
              ><UnderlineIcon class="h-4 w-4"></UnderlineIcon
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
            <Separator orientation="horizontal" class="my-2.5" />
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().setParagraph().run()"
              :class="{ activeToggle: editor.isActive('paragraph') }"
              ><RemoveFormatting class="h-4 w-4"></RemoveFormatting
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
            <Separator orientation="horizontal" class="my-2.5" />
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
          </PopoverContent>
        </Popover>
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5"
              ><AlignCenter
                v-if="editor.isActive({ textAlign: 'center' })"
                class="w-4 h-4"
              ></AlignCenter>
              <AlignJustify
                v-else-if="editor.isActive({ textAlign: 'justify' })"
                class="w-4 h-4"
              ></AlignJustify>
              <AlignRight
                v-else-if="editor.isActive({ textAlign: 'right' })"
                class="w-4 h-4"
              ></AlignRight>
              <AlignLeft v-else class="w-4 h-4"></AlignLeft>
            </Button>
          </PopoverTrigger>
          <PopoverContent class="w-full">
            <Button
              v-if="!editor.isActive({ textAlign: 'left' })"
              @click="editor.chain().focus().setTextAlign('left').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'left' }) }"
              ><AlignLeft class="w-4 h-4"></AlignLeft
            ></Button>
            <Button
              v-if="!editor.isActive({ textAlign: 'center' })"
              @click="editor.chain().focus().setTextAlign('center').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'center' }) }"
              ><AlignCenter class="w-4 h-4"></AlignCenter
            ></Button>
            <Button
              v-if="!editor.isActive({ textAlign: 'justify' })"
              @click="editor.chain().focus().setTextAlign('justify').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'justify' }) }"
              ><AlignJustify class="w-4 h-4"></AlignJustify
            ></Button>
            <Button
              v-if="!editor.isActive({ textAlign: 'right' })"
              @click="editor.chain().focus().setTextAlign('right').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'right' }) }"
              ><AlignRight class="w-4 h-4"></AlignRight></Button></PopoverContent
        ></Popover>
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5"><Plus class="h-4 w-4"></Plus> </Button>
          </PopoverTrigger>
          <PopoverContent>
            <Button variant="ghost" class="h-10 mr-1.5" @click="addYoutubeVideo(1)"
              ><Film class="h-4 w-4"></Film
            ></Button>
            <AlertDialog :open="showYouTubeDialog">
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Add a YouTube Video</AlertDialogTitle>
                  <AlertDialogDescription>
                    Enter a URL to a YouTube video to embed it.
                  </AlertDialogDescription>
                  <Input
                    type="url"
                    placeholder="eg. 'https://www.youtube.com/watch?v=dQw4w9WgXcQ&pp=ygUXbmV2ZXIgZ29ubmEgZ2l2ZSB5b3UgdXA%3D'"
                    v-model="youtubeURL"
                  />
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel @click="addYoutubeVideo(3)">Cancel</AlertDialogCancel>
                  <AlertDialogAction @click="addYoutubeVideo(2)">Add</AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
            <Button variant="ghost" class="h-10 mr-1.5" @click="addImage(1)"
              ><ImageIcon class="h-4 w-4"></ImageIcon
            ></Button>
            <AlertDialog :open="showImageDialog">
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Add an Image</AlertDialogTitle>
                  <AlertDialogDescription>
                    Enter a URL to, or upload an image. Please note that uploaded images have to be
                    under 5MB.
                  </AlertDialogDescription>
                  <div class="flex w-full items-center gap-1.5">
                    <Input type="url" placeholder="Enter a URL to an image" v-model="imageURL" />
                    <Button @click="addImage(2)" type="submit"> Add </Button>
                  </div>
                  <div class="flex w-full items-center gap-1.5">
                    <Input id="imageUpload" type="file" accept="image/*" />
                    <Button @click="addImage(3)" type="submit"> Upload </Button>
                  </div>
                  <AlertDialogDescription
                    >Images themselves are not encrypted.</AlertDialogDescription
                  >
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel @click="addImage(4)">Cancel</AlertDialogCancel>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleCodeBlock().run()"
              :disabled="!editor.can().chain().focus().toggleCodeBlock().run()"
              :class="{ activeToggle: editor.isActive('codeBlock') }"
              ><Code class="h-4 w-4"></Code
            ></Button>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleBlockquote().run()"
              :disabled="!editor.can().chain().focus().toggleBlockquote().run()"
              :class="{ activeToggle: editor.isActive('blockquote') }"
              ><TextQuote class="h-4 w-4"></TextQuote
            ></Button> </PopoverContent
        ></Popover>
      </div>
    </div>
    <bubble-menu
      :editor="editor"
      :tippy-options="{ duration: 100 }"
      v-if="editor"
      class="border rounded-md bg-white overflow-hidden shadow-md"
    >
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleBold().run()"
        :disabled="!editor.can().chain().focus().toggleBold().run()"
        :class="{ activeToggle: editor.isActive('bold') }"
        ><Bold class="h-4 w-4"></Bold
      ></Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleItalic().run()"
        :disabled="!editor.can().chain().focus().toggleItalic().run()"
        :class="{ activeToggle: editor.isActive('italic') }"
        ><Italic class="h-4 w-4"></Italic
      ></Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleUnderline().run()"
        :disabled="!editor.can().chain().focus().toggleUnderline().run()"
        :class="{ activeToggle: editor.isActive('underline') }"
        ><UnderlineIcon class="h-4 w-4"></UnderlineIcon
      ></Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleStrike().run()"
        :disabled="!editor.can().chain().focus().toggleStrike().run()"
        :class="{ activeToggle: editor.isActive('strike') }"
        ><Strikethrough class="h-4 w-4"></Strikethrough
      ></Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleHighlight().run()"
        :disabled="!editor.can().chain().focus().toggleHighlight().run()"
        :class="{ activeToggle: editor.isActive('highlight') }"
        ><Highlighter class="h-4 w-4"></Highlighter
      ></Button>
    </bubble-menu>
    <editor-content class="h-full w-full prose max-w-none" :editor="editor" />
    <div class="h-10 flex items-center border-x border-b rounded-b-md">
      <p class="text text-sm ml-[7.5pt]" style="opacity: 0.5">
        You've typed <b>{{ editor.storage.characterCount.characters() }} Characters </b> or
        <b>{{ editor.storage.characterCount.words() }} Words</b>.
      </p>
    </div>
  </div>
</template>

<script>
//Tiptap imports
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent, BubbleMenu } from '@tiptap/vue-3'
//Tiptap extensions
import Highlight from '@tiptap/extension-highlight'
import CodeBlock from '@tiptap/extension-code-block'
import Underline from '@tiptap/extension-underline'
import Typography from '@tiptap/extension-typography'
import TextStyle from '@tiptap/extension-text-style'
import { Color } from '@tiptap/extension-color'
import TextAlign from '@tiptap/extension-text-align'
import CharacterCount from '@tiptap/extension-character-count'
import Youtube from '@tiptap/extension-youtube'
import Image from '@tiptap/extension-image'
//Firebase imports
import { getStorage, ref, uploadBytes, getDownloadURL } from 'firebase/storage'
//Tailwind imports
import '@tailwindcss/typography'
//Lucide
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
  Highlighter,
  Underline as UnderlineIcon,
  RemoveFormatting,
  Plus,
  TextQuote,
  AlignCenter,
  AlignJustify,
  AlignLeft,
  AlignRight,
  Film,
  Image as ImageIcon
} from 'lucide-vue-next'
//Shadcn Imports
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Separator } from '@/components/ui/separator'
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover'
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger
} from '@/components/ui/alert-dialog'

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
    Highlighter,
    UnderlineIcon,
    Popover,
    PopoverContent,
    PopoverTrigger,
    RemoveFormatting,
    Plus,
    TextQuote,
    AlignCenter,
    AlignJustify,
    AlignLeft,
    AlignRight,
    BubbleMenu,
    Film,
    AlertDialog,
    AlertDialogAction,
    AlertDialogCancel,
    AlertDialogContent,
    AlertDialogDescription,
    AlertDialogFooter,
    AlertDialogHeader,
    AlertDialogTitle,
    AlertDialogTrigger,
    Input,
    ImageIcon
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
      editor: null,
      userId: '',
      showYouTubeDialog: false,
      youtubeURL: '',
      showImageDialog: false,
      imageURL: '',
      downloadURL: '',
      firebaseConfig: {
        apiKey: 'AIzaSyBV9FOnKKOBNLuQsCn9T4OdfxT39cRhF6g',
        authDomain: 'noter-6e08f.firebaseapp.com',
        projectId: 'noter-6e08f',
        storageBucket: 'noter-6e08f.appspot.com',
        messagingSenderId: '967538971502',
        appId: '1:967538971502:web:6d31288c8ade545465277f'
      }
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

  methods: {
    generateKey() {
      //get variables
      let charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+*#&=!$£-_.:,;'
      let charsetLength = charset.length
      let charCount = 5
      //generate a random key
      let key = ''
      for (let i = 0; i < charCount; i++) {
        key += charset.charAt(Math.floor(Math.random() * charsetLength))
      }
      return key
    },
    addYoutubeVideo(a) {
      if (a == 1) {
        this.showYouTubeDialog = true
      } else if (a == 2) {
        this.showYouTubeDialog = false
        this.editor.commands.setYoutubeVideo({
          src: this.youtubeURL,
          width: 640,
          height: 480
        })
        this.youtubeURL = ''
      } else {
        this.showYouTubeDialog = false
        this.youtubeURL = ''
      }
    },
    async addImage(a) {
      if (a == 1) {
        this.showImageDialog = true
      } else if (a == 2) {
        this.editor.commands.setImage({
          src: this.imageURL,
          alt: 'Image'
        })
        this.showImageDialog = false
      } else if (a == 3) {
        //get image
        const image = document.getElementById('imageUpload').files[0]
        //check if under limit of 5MB
        if (image.size > 5242880) {
          alert('Image is too large. Please upload an image under 10MB.')
          this.showImageDialog = true
          return
        }
        //generate reference to storage
        const storage = getStorage()
        let key = this.generateKey()
        const imageRef = ref(storage, this.userId + '/images/' + key)
        //upload image to storage
        await uploadBytes(imageRef, image).then(() => {
          console.log('Uploaded a blob or file!')
        })
        //get download URL
        await getDownloadURL(imageRef)
          .then((url) => {
            this.downloadURL = url
          })
          .catch((error) => {
            alert('Error uploading document:', error)
          })
        //insert image
        this.editor.commands.setImage({
          src: this.downloadURL,
          alt: 'Image'
        })
        this.showImageDialog = false
      } else {
        this.showImageDialog = false
      }
    }
  },
  mounted() {
    this.editor = new Editor({
      editorProps: {
        attributes: {
          class:
            'h-full justify-between flex-grow mb-2.5 w-full border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50'
        }
      },
      extensions: [
        StarterKit,
        Highlight,
        CodeBlock,
        Underline,
        Typography,
        TextStyle,
        Color,
        TextAlign.configure({
          types: ['heading', 'paragraph']
        }),
        CharacterCount,
        Youtube.configure({
          nocookie: true,
          disableKBcontrols: true,
          modestBranding: true,
          progressBarColor: 'black'
        }),
        Image
      ],
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
    this.userId = localStorage.getItem('userId')
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

.prose {
  color: black;
}
</style>
