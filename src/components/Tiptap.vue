<template>
  <div class="flex flex-col h-full w-full box-border">
    <div
      class="h-[3.25rem] w-full justify-start p-0 m-0 border rounded-t-md border-b-0 box-border flex"
    >
      <div type="multiple" class="box-border p-1.5">
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5">
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer bg-black"
                :style="{ backgroundColor: editor.getAttributes('textStyle').color }"
                @click.prevent
              />
            </Button>
          </PopoverTrigger>
          <PopoverContent class="w-full">
            <!--Black-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#000000' }) }"
              @click="editor.chain().focus().setColor('#000000').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#000000' }"
              />
            </Button>
            <!--Red-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#fc0707' }) }"
              @click="editor.chain().focus().setColor('#fc0707').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#fc0707' }"
              />
            </Button>
            <!--Orange-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#fc8a07' }) }"
              @click="editor.chain().focus().setColor('#fc8a07').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#fc8a07' }"
              />
            </Button>
            <!--Yellow-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#fcf007' }) }"
              @click="editor.chain().focus().setColor('#fcf007').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#fcf007' }"
              />
            </Button>
            <!--Green-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#13fc07' }) }"
              @click="editor.chain().focus().setColor('#13fc07').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#13fc07' }"
              />
            </Button>
            <!--Blue-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#0e02f7' }) }"
              @click="editor.chain().focus().setColor('#0e02f7').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#0e02f7' }"
              />
            </Button>
            <!--Purple-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#a207fc' }) }"
              @click="editor.chain().focus().setColor('#a207fc').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#a207fc' }"
              />
            </Button>
            <!--Pink-->
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('textStyle', { color: '#f807fc' }) }"
              @click="editor.chain().focus().setColor('#f807fc').run()"
            >
              <div
                type="color"
                class="h-4 w-4 bg-transparent cursor-pointer"
                :style="{ backgroundColor: '#f807fc' }"
              />
            </Button>
          </PopoverContent>
        </Popover>
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5">
              <Type class="h-4 w-4"></Type>
            </Button>
          </PopoverTrigger>
          <PopoverContent class="w-50 p-2.5">
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleBold().run()"
              :disabled="!editor.can().chain().focus().toggleBold().run()"
              :class="{ activeToggle: editor.isActive('bold') }"
            >
              <Bold class="h-4 w-4"></Bold>
            </Button>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleItalic().run()"
              :disabled="!editor.can().chain().focus().toggleItalic().run()"
              :class="{ activeToggle: editor.isActive('italic') }"
            >
              <Italic class="h-4 w-4"></Italic>
            </Button>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleUnderline().run()"
              :disabled="!editor.can().chain().focus().toggleUnderline().run()"
              :class="{ activeToggle: editor.isActive('underline') }"
            >
              <UnderlineIcon class="h-4 w-4"></UnderlineIcon>
            </Button>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleStrike().run()"
              :disabled="!editor.can().chain().focus().toggleStrike().run()"
              :class="{ activeToggle: editor.isActive('strike') }"
            >
              <Strikethrough class="h-4 w-4"></Strikethrough>
            </Button>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleHighlight().run()"
              :disabled="!editor.can().chain().focus().toggleHighlight().run()"
              :class="{ activeToggle: editor.isActive('highlight') }"
            >
              <Highlighter class="h-4 w-4"></Highlighter>
            </Button>
            <Separator orientation="horizontal" class="my-2.5" />
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleBulletList().run()"
              :class="{ activeToggle: editor.isActive('bulletList') }"
            >
              <List class="h-4 w-4"></List>
            </Button>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleOrderedList().run()"
              :class="{ activeToggle: editor.isActive('orderedList') }"
            >
              <ListOrdered class="h-4 w-4"></ListOrdered>
            </Button>
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
            >
              <TextQuote class="h-4 w-4"></TextQuote>
            </Button>
          </PopoverContent>
        </Popover>
        <Popover>
          <PopoverTrigger>
            <Button v-if="editor.isActive('paragraph')" variant="ghost" class="h-10 mr-1.5">
              <Pilcrow class="h-4 w-4"></Pilcrow>
            </Button>
            <Button
              v-if="editor.isActive('heading', { level: 1 })"
              variant="ghost"
              class="h-10 mr-1.5"
            >
              <Heading1 class="h-4 w-4"></Heading1>
            </Button>
            <Button
              v-if="editor.isActive('heading', { level: 2 })"
              variant="ghost"
              class="h-10 mr-1.5"
            >
              <Heading2 class="h-4 w-4"></Heading2>
            </Button>
            <Button
              v-if="editor.isActive('heading', { level: 3 })"
              variant="ghost"
              class="h-10 mr-1.5"
            >
              <Heading3 class="h-4 w-4"></Heading3>
            </Button>
            <Button
              v-if="editor.isActive('heading', { level: 4 })"
              variant="ghost"
              class="h-10 mr-1.5"
            >
              <Heading4 class="h-4 w-4"></Heading4>
            </Button>
          </PopoverTrigger>
          <PopoverContent class="w-full">
            <Button
              v-if="!editor.isActive('paragraph')"
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().setParagraph().run()"
            >
              <Pilcrow class="h-4 w-4"></Pilcrow>
            </Button>
            <Button
              v-if="!editor.isActive('heading', { level: 1 })"
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleHeading({ level: 1 }).run()"
            >
              <Heading1 class="h-4 w-4"></Heading1>
            </Button>
            <Button
              v-if="!editor.isActive('heading', { level: 2 })"
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleHeading({ level: 2 }).run()"
            >
              <Heading2 class="h-4 w-4"></Heading2>
            </Button>
            <Button
              v-if="!editor.isActive('heading', { level: 3 })"
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleHeading({ level: 3 }).run()"
            >
              <Heading3 class="h-4 w-4"></Heading3>
            </Button>
            <Button
              v-if="!editor.isActive('heading', { level: 4 })"
              variant="ghost"
              class="h-10 mr-1.5"
              @click="editor.chain().focus().toggleHeading({ level: 4 }).run()"
            >
              <Heading4 class="h-4 w-4"></Heading4>
            </Button>
          </PopoverContent>
        </Popover>
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5">
              <AlignCenter
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
            >
              <AlignLeft class="w-4 h-4"></AlignLeft>
            </Button>
            <Button
              v-if="!editor.isActive({ textAlign: 'center' })"
              @click="editor.chain().focus().setTextAlign('center').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'center' }) }"
            >
              <AlignCenter class="w-4 h-4"></AlignCenter>
            </Button>
            <Button
              v-if="!editor.isActive({ textAlign: 'justify' })"
              @click="editor.chain().focus().setTextAlign('justify').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'justify' }) }"
            >
              <AlignJustify class="w-4 h-4"></AlignJustify>
            </Button>
            <Button
              v-if="!editor.isActive({ textAlign: 'right' })"
              @click="editor.chain().focus().setTextAlign('right').run()"
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive({ textAlign: 'right' }) }"
            >
              <AlignRight class="w-4 h-4"></AlignRight>
            </Button>
          </PopoverContent>
        </Popover>
        <Popover>
          <PopoverTrigger>
            <Button variant="ghost" class="h-10 mr-1.5">
              <File class="h-4 w-4"></File>
            </Button>
          </PopoverTrigger>
          <PopoverContent class="w-full">
            <Button variant="ghost" class="h-10 mr-1.5" @click="addYoutubeVideo(1)">
              <Film class="h-4 w-4"></Film>
            </Button>
            <AlertDialog :open="showYouTubeDialog">
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Add a YouTube Video</AlertDialogTitle>
                  <AlertDialogDescription>
                    Enter a URL to a YouTube video and select a size to embed it.
                  </AlertDialogDescription>
                  <Input
                    type="url"
                    placeholder="eg. 'https://www.youtube.com/watch?v=dQw4w9WgXcQ&pp=ygUXbmV2ZXIgZ29ubmEgZ2l2ZSB5b3UgdXA%3D'"
                    v-model="youtubeURL"
                  />
                  <Tabs v-model="youtubeVideoSize" default-value="medium" class="w-full">
                    <TabsList class="flex">
                      <TabsTrigger value="small" class="w-1/3"> Small</TabsTrigger>
                      <TabsTrigger value="medium" class="w-1/3"> Medium</TabsTrigger>
                      <TabsTrigger value="large" class="w-1/3"> Large</TabsTrigger>
                    </TabsList>
                  </Tabs>
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel @click="addYoutubeVideo(3)">Cancel</AlertDialogCancel>
                  <AlertDialogAction @click="addYoutubeVideo(2)">Add</AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
            <Button variant="ghost" class="h-10 mr-1.5" @click="addImage(1)">
              <ImageIcon class="h-4 w-4"></ImageIcon>
            </Button>
            <AlertDialog :open="showImageDialog">
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Add an Image</AlertDialogTitle>
                  <AlertDialogDescription>
                    Select the size of your image, then enter a URL to, or upload an image. Please
                    note that uploaded images have to be under 5MB. The images themselves are not
                    encrypted.
                  </AlertDialogDescription>
                  <div class="flex w-full items-center gap-1.5">
                    <Input type="url" placeholder="Enter a URL to an image" v-model="imageURL" />
                    <Button @click="addImage(2)" type="submit"> Add</Button>
                  </div>
                  <div class="flex w-full items-center gap-1.5">
                    <Input id="imageUpload" type="file" accept="image/*" />
                    <Button @click="addImage(3)" type="submit"> Upload</Button>
                  </div>
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel @click="addImage(4)">Cancel</AlertDialogCancel>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
            <Button
              variant="ghost"
              class="h-10 mr-1.5"
              :class="{ activeToggle: editor.isActive('link') }"
              @click="setLink(1)"
            >
              <LinkIcon class="h-4 w-4"></LinkIcon>
            </Button>
            <AlertDialog :open="showLinkDialog">
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Add or edit a Link</AlertDialogTitle>
                  <AlertDialogDescription>
                    Enter a URL to link the text to.
                  </AlertDialogDescription>
                  <Input type="url" placeholder="e.g. https://apple.com" v-model="url" />
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel v-if="editor.isActive('link')" @click="setLink(4)"
                    >Remove Link
                  </AlertDialogCancel>
                  <AlertDialogCancel @click="setLink(3)">Cancel</AlertDialogCancel>
                  <AlertDialogAction @click="setLink(2)">Save</AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
          </PopoverContent>
        </Popover>
        <Button variant="ghost" class="h-10 mr-1.5" @click="share(1)">
          <Share class="h-4 w-4"></Share>
        </Button>
        <AlertDialog :open="showShareDialog">
          <AlertDialogContent>
            <AlertDialogHeader>
              <AlertDialogTitle>Share "{{ activeDocument }}"</AlertDialogTitle>
              <div v-if="shareURL == null">
                <AlertDialogDescription>
                  This will generate a link that you can share with others. Anyone with the link can
                  access and copy the note, thus an unencrypted version of it will be stored on our
                  servers. The unencrypted version cannot be linked back to your account. Changes
                  you make to your note will not be reflected in the shared version.
                </AlertDialogDescription>
                <FormField name="password">
                  <FormItem
                    class="flex flex-row items-center justify-between rounded-md border p-2.5 mt-2.5 h-16"
                  >
                    <div v-if="!protectSharedNoteWithPassword" class="space-y-0.5">
                      <FormLabel class="text-base"> Password protection</FormLabel>
                      <FormDescription> Protect your shared note with a password.</FormDescription>
                    </div>
                    <Input
                      v-else
                      type="password"
                      placeholder="Your super secure password"
                      class="w-80 ml-px"
                      v-model="sharedNotePassword"
                    />
                    <FormControl>
                      <Switch
                        :checked="protectSharedNoteWithPassword"
                        @update:checked="changeProtectSharedNoteWithPassword"
                      />
                    </FormControl>
                  </FormItem>
                </FormField>
              </div>
              <div v-else>
                <AlertDialogDescription> This is the link:</AlertDialogDescription>
                <div class="flex w-full items-center gap-1.5">
                  <Input type="url" v-model="shareURL" style="cursor: text" disabled />
                  <Button variant="outline" @click="copyToClipboard(shareURL)">
                    <Copy class="h-4 w-4"></Copy>
                  </Button>
                </div>
              </div>
            </AlertDialogHeader>
            <AlertDialogFooter>
              <div v-if="shareURL == null">
                <Button class="mr-2" variant="outline" @click="share(3)">Cancel</Button>
                <Button @click="share(2)">Share</Button>
              </div>
              <div v-else>
                <Button variant="outline" @click="share(3)">Close</Button>
              </div>
            </AlertDialogFooter>
          </AlertDialogContent>
        </AlertDialog>
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
      >
        <Bold class="h-4 w-4"></Bold>
      </Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleItalic().run()"
        :disabled="!editor.can().chain().focus().toggleItalic().run()"
        :class="{ activeToggle: editor.isActive('italic') }"
      >
        <Italic class="h-4 w-4"></Italic>
      </Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleUnderline().run()"
        :disabled="!editor.can().chain().focus().toggleUnderline().run()"
        :class="{ activeToggle: editor.isActive('underline') }"
      >
        <UnderlineIcon class="h-4 w-4"></UnderlineIcon>
      </Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleStrike().run()"
        :disabled="!editor.can().chain().focus().toggleStrike().run()"
        :class="{ activeToggle: editor.isActive('strike') }"
      >
        <Strikethrough class="h-4 w-4"></Strikethrough>
      </Button>
      <Button
        variant="ghost"
        class="h-10 rounded-none"
        @click="editor.chain().focus().toggleHighlight().run()"
        :disabled="!editor.can().chain().focus().toggleHighlight().run()"
        :class="{ activeToggle: editor.isActive('highlight') }"
      >
        <Highlighter class="h-4 w-4"></Highlighter>
      </Button>
    </bubble-menu>
    <editor-content
      class="w-full prose md:prose-lg lg:prose-xl max-w-none overflow-y-scroll"
      style="height: calc(100vh - 72px)"
      :editor="editor"
    />
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
import Document from '@tiptap/extension-document'
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
import Dropcursor from '@tiptap/extension-dropcursor'
import Placeholder from '@tiptap/extension-placeholder'
import Link from '@tiptap/extension-link'

