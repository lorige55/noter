<!-- eslint-disable no-undef -->
<script>
import CryptoJS from 'crypto-js'
import '@tailwindcss/typography'
import '@passageidentity/passage-elements/passage-auth'
import { PassageUser } from '@passageidentity/passage-auth/passage-user'
//Firebase Imports
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, getDoc, setDoc, deleteDoc } from 'firebase/firestore'
//shadcn Imports:
import { Button } from '@/components/ui/button'
import {
  Menubar,
  MenubarContent,
  MenubarItem,
  MenubarMenu,
  MenubarSeparator,
  MenubarShortcut,
  MenubarSub,
  MenubarSubContent,
  MenubarSubTrigger,
  MenubarTrigger
} from '@/components/ui/menubar'
import { Input } from '@/components/ui/input'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle
} from '@/components/ui/card'
import { Switch } from '@/components/ui/switch'
import {
  AlertDialog,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle
} from '@/components/ui/alert-dialog'
import { Badge } from '@/components/ui/badge'
import { ResizableHandle, ResizablePanel, ResizablePanelGroup } from '@/components/ui/resizable'
import { HoverCard, HoverCardContent, HoverCardTrigger } from '@/components/ui/hover-card'
import { Label } from '@/components/ui/label'

//Tiptap Import
import Tiptap from './components/Tiptap.vue'

//icon imports
import {
  HeartCrack,
  AlertCircle,
  User,
  Lock,
  Github,
  LifeBuoy,
  LogOut,
  X,
  Info,
  File,
  Trash2,
  Sticker,
  Frown
} from 'lucide-vue-next'

