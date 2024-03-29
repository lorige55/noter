<!-- eslint-disable no-undef -->
<script>
import CryptoJS from 'crypto-js'
import '@passageidentity/passage-elements/passage-auth'
import { PassageUser } from '@passageidentity/passage-auth/passage-user'
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, getDoc, setDoc, deleteDoc } from 'firebase/firestore'
//shadecn Imports:
import { Progress } from '@/components/ui/progress'
import { Button } from '@/components/ui/button'
import {
  Menubar,
  MenubarCheckboxItem,
  MenubarContent,
  MenubarItem,
  MenubarMenu,
  MenubarRadioGroup,
  MenubarRadioItem,
  MenubarSeparator,
  MenubarShortcut,
  MenubarSub,
  MenubarSubContent,
  MenubarSubTrigger,
  MenubarTrigger
} from '@/components/ui/menubar'
import { ScrollArea } from '@/components/ui/scroll-area'
import { Separator } from '@/components/ui/separator'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger
} from '@/components/ui/dropdown-menu'
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
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle
} from '@/components/ui/alert-dialog'
import { Badge } from '@/components/ui/badge'
import { ResizableHandle, ResizablePanel, ResizablePanelGroup } from '@/components/ui/resizable'
import { HoverCard, HoverCardContent, HoverCardTrigger } from '@/components/ui/hover-card'

//icon imports
import {
  MoreHorizontal,
  AlertCircle,
  User,
  Lock,
  Github,
  LifeBuoy,
  LogOut,
  X,
  Info,
  File
} from 'lucide-vue-next'