const CustomDocument = Document.extend({
  content: 'heading block*'
})
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
  Heading4,
  List,
  ListOrdered,
  Highlighter,
  Underline as UnderlineIcon,
  Pilcrow,
  File,
  TextQuote,
  AlignCenter,
  AlignJustify,
  AlignLeft,
  AlignRight,
  Film,
  Image as ImageIcon,
  Link2 as LinkIcon,
  Link2Off as LinkIconOff,
  Share,
  Copy
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
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'
import { FormControl, FormDescription, FormField, FormItem, FormLabel } from '@/components/ui/form'
import { Switch } from '@/components/ui/switch'
//Firebase imports
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, setDoc } from 'firebase/firestore'
//crypto js
import CryptoJS from 'crypto-js'

export default {
  components: {
    EditorContent,
    // eslint-disable-next-line vue/no-reserved-component-names
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
    Heading4,
    List,
    ListOrdered,
    Highlight,
    Highlighter,
    UnderlineIcon,
    Popover,
    PopoverContent,
    PopoverTrigger,
    Pilcrow,
    File,
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
    ImageIcon,
    Tabs,
    TabsContent,
    TabsList,
    TabsTrigger,
    LinkIcon,
    LinkIconOff,
    Share,
    Copy,
    FormControl,
    FormDescription,
    FormField,
    FormItem,
    FormLabel,
    Switch
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
      youtubeVideoSize: 'medium',
      showImageDialog: false,
      imageURL: '',
      downloadURL: '',
      showLinkDialog: false,
      url: '',
      showShareDialog: false,
      shareURL: null,
      protectSharedNoteWithPassword: false,
      sharedNotePassword: '',
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
  computed: {
    activeDocument() {
      return localStorage.getItem('activeDocument')
    }
  },
  methods: {
    generateKey() {
      //get variables
      let charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+*&=!-_.:,;'
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
      if (a === 1) {
        this.showYouTubeDialog = true
      } else if (a === 2) {
        this.showYouTubeDialog = false
        let width = 640,
          height = 480
        if (this.youtubeVideoSize === 'small') {
          width = 320
          height = 240
        } else if (this.youtubeVideoSize === 'large') {
          width = 860
          height = 720
        }
        this.editor.commands.setYoutubeVideo({
          src: this.youtubeURL,
          width: width,
          height: height
        })
        this.youtubeURL = ''
      } else {
        this.showYouTubeDialog = false
        this.youtubeURL = ''
      }
    },
    async addImage(a) {
      if (a === 1) {
        this.showImageDialog = true
      } else if (a === 2) {
        this.editor.commands.setImage({
          src: this.imageURL,
          alt: 'Image'
        })
        this.showImageDialog = false
      } else if (a === 3) {
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
            alert('Error uploading document: ' + error)
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
    },
    setLink(a) {
      if (a === 1) {
        this.url = this.editor.getAttributes('link').href
        this.showLinkDialog = true
      } else if (a === 2) {
        this.showLinkDialog = false
        // cancelled
        if (this.url === null) {
          return
        }
        // empty
        if (this.url === '') {
          this.editor.chain().focus().extendMarkRange('link').unsetLink().run()
          return
        }
        // update link
        this.editor.chain().focus().extendMarkRange('link').setLink({ href: this.url }).run()
        this.url = ''
      } else if (a === 3) {
        this.showLinkDialog = false
        this.url = ''
      } else {
        this.editor.chain().focus().unsetLink().run()
        this.showLinkDialog = false
      }
    },
    share(a) {
      if (a === 1) {
        this.showShareDialog = true
      } else if (a === 2) {
        let key = this.generateKey()
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, 'shared', key)
        //save document to firebase
        if (this.protectSharedNoteWithPassword) {
          const salt = CryptoJS.lib.WordArray.random(128 / 8) //Generate Salt
          // Concatenate password and salt
          const saltedPassword = this.sharedNotePassword + salt
          // Hash the salted password
          const hash = CryptoJS.SHA256(saltedPassword)
          // Convert the hash to a string to store it easily
          const hashHEX = hash.toString(CryptoJS.enc.Hex)
          //set to firebase
          setDoc(docRef, {
            note: this.editor.getHTML(),
            title: localStorage.getItem('activeDocument'),
            password: hashHEX,
            salt: salt.toString()
          })
          this.sharedNotePassword = ''
          this.protectSharedNoteWithPassword = false
        } else {
          setDoc(docRef, {
            note: this.editor.getHTML(),
            title: localStorage.getItem('activeDocument')
          })
        }
        this.shareURL = window.location.protocol + '//' + window.location.host + '/shared/' + key
      } else {
        this.showShareDialog = false
        this.shareURL = null
      }
    },
    copyToClipboard(text) {
      navigator.clipboard.writeText(text).then
    },
    changeProtectSharedNoteWithPassword() {
      if (!this.protectSharedNoteWithPassword) {
        this.protectSharedNoteWithPassword = true
      } else if (this.protectSharedNoteWithPassword) {
        this.protectSharedNoteWithPassword = false
      }
    }
  },
  mounted() {
    this.editor = new Editor({
      editorProps: {
        attributes: {
          class:
            'h-full overflow-y-scroll justify-between mb-2.5 w-full border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50'
        }
      },
      extensions: [
        CustomDocument,
        StarterKit.configure({
          document: false
        }),
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
        Image,
        Dropcursor,
        Placeholder.configure({
          placeholder: ({ node }) => {
            if (node.type.name === 'heading') {
              return 'What’s the title?'
            }

            return "C'mon, say some more!"
          }
        }),
        Link.configure({
          HTMLAttributes: {
            class: 'tiptapLink'
          }
        })
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
<style>
.activeToggle {
  background-color: #f4f4f5;
}

.tiptap p.is-editor-empty:first-child::before {
  content: attr(data-placeholder);
  float: left;
  color: #7f7f7f;
  pointer-events: none;
  height: 0;
}

.tiptapLink {
  cursor: pointer;
}

.tiptap .is-empty::before {
  content: attr(data-placeholder);
  float: left;
  color: #ced4da;
  pointer-events: none;
  height: 0;
}

.prose {
  --tw-prose-body: #000000;
  --tw-prose-headings: #000000;
}
</style>