export default {
  components: {
    // eslint-disable-next-line vue/no-reserved-component-names
    Label,
    Tiptap,
    // eslint-disable-next-line vue/no-reserved-component-names
    Button,
    Menubar,
    MenubarContent,
    MenubarItem,
    MenubarMenu,
    MenubarSeparator,
    MenubarShortcut,
    MenubarSub,
    MenubarSubContent,
    MenubarSubTrigger,
    MenubarTrigger,
    // eslint-disable-next-line vue/no-reserved-component-names
    Input,
    AlertCircle,
    Alert,
    AlertDescription,
    AlertTitle,
    User,
    Lock,
    Github,
    LifeBuoy,
    LogOut,
    Tabs,
    TabsContent,
    TabsList,
    TabsTrigger,
    Card,
    CardContent,
    CardDescription,
    CardFooter,
    CardHeader,
    CardTitle,
    X,
    // eslint-disable-next-line vue/no-reserved-component-names
    Switch,
    AlertDialog,
    AlertDialogContent,
    AlertDialogDescription,
    AlertDialogFooter,
    AlertDialogHeader,
    AlertDialogTitle,
    Badge,
    ResizableHandle,
    ResizablePanel,
    ResizablePanelGroup,
    Info,
    HoverCard,
    HoverCardContent,
    HoverCardTrigger,
    File,
    Trash2,
    Sticker,
    Frown,
    HeartCrack
  },
  watch: {
    activeDocument(newActiveDocument) {
      document.title = 'Noter - ' + newActiveDocument
      localStorage.setItem('activeDocument', newActiveDocument)
    },
    activeDocumentContent() {
      let newActiveDocument = this.activeDocumentContent.split('<h1>')[1].split('</h1>')[0]
      if (newActiveDocument.includes('<span')) {
        newActiveDocument = newActiveDocument.split('">')[1].split('</span')[0]
      }
      this.activeDocument = newActiveDocument
      this.autoSave()
    }
  },
  data() {
    return {
      shared: false,
      sharedPasswordProtected: false,
      sharedKey: '',
      sharedPasswordHash: '',
      sharedPasswordSalt: '',
      sharedNoteInputPassword: '',
      sharedTitle: '',
      sharedContent: '',
      isLoggedIn: false,
      user: null,
      userId: null,
      appId: null,
      index: [],
      keyIndex: [],
      activeDocumentContent: '',
      activeDocument: '',
      activeDocumentIndex: null,
      firebaseConfig: {
        apiKey: 'AIzaSyBV9FOnKKOBNLuQsCn9T4OdfxT39cRhF6g',
        authDomain: 'noter-6e08f.firebaseapp.com',
        projectId: 'noter-6e08f',
        storageBucket: 'noter-6e08f.appspot.com',
        messagingSenderId: '967538971502',
        appId: '1:967538971502:web:6d31288c8ade545465277f'
      },
      activeSavingProcesses: 0,
      secretKey: '',
      trust: true,
      documentToRename: '',
      displayError: false,
      errorMessage: 'An Error has occurred. Please try again or submit an Issue.',
      ready: false,
      progress: 'Ready, set go!',
      showSettings: false,
      tabToOpen: 'account',
      showNewSecretKeyConformation: false,
      showChangeSecretKeyConformation: false,
      customSecretKeyInput: null,
      showAccountDataDeletionConformation: false,
      showImportConformation: false,
      showNoteDeleteConformation: false,
      noteToDelete: ''
    }
  },
  methods: {
    async enterSharedNoteWithPassword() {
      // Concatenate password and salt
      const saltedPassword = this.sharedNoteInputPassword + this.sharedPasswordSalt
      // Hash the salted password
      const hash = CryptoJS.SHA256(saltedPassword)
      // Convert the hash to a string to store it easily
      const hashHEX = hash.toString(CryptoJS.enc.Hex)
      if (hashHEX === this.sharedPasswordHash) {
        //initialize firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, 'shared', this.sharedKey)
        let docSnap = await getDoc(docRef)
        this.sharedTitle = docSnap.data().title
        this.sharedContent = docSnap.data().note
        document.title = 'Noter - ' + this.sharedTitle
      } else {
        alert('Nice try! Wrong password.')
      }
    },
    isMobileDevice() {
      const userAgent = navigator.userAgent || navigator.vendor || window.opera
      // Check against all known Mobile Device User Agent substrings
      if (/windows phone/i.test(userAgent)) {
        return true
      }
      if (/android/i.test(userAgent)) {
        return true
      }
      // iOS detection from: http://stackoverflow.com/a/9039885/177710
      if (/iPad|iPhone|iPod/.test(userAgent) && !window.MSStream) {
        return true
      }
      // Default to 'not a Mobile Device'
      return false
    },
    async getDocument(item) {
      if (this.activeDocument !== '') {
        await this.saveActiveDocument()
      }
      //set activeDocument
      this.activeDocument = item
      //set active document index
      this.activeDocumentIndex = this.index.indexOf(item)
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      //get document from firebase and set activeDocumentContent to content
      let docRef = doc(db, this.userId, this.keyIndex[this.activeDocumentIndex])
      let docSnap = await getDoc(docRef)
      if (docSnap.exists()) {
        this.activeDocumentContent = this.decryptString(docSnap.data().note)
      } else {
        console.error('A note could not be found. Please try again or create an Issue on GitHub.')
      }
      localStorage.setItem('lastNote', this.activeDocument)
    },
    async saveActiveDocument() {
      //input validation
      if (
        this.index.includes(this.activeDocument) &&
        this.index[this.activeDocumentIndex] !== this.activeDocument
      ) {
        this.errorMessage =
          'You cannot name a note the same as another note. Please rename the note. It will not be saved under this title. Click this message to make it disappear.'
        this.displayError = true
        return
      }
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.userId, this.keyIndex[this.activeDocumentIndex])
      //save document to firebase
      await setDoc(docRef, {
        note: this.encryptString(this.activeDocumentContent),
        title: this.encryptString(this.activeDocument)
      })
      localStorage.setItem('lastNote', this.activeDocument)
      this.index[this.activeDocumentIndex] = this.activeDocument
    },
    autoSave() {
      //save active document after 3 seconds of inactivity
      this.activeSavingProcesses++
      setTimeout(() => {
        if (this.activeSavingProcesses === 1) {
          this.saveActiveDocument()
        }
        this.activeSavingProcesses--
      }, 2500)
    },
    async deleteDocument(item) {
      //safe active document before proceeding
      if (this.activeDocument !== '') {
        await this.saveActiveDocument()
      }
      //initialize firebase and the index of the item to remove
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const indexToRemove = this.index.indexOf(item)
      //make sure if the item to remove is the last note to remain
      if (this.index.length === 1) {
        this.activeDocument = ''
        this.activeDocumentContent = null
        this.activeDocumentIndex = null
      }
      //remove item from lastNote in localStorage if it is the lastNote
      if (localStorage.getItem('lastNote') === this.index[item]) {
        localStorage.removeItem('lastNote')
      }
      //if indexToRemove is valid
      if (indexToRemove !== -1) {
        //remove item from index
        this.index.splice(indexToRemove, 1)
        //delete document from firebase
        await deleteDoc(doc(db, this.userId, this.keyIndex[indexToRemove]))
        //remove key from keyIndex and push new keyIndex to firebase
        this.keyIndex.splice(indexToRemove, 1)
        let docRef = doc(db, this.userId, 'keyIndex')
        await setDoc(docRef, { index: this.keyIndex })
        //set activeDocument Variables
        this.activeDocument = ''
        this.activeDocumentContent = ''
        this.activeDocumentIndex = null
        //hide dialog
        this.showNoteDeleteConformation = false
      } else {
        console.log('An Error has occurred. Please try again or create an Issue on GitHub.')
        this.errorMessage = 'An Error has occurred. Please try again or submit an Issue.'
        this.displayError = true
      }
    },
    async createNewDocument() {
      if (this.index.length !== 0) {
        await this.saveActiveDocument()
      }
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      //find name
      let newNoteName = 'New Note'
      let nameFound = false
      for (let i = 0; i < this.index.length && nameFound === false; i++) {
        if (this.index.includes(newNoteName)) {
          newNoteName = 'New Note ' + (i + 1)
        } else {
          nameFound = true
        }
      }
      //push new Note to Index
      this.index.unshift(newNoteName)
      //generate new key and push it to keyIndex
      let key = await this.generateKey(128)
      this.keyIndex.unshift(key)
      //push new keyIndex to firebase
      let docRef = doc(db, this.userId, 'keyIndex')
      await setDoc(docRef, { index: this.keyIndex })
      //push new note to firebase
      docRef = doc(db, this.userId, key)
      await setDoc(docRef, {
        note: this.encryptString(''),
        title: this.encryptString(newNoteName)
      })
      //set activeDocument and lastNote to new note
      this.activeDocumentContent = ''
      this.activeDocument = newNoteName
      this.activeDocumentIndex = this.index.indexOf(newNoteName)
      localStorage.setItem('lastNote', this.activeDocument)
    },
    encryptString(string) {
      //verify that input is a string
      string.toString()
      //encrypt with AES-256
      let encrypted = CryptoJS.Rabbit.encrypt(string, this.secretKey)
      //encode to custom charset base64
      let encoded = btoa(encrypted)
      return encoded.replace(/\//g, '_').replace(/=/g, '-')
    },
    decryptString(string) {
      //verify that input is a string
      string.toString()
      //decode from base64
      string = string.replace(/_/g, '/').replace(/-/g, '=')
      string = atob(string)
      //decrypt with AES-256
      let decrypted = CryptoJS.Rabbit.decrypt(string, this.secretKey)
      return decrypted.toString(CryptoJS.enc.Utf8)
    },
    async generateKey(length) {
      //generate a random key
      let key = CryptoJS.lib.WordArray.random(length / 8)
      key.toString(CryptoJS.enc.Base64)
      //encode to base64 and return
      return btoa(key)
    },
    generateNewSecretKey() {
      //generate new key and set it to secretKey
      this.secretKey = this.generateKey(256)
      //push new key to firebase if trust is true
      if (this.trust) {
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        setDoc(docRef, { key: this.secretKey })
      }
      //save new key to localStorage
      localStorage.setItem('secretKey', this.secretKey)
    },
    saveNewSecretKey() {
      //save new secret key
      this.secretKey = this.customSecretKeyInput
      localStorage.setItem('secretKey', this.secretKey)
      //close dialog
      this.showChangeSecretKeyConformation = false
    },
    async trustChanged() {
      if (!this.trust) {
        this.trust = true
        //if trust changed to true, push secretKey to firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        await setDoc(docRef, { key: this.secretKey })
      } else if (this.trust) {
        this.trust = false
        //if trust changed to false remove secretKey from firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        await setDoc(docRef, { key: 'nah' })
        localStorage.setItem('secretKey', this.secretKey)
      }
    },
    async logout() {
      localStorage.removeItem('psg_auth_token')
      this.reload()
    },
    async deleteAccountData() {
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      //delete all documents from firebase
      for (let i = 0; i < this.keyIndex.length; i++) {
        await deleteDoc(doc(db, this.userId, this.keyIndex[i]))
      }
      //delete keyIndex from firebase
      await deleteDoc(doc(db, this.userId, 'keyIndex'))
      //delete secretKey from firebase
      await deleteDoc(doc(db, this.userId, 'secretKey'))
      this.reload()
    },
    async exportData() {
      // Initialize Firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)

      //create base dataToExport
      let dataToExport = {
        keyIndex: this.keyIndex
      }
      //create rest of the dataToExport json
      for (let i = 0; i < this.keyIndex.length; i++) {
        let docRef = doc(db, this.userId, this.keyIndex[i])
        let docSnap = await getDoc(docRef)
        dataToExport[this.keyIndex[i]] = { note: docSnap.data().note, title: docSnap.data().title }
      }

      //download data as json
      // Convert the data to a JSON string
      const jsonString = JSON.stringify(dataToExport, null, 2)
      // Create a Blob object
      const blob = new Blob([jsonString], { type: 'application/json' })
      // Create a link element
      const a = document.createElement('a')
      a.href = window.URL.createObjectURL(blob)
      a.download = 'noter_data.json'
      // Append the link to the body
      document.body.appendChild(a)
      // Trigger the click event of the link
      a.click()
      // Remove the link from the body
      document.body.removeChild(a)
    },
    async importData() {
      const fileInput = document.getElementById('fileUpload')
      const file = fileInput.files[0]

      if (!file) {
        alert('Please select a file.')
        return
      }

      const reader = new FileReader()
      reader.onload = async (event) => {
        const jsonContent = event.target.result
        try {
          const parsedData = JSON.parse(jsonContent)

          // Initialize Firebase
          const app = initializeApp(this.firebaseConfig)
          const db = getFirestore(app)

          //delete all documents from firebase
          for (let i = 0; i < this.keyIndex.length; i++) {
            await deleteDoc(doc(db, this.userId, this.keyIndex[i]))
          }
          //delete keyIndex from firebase
          await deleteDoc(doc(db, this.userId, 'keyIndex'))

          //keyIndex
          let docRef = doc(db, this.userId, 'keyIndex')
          await setDoc(docRef, {
            index: parsedData.keyIndex
          })
          //notes
          for (let i = 0; i < parsedData.keyIndex.length; i++) {
            let key = parsedData.keyIndex[i]
            docRef = doc(db, this.userId, key)
            await setDoc(docRef, {
              note: parsedData[key].note,
              title: parsedData[key].title
            })
          }
          this.reload()
        } catch (error) {
          console.error('Error parsing JSON:', error)
        }
      }
      reader.readAsText(file)
      this.showImportConformation = false
    },
    fullscreen() {
      //check if fullscreen is active, if so exit, else enter
      if (
        document.fullscreenElement ||
        document.webkitFullscreenElement ||
        document.mozFullScreenElement ||
        document.msFullscreenElement
      ) {
        if (document.exitFullscreen) {
          document.exitFullscreen()
        } else if (document.mozCancelFullScreen) {
          // Firefox
          document.mozCancelFullScreen()
        } else if (document.webkitExitFullscreen) {
          // Chrome, Safari and Opera
          document.webkitExitFullscreen()
        } else if (document.msExitFullscreen) {
          // IE/Edge
          document.msExitFullscreen()
        }
      } else {
        if (document.documentElement.requestFullscreen) {
          document.documentElement.requestFullscreen()
        } else if (document.documentElement.mozRequestFullScreen) {
          // Firefox
          document.documentElement.mozRequestFullScreen()
        } else if (document.documentElement.webkitRequestFullscreen) {
          // Chrome, Safari and Opera
          document.documentElement.webkitRequestFullscreen()
        } else if (document.documentElement.msRequestFullscreen) {
          // IE/Edge
          document.documentElement.msRequestFullscreen()
        }
      }
    },
    reload() {
      location.reload()
    },
    cloneAndEdit() {
      let key = window.location.pathname.split('/')[2]
      window.open('localhost:4173/clone/' + key + '/')
    }
  },
  async mounted() {
    if (window.location.pathname.includes('shared/')) {
      this.sharedKey = window.location.pathname.split('/')[2]
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, 'shared', this.sharedKey)
      let docSnap = await getDoc(docRef)
      if (docSnap.exists()) {
        this.shared = true
        let data = docSnap.data()
        if (data.password) {
          this.sharedPasswordProtected = true
          this.sharedPasswordHash = data.password
          this.sharedPasswordSalt = data.salt
          document.title = 'Noter - Enter Password'
        } else {
          this.sharedContent = data.note
          this.sharedTitle = data.title
          document.title = 'Noter - ' + this.sharedTitle
        }
      } else {
        alert('This shared note does not exist')
      }
    } else {
      if (window.location.hostname === 'noter.yeomid.com') {
        //setting appId based on location
        this.appId = 'LH8ZzpbwJuHH6xGFk6GgmtSC'
      } else {
        this.appId = 'JlXUGO3ZcoTO3pK2BSb38cc2'
      }
      //try to authenticate
      try {
        const user = new PassageUser()
        const userInfo = await user.userInfo()
        if (userInfo === undefined) {
          this.isLoggedIn = false
          return
        }
        this.isLoggedIn = true
        this.user = userInfo
        this.userId = userInfo.id
        localStorage.setItem('userId', this.userId)
      } catch (error) {
        console.log(error)
        this.errorMessage = 'An Error has occurred. Please try again or submit an Issue.'
        this.displayError = true
      }
      this.progress = 'Initializing secret key...'
      //get secret Key from firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.userId, 'secretKey')
      let docSnap = await getDoc(docRef)
      //if trust is true, get secretKey from firebase, else get it from localStorage or open profileModal if it is not set in localStorage
      if (docSnap.exists()) {
        if (docSnap.data().key !== 'nah') {
          this.secretKey = docSnap.data().key
        } else {
          this.trust = false
          if (localStorage.getItem('secretKey') !== null) {
            this.secretKey = localStorage.getItem('secretKey')
          } else {
            this.showChangeSecretKeyConformation = true
          }
        }
      } else {
        //if there is no secretKey in firebase, generate a new one and push it to firebase
        let key = await this.generateKey(256)
        docRef = doc(db, this.userId, 'secretKey')
        await setDoc(docRef, { key: key })
        this.secretKey = key
      }
      this.progress = 'Syncing your notes...'
      //get keyIndex from firebase
      if (this.secretKey !== '') {
        docRef = doc(db, this.userId, 'keyIndex')
        docSnap = await getDoc(docRef)
        //if keyIndex exists, get it
        if (docSnap.exists()) {
          this.keyIndex = docSnap.data().index
          //loop through keyIndex and build index
          if (this.keyIndex.length > 0) {
            for (let i = 0; i < this.keyIndex.length; i++) {
              let docRef = doc(db, this.userId, this.keyIndex[i])
              let docSnap = await getDoc(docRef)
              try {
                this.index[i] = this.decryptString(docSnap.data().title)
              } catch {
                console.error("couldn't get document, skipped")
              }
            }
            this.progress = 'Loading your last note...'
            //recover last note or select first in array if equal to null
            if (localStorage.getItem('lastNote') !== 'undefined') {
              await this.getDocument(localStorage.getItem('lastNote'))
            }
          }
        } else {
          //if keyIndex does not exist, create a new one and push it to firebase
          let key = await this.generateKey(128)
          this.keyIndex = [key]
          await setDoc(docRef, { index: this.keyIndex })
          //set to no note
          this.index = []
          this.activeDocument = ''
          this.activeDocumentContent = ''
          this.activeDocumentIndex = null
        }
      }
      if (window.location.pathname.includes('clone/')) {
        this.progress = 'Cloning note...'
        let key = window.location.pathname.split('/')[2]
        let docRef = doc(db, 'shared', key)
        let docSnap = await getDoc(docRef)
        if (docSnap.exists()) {
          let data = docSnap.data()
          //push new Note to Index
          this.index.unshift(data.title)
          //generate new key and push it to keyIndex
          let key = await this.generateKey(128)
          this.keyIndex.unshift(key)
          //push new keyIndex to firebase
          let docRef = doc(db, this.userId, 'keyIndex')
          await setDoc(docRef, { index: this.keyIndex })
          //push new note to firebase
          docRef = doc(db, this.userId, key)
          await setDoc(docRef, {
            note: this.encryptString(data.note),
            title: this.encryptString(data.title + ' Copy')
          })
          //getDocument
          await this.getDocument(data.title)
        } else {
          alert("Couldn't clone note.")
        }
      }
      this.progress = 'Done!'
      this.ready = true
    }
  }
}
</script>