export default {
  components: {
    Progress,
    Button,
    Menubar,
    MenubarCheckboxItem,
    MenubarContent,
    MenubarItem,
    MenubarMenu,
    MenubarRadioGroup,
    MenubarRadioItem,
    MenubarSeparator,
    MenubarShortcut,
    MenubarSub,
    MenubarSubContent,
    MenubarSubTrigger,
    MenubarTrigger,
    ScrollArea,
    Separator,
    Input,
    Textarea,
    MoreHorizontal,
    DropdownMenu,
    DropdownMenuContent,
    DropdownMenuItem,
    DropdownMenuLabel,
    DropdownMenuSeparator,
    DropdownMenuTrigger,
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
    Switch,
    AlertDialog,
    AlertDialogAction,
    AlertDialogCancel,
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
    File
  },
  watch: {
    activeDocument(newActiveDocument) {
      document.title = 'Noter - ' + newActiveDocument
    }
  },
  data() {
    return {
      isLoggedIn: false,
      user: null,
      userId: null,
      appId: 'LH8ZzpbwJuHH6xGFk6GgmtSC', //Production: 'LH8ZzpbwJuHH6xGFk6GgmtSC'; Development: 'JlXUGO3ZcoTO3pK2BSb38cc2'
      noteIndex: [],
      keyIndex: [],
      activeDocumentContent: 'Loading...',
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
      creatingNewDocument: false,
      documentToRename: '',
      displayError: false,
      errorMessage: 'An Error has occured. Please try again or submit an Issue.',
      ready: false,
      progress: 'Ready, set go!',
      showSettings: false,
      tabToOpen: 'account',
      showNewSecretKeyConformation: false,
      showAccountDataDeletionConformation: false,
      showImportConformation: false,
      showNoteDeleteConformation: false,
      noteToDelete: ''
    }
  },
  methods: {
    async getDocument(item) {
      if (this.activeDocument !== '') {
        await this.saveActiveDocument()
      }
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      //set active document and lastNote
      this.activeDocumentIndex = this.noteIndex.indexOf(item)
      this.activeDocument = this.noteIndex[this.activeDocumentIndex]
      localStorage.setItem('lastNote', this.activeDocument)
      //get document from firebase and set activeDocumentContent to content
      let docRef = doc(db, this.userId, this.keyIndex[this.activeDocumentIndex])
      let docSnap = await getDoc(docRef)
      this.activeDocumentContent = this.decryptString(docSnap.data().note)
    },
    async saveActiveDocument() {
      //input validation
      if (
        this.noteIndex.includes(this.activeDocument) &&
        this.noteIndex[this.activeDocumentIndex] !== this.activeDocument
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
      setDoc(docRef, {
        note: this.encryptString(this.activeDocumentContent),
        title: this.encryptString(this.activeDocument)
      })
      localStorage.setItem('lastNote', this.activeDocument)
      this.noteIndex[this.activeDocumentIndex] = this.activeDocument
    },
    autoSave() {
      //save active document after 3 seconds of inactivity
      this.activeSavingProcesses++
      setTimeout(() => {
        if (this.activeSavingProcesses == 1) {
          this.saveActiveDocument()
        }
        this.activeSavingProcesses--
      }, 3000)
    },
    async deleteDocument(item) {
      //safe active document before proceeding
      if (this.activeDocument !== '') {
        await this.saveActiveDocument()
      }
      //initialize firebase and the index of the item to remove
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const indexToRemove = this.noteIndex.indexOf(item)
      //remove item from lastNote in localStorage if it is the lastNote
      if (localStorage.getItem('lastNote') === this.noteIndex[item]) {
        localStorage.removeItem('lastNote')
      }
      //if indexToRemove is valid
      if (indexToRemove !== -1) {
        //change active document if it is the one to be removed
        if (this.activeDocument === item && this.noteIndex.length > 0) {
          if (indexToRemove === 0) {
            this.getDocument(this.noteIndex[0])
          } else {
            this.getDocument(this.noteIndex[indexToRemove - 1])
          }
        }
        //delete document from firebase
        await deleteDoc(doc(db, this.userId, this.keyIndex[indexToRemove]))
        //remove key from keyIndex and push new keyIndex to firebase
        this.keyIndex.splice(indexToRemove, 1)
        let docRef = doc(db, this.userId, 'keyIndex')
        setDoc(docRef, { index: this.keyIndex })
        //remove item from noteIndex and shortened
        this.noteIndex.splice(indexToRemove, 1)
        this.showNoteDeleteConformation = false
      } else {
        console.log('An Error has occured. Please try again or create an Issue on GitHub.')
        this.errorMessage = 'An Error has occured. Please try again or submit an Issue.'
        this.displayError = true
      }
    },
    async createNewDocument() {
      await this.saveActiveDocument()
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let newNoteName = 'New Note'
      let nameFound = false
      for (let i = 0; i < this.noteIndex.length && nameFound == false; i++) {
        if (this.noteIndex.includes(newNoteName)) {
          newNoteName = 'New Note ' + (i + 1)
        } else {
          nameFound = true
        }
      }
      //push new Note to Index
      this.noteIndex.push(newNoteName)
      //generate new key and push it to keyIndex
      let key = this.generateKey(128)
      this.keyIndex.push(key)
      //push new keyIndex to firebase
      let docRef = doc(db, this.userId, 'keyIndex')
      setDoc(docRef, { index: this.keyIndex })
      //push new note to firebase
      docRef = doc(db, this.userId, key)
      setDoc(docRef, {
        note: this.encryptString('This is a new note. Feel free to edit it!'),
        title: this.encryptString(newNoteName)
      })
      //set activeDocument and lastNote to new note
      this.activeDocumentContent = 'This is a new note. Feel free to edit it!'
      this.activeDocument = newNoteName
      this.activeDocumentIndex = this.noteIndex.indexOf(newNoteName)
      localStorage.setItem('lastNote', this.activeDocument)
      this.creatingNewDocument = false
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
    generateKey(length) {
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
    async trustChanged() {
      if (!this.trust) {
        this.trust = true
        //if trust changed to true, push secretKey to firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        setDoc(docRef, { key: this.secretKey })
      } else if (this.trust) {
        this.trust = false
        //if trust changed to false remove secretKey from firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        setDoc(docRef, { key: 'nah' })
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
          setDoc(docRef, {
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
    }
  },
  async mounted() {
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
    } catch (error) {
      console.log(error)
      this.errorMessage = 'An Error has occured. Please try again or submit an Issue.'
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
          this.$refs.profileModal.show()
        }
      }
    } else {
      //if there is no secretKey in firebase, generate a new one and push it to firebase
      let key = this.generateKey(256)
      docRef = doc(db, this.userId, 'secretKey')
      setDoc(docRef, { key: key })
      this.secretKey = key
    }
    this.progress = 'Syncing your notes...'
    //get keyIndex from firebase
    if (this.secretKey !== '') {
      docRef = doc(db, this.userId, 'keyIndex')
      docSnap = await getDoc(docRef)
      //if keyIndex exists, get it
      if (docSnap.exists() && docSnap.data().index[0].length > 0) {
        this.keyIndex = docSnap.data().index
        //loop through keyIndex and build noteIndex
        for (let i = 0; i < this.keyIndex.length; i++) {
          let docRef = doc(db, this.userId, this.keyIndex[i])
          let docSnap = await getDoc(docRef)
          this.noteIndex[i] = this.decryptString(docSnap.data().title)
        }
        this.progress = 'Loading your last note...'
        //recover last note or select first in array if equal to null
        if (
          localStorage.getItem('lastNote') == undefined ||
          localStorage.getItem('lastNote') === 'undefined'
        ) {
          localStorage.setItem('lastNote', this.noteIndex[0])
        }
        this.getDocument(localStorage.getItem('lastNote'))
      } else {
        //if keyIndex does not exist, create a new one and push it to firebase
        let key = this.generateKey(128)
        this.keyIndex = [key]
        setDoc(docRef, { index: this.keyIndex })
        //create default note
        this.noteIndex = ['Welcome!']
        docRef = doc(db, this.userId, key)
        setDoc(docRef, {
          note: this.encryptString('This is your first note! Have fun!'),
          title: this.encryptString('Welcome!')
        })
        //set default note to active Note
        this.activeDocument = 'Welcome!'
        this.activeDocumentContent = 'This is your first note! Have fun!'
        this.activeDocumentIndex = 0
      }
    }
    this.progress = 'Done!'
    this.ready = true
  }
}
</script>

<template>
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
            <Button variant="outline" class="mt-3"><Info></Info></Button
          ></HoverCardTrigger>
          <HoverCardContent>
            <p class="text-sm text-center">
              This should only take a few seconds. If it takes too long, a slow internet connection
              or corrupt data could be the cause. If you think your data is corrupt, consider
              resetting it. If you have a backup of your data, you can import it after resetting.
            </p>
            <Button variant="destructive" @click="deleteAccountData()" class="w-full mt-3"
              >Delete My Data</Button
            >
            ></HoverCardContent
          >
        </HoverCard>
      </div>
    </div>
    <div v-else>
      <div v-if="!showSettings">
        <div class="relative flex flex-col h-screen w-100">
          <!--Menubar-->
          <Menubar class="h-11 mx-2.5 mt-2.5">
            <MenubarMenu>
              <MenubarTrigger>Home</MenubarTrigger>
              <MenubarContent>
                <MenubarItem @click="(tabToOpen = 'account'), (showSettings = true)">
                  <User class="mr-2 h-4 w-4" /> Account
                </MenubarItem>
                <MenubarItem @click="(tabToOpen = 'privacyAndSecurity'), (showSettings = true)">
                  <Lock class="mr-2 h-4 w-4" /> Privacy & Security
                </MenubarItem>
                <MenubarSeparator />
                <a href="https://github.com/lorige55/noter" target="_blank">
                  <MenubarItem> <Github class="mr-2 h-4 w-4" /> GitHub </MenubarItem>
                </a>
                <MenubarItem disabled> <LifeBuoy class="mr-2 h-4 w-4" /> Support </MenubarItem>
                <MenubarSeparator />
                <MenubarItem @click="logout()">
                  <LogOut class="mr-2 h-4 w-4" /> Logout
                </MenubarItem>
              </MenubarContent>
            </MenubarMenu>
            <MenubarMenu>
              <MenubarTrigger>File</MenubarTrigger>
              <MenubarContent>
                <MenubarItem @click="createNewDocument()"> New File </MenubarItem>
                <MenubarItem disabled> New Folder </MenubarItem>
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
                  >Import Data</MenubarItem
                >
                <MenubarItem @click="exportData()">Export Data</MenubarItem>
              </MenubarContent>
            </MenubarMenu>
            <MenubarMenu>
              <MenubarTrigger>Edit</MenubarTrigger>
              <MenubarContent>
                <MenubarItem disabled> Undo <MenubarShortcut>⌘Z</MenubarShortcut> </MenubarItem>
                <MenubarItem disabled> Redo <MenubarShortcut>⇧⌘Z</MenubarShortcut> </MenubarItem>
                <MenubarSeparator />
                <MenubarSub>
                  <MenubarSubTrigger>Find</MenubarSubTrigger>
                  <MenubarSubContent>
                    <MenubarItem disabled>Search the web</MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem disabled>Find...</MenubarItem>
                    <MenubarItem disabled>Find Next</MenubarItem>
                    <MenubarItem disabled>Find Previous</MenubarItem>
                  </MenubarSubContent>
                </MenubarSub>
                <MenubarSeparator />
                <MenubarItem disabled>Cut</MenubarItem>
                <MenubarItem disabled>Copy</MenubarItem>
                <MenubarItem disabled>Paste</MenubarItem>
              </MenubarContent>
            </MenubarMenu>
            <MenubarMenu>
              <MenubarTrigger>View</MenubarTrigger>
              <MenubarContent>
                <MenubarItem @click="reload()">
                  Reload <MenubarShortcut>⌘R</MenubarShortcut>
                </MenubarItem>
                <MenubarSeparator />
                <MenubarItem @click="fullscreen()"> Toggle Fullscreen </MenubarItem>
              </MenubarContent>
            </MenubarMenu>
          </Menubar>
          <!--Saving Status-->
          <div class="absolute top-0 right-0">
            <div v-if="activeSavingProcesses > 0">
              <Badge class="py-0 mt-6 mr-6">Saving</Badge>
            </div>
            <div v-else>
              <Badge class="py-0 mt-6 mr-6">Saved</Badge>
            </div>
          </div>

          <ResizablePanelGroup class="w-screen flex flex-1 overflow-scroll" direction="horizontal">
            <!--Note List-->
            <ResizablePanel :default-size="25" class="py-2.5">
              <div class="h-full rounded-md border ml-2.5 overflow-auto">
                <div v-for="item in noteIndex" :key="item" style="cursor: pointer">
                  <div
                    v-if="noteIndex.indexOf(item) === activeDocumentIndex"
                    class="bg-gray-200 rounded-sm border mx-1.5 my-1.5 px-4 py-3 text-sm flex justify-between items-center"
                  >
                    <File class="mx-1 h-4 w-4"></File>
                    <div class="noteListText activeNoteListText">
                      {{ activeDocument }}
                    </div>

                    <!--Dropdown Menu-->
                    <DropdownMenu>
                      <DropdownMenuTrigger>
                        <div class="flex items-center">
                          <MoreHorizontal class="ml-1 h-4 w-4"></MoreHorizontal>
                        </div>
                      </DropdownMenuTrigger>
                      <DropdownMenuContent>
                        <DropdownMenuItem disabled> Rename </DropdownMenuItem>
                        <DropdownMenuItem
                          @click="(showNoteDeleteConformation = true), (noteToDelete = item)"
                        >
                          Delete
                        </DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
                  </div>
                  <div
                    v-else
                    @click="getDocument(item)"
                    class="noteListItem passiveNoteListItem rounded-sm border mx-1.5 my-1.5 px-4 py-3 text-sm flex justify-between items-center"
                  >
                    <File class="mx-1 h-4 w-4"></File>
                    <div class="noteListText passiveNoteListText">
                      {{ item }}
                    </div>

                    <!--Dropdown Menu-->
                    <DropdownMenu>
                      <DropdownMenuTrigger>
                        <div class="flex items-center">
                          <MoreHorizontal class="ml-1 h-4 w-4"></MoreHorizontal>
                        </div>
                      </DropdownMenuTrigger>
                      <DropdownMenuContent>
                        <DropdownMenuItem disabled> Rename </DropdownMenuItem>
                        <DropdownMenuItem
                          @click="(showNoteDeleteConformation = true), (noteToDelete = item)"
                        >
                          Delete
                        </DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
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
                      >Continue</Button
                    >
                  </AlertDialogFooter>
                </AlertDialogContent>
              </AlertDialog>
            </ResizablePanel>

            <ResizableHandle
              style="cursor: col-resize !important; width: 6px; background-color: white"
            />

            <ResizablePanel class="flex flex-col w-3/4 pt-2.5 pr-2.5">
              <!--Editor-->
              <div class="h-full pl-1 flex flex-col">
                <!--Title Editor-->
                <Input
                  class="customInput h-11 justify-between text-base font-semibold"
                  type="text"
                  v-model="activeDocument"
                  @input="autoSave()"
                />

                <!--Content Editor-->
                <Textarea
                  class="customInput mt-2.5 justify-between flex-grow mb-2.5"
                  type="text"
                  v-model="activeDocumentContent"
                  @input="autoSave()"
                ></Textarea>
              </div>
            </ResizablePanel>
          </ResizablePanelGroup>
          <!--Error Alert-->
          <div v-if="displayError == true">
            <Alert
              variant="destructive"
              class="absolute bottom-0 right-0 z-50 fixed w-1/3 mr-5 mb-5"
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
            <TabsTrigger value="account"> Account </TabsTrigger>
            <TabsTrigger value="privacyAndSecurity"> Privacy & Security </TabsTrigger>
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
                  >Import Data</Button
                >
                <Button
                  class="mr-3"
                  variant="destructive"
                  @click="showAccountDataDeletionConformation = true"
                  >Delete My Data</Button
                >
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
                      This action cannot be undone. This will permanently all your accounts data. If
                      you do not want to loose your data, export it before completing this action.
                    </AlertDialogDescription>
                  </AlertDialogHeader>
                  <AlertDialogFooter>
                    <Button variant="destructive" @click="deleteAccountData()">Delete Data</Button>
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
                  before saving to the cloud. The encryption key is crucial for data security and is
                  stored by default in the cloud. For full privacy, store the key yourself, but
                  remember you'll need to enter it every time you switch devices or browsers. Losing
                  the key means losing all your data permanently.
                </CardDescription>
                <X
                  @click="showSettings = false"
                  style="cursor: pointer"
                  class="mr-5 mt-5 absolute top-0 right-0"
                ></X>
              </CardHeader>
              <CardContent class="space-y-2">
                <div v-if="trust">
                  <p>This is your Secret Key:</p>
                  <Input
                    type="text"
                    v-model="secretKey"
                    aria-label="Secret Key"
                    disabled
                    readonly
                  />
                </div>
                <div v-else>
                  <p>Please enter your Secret Key:</p>
                  <div class="flex w-full gap-1.5">
                    <Input type="text" v-model="secretKey" aria-label="Secret Key" disabled />
                    <Button variant="destructive" @click="showNewSecretKeyConformation = true">
                      Generate
                    </Button>
                  </div>
                </div>
                <div class="flex items-center space-x-2">
                  <Switch id="secretKeyInCloudSwitch" :checked="trust" @click="trustChanged" />
                  <Label for="secretKeyInCloudSwitch">Store Secret Key in Cloud</Label>
                </div>
                <!--Secret Key Conformation Dialog-->
                <AlertDialog v-model:open="showNewSecretKeyConformation">
                  <AlertDialogContent>
                    <AlertDialogHeader>
                      <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                      <AlertDialogDescription>
                        This action cannot be undone. This will permanently make all the data you
                        have stored in noter useless. If you do not want to loose your data, export
                        it before completing this action.
                      </AlertDialogDescription>
                    </AlertDialogHeader>
                    <AlertDialogFooter>
                      <Button variant="destructive" @click="generateNewSecretKey()"
                        >Generate New</Button
                      >
                      <Button @click="showNewSecretKeyConformation = false">Cancel</Button>
                    </AlertDialogFooter>
                  </AlertDialogContent>
                </AlertDialog>
              </CardContent>
              <CardFooter> </CardFooter>
            </Card>
          </TabsContent>
        </Tabs>
      </div>
    </div>
  </div>
</template>