<template>
  <div v-if="this.shared === true" class="flex flex-col h-screen">
    <div v-if="this.sharedContent !== ''" class="h-screen">
      <div
        class="relative mx-2.5 my-2.5 border rounded-md overflow-y-scroll"
        style="height: calc(100vh - 20px)"
      >
        <div class="prose p-2.5 max-w-none" v-html="this.sharedContent"></div>
      </div>
      <div class="absolute top-0 right-0">
        <Button class="m-5 shadow" variant="outline" @click="cloneAndEdit()">Clone & Edit</Button>
      </div>
    </div>
    <div v-else>
      <div class="flex items-center justify-center h-screen w-screen">
        <div class="flex w-full max-w-sm items-center gap-1.5">
          <Input
            type="password"
            placeholder="Enter the password"
            v-model="sharedNoteInputPassword"
          />
          <Button type="submit" @click="enterSharedNoteWithPassword">Enter</Button>
        </div>
      </div>
    </div>
  </div>
  <div v-else-if="isMobileDevice() === true">
    <div
      class="absolute inset-0 mx-2.5 my-2.5 border rounded-md flex items-center justify-center flex-col"
    >
      <HeartCrack></HeartCrack>
      <p class="text text-sm">Noter is sadly not available on mobile (yet).</p>
    </div>
  </div>
  <div v-else>
    <div v-if="!isLoggedIn">
      <div class="flex justify-center items-center h-screen">
        <div class="authContainer">
          <passage-auth :app-id="appId"></passage-auth>
        </div>
      </div>
    </div>
    <div v-else>
      <div v-if="!ready">
        <div class="flex flex-col justify-center items-center h-screen">
          <div
            class="inline-block h-8 w-8 animate-spin rounded-full border-4 border-solid border-current border-e-transparent align-[-0.125em] text-surface motion-reduce:animate-[spin_1.5s_linear_infinite] mb-3 dark:text-white"
            role="status"
          ></div>
          <p class="text-xl w-1/3 text-center">{{ progress }}</p>
          <HoverCard class="w-96">
            <HoverCardTrigger>
              <Button variant="outline" class="mt-3">
                <Info></Info>
              </Button>
            </HoverCardTrigger>
            <HoverCardContent>
              <p class="text-sm text-center">
                This should only take a few seconds. If it takes too long, a slow internet
                connection or corrupt data could be the cause. If you think your data is corrupt,
                consider resetting it. If you have a backup of your data, you can import it after
                resetting.
              </p>
              <Button variant="destructive" @click="deleteAccountData()" class="w-full mt-3"
                >Delete My Data
              </Button>
            </HoverCardContent>
          </HoverCard>
        </div>
      </div>
      <div v-else>
        <div v-if="!showSettings">
          <div class="relative flex flex-col h-screen w-screen">
            <!--Menubar-->
            <Menubar class="h-11 mx-2.5 mt-2.5">
              <MenubarMenu>
                <MenubarTrigger>Home</MenubarTrigger>
                <MenubarContent>
                  <MenubarItem @click="(tabToOpen = 'account'), (showSettings = true)">
                    <User class="mr-2 h-4 w-4" />
                    Account
                  </MenubarItem>
                  <MenubarItem @click="(tabToOpen = 'privacyAndSecurity'), (showSettings = true)">
                    <Lock class="mr-2 h-4 w-4" />
                    Privacy & Security
                  </MenubarItem>
                  <MenubarSeparator />
                  <a href="https://github.com/lorige55/noter" target="_blank">
                    <MenubarItem>
                      <Github class="mr-2 h-4 w-4" />
                      GitHub
                    </MenubarItem>
                  </a>
                  <a href="https://github.com/lorige55/noter/issues" target="_blank">
                    <MenubarItem>
                      <LifeBuoy class="mr-2 h-4 w-4" />
                      Support
                    </MenubarItem>
                  </a>
                  <MenubarSeparator />
                  <MenubarItem @click="logout()">
                    <LogOut class="mr-2 h-4 w-4" />
                    Logout
                  </MenubarItem>
                </MenubarContent>
              </MenubarMenu>
              <MenubarMenu>
                <MenubarTrigger>File</MenubarTrigger>
                <MenubarContent>
                  <MenubarItem @click="createNewDocument()"> New File</MenubarItem>
                  <MenubarItem disabled> New Folder</MenubarItem>
                  <MenubarSeparator />
                  <MenubarItem
                    @click="(showNoteDeleteConformation = true), (noteToDelete = activeDocument)"
                  >
                    Delete
                  </MenubarItem>
                  <MenubarSeparator />
                  <MenubarSub>
                    <MenubarSubTrigger>Share</MenubarSubTrigger>
                    <MenubarSubContent>
                      <MenubarItem disabled>Link</MenubarItem>
                      <MenubarItem disabled>Email</MenubarItem>
                      <MenubarItem disabled>SMS</MenubarItem>
                    </MenubarSubContent>
                  </MenubarSub>
                  <MenubarSeparator />
                  <MenubarItem @click="(tabToOpen = 'account'), (showSettings = true)"
                    >Import Data
                  </MenubarItem>
                  <MenubarItem @click="exportData()">Export Data</MenubarItem>
                </MenubarContent>
              </MenubarMenu>
              <MenubarMenu>
                <MenubarTrigger>Edit</MenubarTrigger>
                <MenubarContent>
                  <MenubarItem disabled>
                    Undo
                    <MenubarShortcut>⌘Z</MenubarShortcut>
                  </MenubarItem>
                  <MenubarItem disabled>
                    Redo
                    <MenubarShortcut>⇧⌘Z</MenubarShortcut>
                  </MenubarItem>
                  <MenubarSeparator />
                  <MenubarItem disabled>
                    Find
                    <MenubarShortcut>⌘F</MenubarShortcut>
                  </MenubarItem>
                  <MenubarSeparator />
                  <MenubarItem disabled
                    >Cut
                    <MenubarShortcut>⌘X</MenubarShortcut>
                  </MenubarItem>
                  <MenubarItem disabled
                    >Copy
                    <MenubarShortcut>⌘C</MenubarShortcut>
                  </MenubarItem>
                  <MenubarItem disabled
                    >Paste
                    <MenubarShortcut>⌘P</MenubarShortcut>
                  </MenubarItem>
                </MenubarContent>
              </MenubarMenu>
              <MenubarMenu>
                <MenubarTrigger>View</MenubarTrigger>
                <MenubarContent>
                  <MenubarItem @click="reload()">
                    Reload
                    <MenubarShortcut>⌘R</MenubarShortcut>
                  </MenubarItem>
                  <MenubarSeparator />
                  <MenubarItem @click="fullscreen()"> Toggle Fullscreen</MenubarItem>
                </MenubarContent>
              </MenubarMenu>
            </Menubar>
            <!--Saving Status-->
            <div class="absolute top-0 right-0">
              <div v-if="activeSavingProcesses > 0">
                <Badge class="py-0 mt-6 mr-6 w-16 justify-center">Saving</Badge>
              </div>
              <div v-else>
                <Badge class="py-0 mt-6 mr-6 w-16 justify-center">Saved</Badge>
              </div>
            </div>

            <ResizablePanelGroup
              v-if="index[0] !== undefined"
              class="w-screen flex flex-1 overflow-scroll"
              direction="horizontal"
            >
              <!--Note List-->
              <ResizablePanel :default-size="25" class="py-2.5 flex flex-col min-w-[150px]">
                <Button
                  @click="createNewDocument()"
                  variant="outline"
                  class="w-auto ml-2.5 mb-2.5 h-10"
                  >New Note
                </Button>
                <div class="rounded-md border h-full ml-2.5 overflow-y-auto">
                  <div v-for="item in index" :key="item" style="cursor: pointer">
                    <div
                      v-if="index.indexOf(item) === activeDocumentIndex"
                      class="bg-gray-200 rounded-sm border mx-1.5 my-1.5 text-sm flex justify-between items-center"
                    >
                      <!--File Icon-->
                      <div class="flex items-center p-2">
                        <File class="ml-1 h-4 w-4"></File>
                      </div>
                      <!--Note Title-->
                      <div class="noteListText activeNoteListText">
                        {{ activeDocument }}
                      </div>
                      <!--Delete Button-->
                      <div class="flex items-center p-2">
                        <Trash2
                          class="ml-1 h-4 w-4"
                          @click="(showNoteDeleteConformation = true), (noteToDelete = item)"
                        ></Trash2>
                      </div>
                    </div>
                    <div
                      v-else
                      @click="getDocument(item)"
                      class="noteListItem passiveNoteListItem rounded-sm border mx-1.5 my-1.5 text-sm flex justify-between items-center"
                    >
                      <!--File Icon-->
                      <div class="flex items-center p-2">
                        <File class="ml-1 h-4 w-4"></File>
                      </div>
                      <!--Note Title-->
                      <div class="noteListText passiveNoteListText">
                        {{ item }}
                      </div>
                      <!--Delete Button-->
                      <div class="flex items-center p-2">
                        <Trash2
                          class="ml-1 h-4 w-4"
                          @click="(showNoteDeleteConformation = true), (noteToDelete = item)"
                        ></Trash2>
                      </div>
                    </div>
                  </div>
                </div>

                <!--Note Deletion conformation-->
                <AlertDialog v-model:open="showNoteDeleteConformation">
                  <AlertDialogContent>
                    <AlertDialogHeader>
                      <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                      <AlertDialogDescription>
                        This action cannot be undone. This will permanently delete your note, called
                        <b>"{{ noteToDelete }}"</b>. Think about it twice.
                      </AlertDialogDescription>
                    </AlertDialogHeader>
                    <AlertDialogFooter>
                      <Button @click="showNoteDeleteConformation = false">Cancel</Button>
                      <Button variant="destructive" @click="deleteDocument(noteToDelete)"
                        >Continue
                      </Button>
                    </AlertDialogFooter>
                  </AlertDialogContent>
                </AlertDialog>
              </ResizablePanel>

              <ResizableHandle
                style="cursor: col-resize !important; width: 6px; background-color: white"
              />

              <ResizablePanel class="w-3/4 pt-2.5 pr-2.5 pl-1 pb-2.5 min-w-[500px]">
                <div v-if="activeDocumentIndex !== null" class="flex flex-col h-full w-full">
                  <!--Content Editor-->
                  <tiptap v-model="activeDocumentContent"></tiptap>
                </div>
                <div
                  v-else
                  class="h-full w-full border rounded-md grid place-content-center flex-col"
                >
                  <Sticker class="mx-auto"></Sticker>
                  <p class="text-center text-lg">Select a note to start editing.</p>
                  <p class="text-center text-sm">
                    Chose any note in the note list or create a new one.
                  </p>
                </div>
              </ResizablePanel>
            </ResizablePanelGroup>
            <!--No file message-->
            <div v-else class="h-screen border rounded-md m-2.5 grid place-content-center flex-col">
              <Frown class="mx-auto"></Frown>
              <p class="text-center text-lg">Create a new note to edit.</p>
              <p class="text-center text-sm">Select "File", "New File" in the Menubar</p>
            </div>
            <!--Error Alert-->
            <div v-if="displayError === true">
              <Alert
                variant="destructive"
                class="absolute bottom-0 right-0 z-50 w-1/3 mr-5 mb-5"
                @click="() => (displayError = false)"
                style="cursor: pointer"
              >
                <AlertCircle class="w-4 h-4" />
                <AlertTitle>Error</AlertTitle>
                <AlertDescription>{{ this.errorMessage }}</AlertDescription>
              </Alert>
            </div>
          </div>
        </div>
        <div v-else>
          <!--Account, Privacy and Security-->
          <Tabs :default-value="tabToOpen" class="mx-2.5 my-2.5 h-screen">
            <TabsList class="grid w-full grid-cols-2">
              <TabsTrigger value="account"> Account</TabsTrigger>
              <TabsTrigger value="privacyAndSecurity"> Privacy & Security</TabsTrigger>
            </TabsList>
            <TabsContent value="account" class="h-full">
              <Card class="relative" style="height: calc(100vh - 70px)">
                <CardHeader>
                  <CardTitle>My Account</CardTitle>
                  <X
                    @click="showSettings = false"
                    style="cursor: pointer"
                    class="mr-5 mt-5 absolute top-0 right-0"
                  ></X>
                </CardHeader>
                <CardContent class="space-y-2">
                  <div class="space-y-1">
                    <Label for="email">E-Mail</Label>
                    <Input id="email" v-model="user.email" disabled />
                  </div>
                  <div class="space-y-1">
                    <Label for="userid">UserID</Label>
                    <Input id="userid" v-model="userId" disabled />
                  </div>
                </CardContent>
                <CardFooter>
                  <Button class="mr-3" @click="logout()">Log Out</Button>
                  <Button class="mr-3" variant="outline" @click="exportData()">Export Data</Button>
                  <Button class="mr-3" variant="outline" @click="showImportConformation = true"
                    >Import Data
                  </Button>
                  <Button
                    class="mr-3"
                    variant="destructive"
                    @click="showAccountDataDeletionConformation = true"
                    >Delete My Data
                  </Button>
                </CardFooter>
                <!--Import Conformation Dialog-->
                <AlertDialog v-model:open="showImportConformation">
                  <AlertDialogContent>
                    <AlertDialogHeader>
                      <AlertDialogTitle>One more thing.</AlertDialogTitle>
                      <AlertDialogDescription>
                        This action cannot be undone. This will permanently replace all the data you
                        have stored in noter. Please select a file below to import.
                      </AlertDialogDescription>
                    </AlertDialogHeader>
                    <div class="grid w-full max-w-sm items-center gap-1.5">
                      <Label for="fileUpload">Picture</Label>
                      <Input id="fileUpload" type="file" />
                    </div>
                    <AlertDialogFooter>
                      <Button variant="destructive" @click="importData()">Import</Button>
                      <Button @click="showImportConformation = false">Cancel</Button>
                    </AlertDialogFooter>
                  </AlertDialogContent>
                </AlertDialog>
                <!--Delete Account Data Conformation Dialog-->
                <AlertDialog v-model:open="showAccountDataDeletionConformation">
                  <AlertDialogContent>
                    <AlertDialogHeader>
                      <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                      <AlertDialogDescription>
                        This action cannot be undone. This will permanently all your accounts data.
                        If you do not want to loose your data, export it before completing this
                        action.
                      </AlertDialogDescription>
                    </AlertDialogHeader>
                    <AlertDialogFooter>
                      <Button variant="destructive" @click="deleteAccountData()"
                        >Delete Data
                      </Button>
                      <Button @click="showAccountDataDeletionConformation = false">Cancel</Button>
                    </AlertDialogFooter>
                  </AlertDialogContent>
                </AlertDialog>
              </Card>
            </TabsContent>
            <TabsContent value="privacyAndSecurity">
              <Card class="relative" style="height: calc(100vh - 70px)">
                <CardHeader>
                  <CardTitle>Privacy & Security</CardTitle>
                  <CardDescription>
                    To ensure privacy, Notes are encrypted using
                    <Button
                      class="link-dark link-offset-2 link-underline-opacity-25 link-underline-opacity-100-hover"
                      variant="link"
                      style="padding: 0"
                      ><a href="https://en.wikipedia.org/wiki/Advanced_Encryption_Standard"
                        >AES-256</a
                      ></Button
                    >
                    before saving to the cloud. The encryption key is crucial for data security and
                    is stored by default in the cloud. For full privacy, store the key yourself, but
                    remember you'll need to enter it every time you switch devices or browsers.
                    Losing the key means losing all your data permanently.
                  </CardDescription>
                  <X
                    @click="showSettings = false"
                    style="cursor: pointer"
                    class="mr-5 mt-5 absolute top-0 right-0"
                  ></X>
                </CardHeader>
                <CardContent class="space-y-2">
                  <p>This is your Secret Key:</p>
                  <div v-if="trust">
                    <div class="flex w-full gap-1.5">
                      <Input
                        type="text"
                        v-model="secretKey"
                        aria-label="Secret Key"
                        style="cursor: text"
                        disabled
                      />
                      <Button variant="destructive" @click="showNewSecretKeyConformation = true">
                        Generate New
                      </Button>
                    </div>
                  </div>
                  <div v-else>
                    <div class="flex w-full gap-1.5">
                      <Input
                        type="text"
                        v-model="secretKey"
                        aria-label="Secret Key"
                        style="cursor: text"
                        disabled
                      />
                      <Button variant="destructive" @click="showChangeSecretKeyConformation = true">
                        Change
                      </Button>
                      <Button variant="destructive" @click="showNewSecretKeyConformation = true">
                        Generate New
                      </Button>
                    </div>
                  </div>
                  <div class="flex items-center space-x-2">
                    <Switch id="secretKeyInCloudSwitch" :checked="trust" @click="trustChanged" />
                    <Label for="secretKeyInCloudSwitch">Store Secret Key in Cloud</Label>
                  </div>
                  <!--Generate Secret Key Conformation Dialog-->
                  <AlertDialog v-model:open="showNewSecretKeyConformation">
                    <AlertDialogContent>
                      <AlertDialogHeader>
                        <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                        <AlertDialogDescription>
                          This action cannot be undone. This will permanently make all the data you
                          have stored in noter useless. If you do not want to loose your data,
                          export it before completing this action.
                        </AlertDialogDescription>
                      </AlertDialogHeader>
                      <AlertDialogFooter>
                        <Button variant="destructive" @click="generateNewSecretKey()"
                          >Generate New
                        </Button>
                        <Button @click="showNewSecretKeyConformation = false">Cancel</Button>
                      </AlertDialogFooter>
                    </AlertDialogContent>
                  </AlertDialog>
                  <!--Change Secret Key Conformation Dialog-->
                  <AlertDialog v-model:open="showChangeSecretKeyConformation">
                    <AlertDialogContent>
                      <AlertDialogHeader>
                        <AlertDialogTitle>Please enter your secret key here:</AlertDialogTitle>
                        <AlertDialogDescription>
                          If you've chosen not to store your secret key in the cloud for maximum
                          privacy and this is a new device, this is the place to import the secret
                          key you got assigned on the device you created your account.
                        </AlertDialogDescription>
                      </AlertDialogHeader>
                      <Input
                        type="text"
                        placeholder="Enter your secret key here"
                        v-model="customSecretKeyInput"
                      />
                      <AlertDialogDescription
                        >The secret key is case sensitive.
                      </AlertDialogDescription>
                      <AlertDialogFooter>
                        <Button variant="destructive" @click="saveNewSecretKey()">Save</Button>
                        <Button @click="showChangeSecretKeyConformation = false">Cancel</Button>
                      </AlertDialogFooter>
                    </AlertDialogContent>
                  </AlertDialog>
                </CardContent>
                <CardFooter></CardFooter>
              </Card>
            </TabsContent>
          </Tabs>
        </div>
      </div>
    </div>
  </div>
</template>
