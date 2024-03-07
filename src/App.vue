<!-- eslint-disable no-undef -->
<script>
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

//icon imports
import {
  MoreHorizontal,
  AlertCircle,
  User,
  Lock,
  Github,
  LifeBuoy,
  LogOut,
  X
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
    AlertDialogTitle
  },
  data() {
    return {
      isLoggedIn: false,
      user: null,
      userId: null,
      appId: 'JlXUGO3ZcoTO3pK2BSb38cc2',
      noteIndex: [],
      shortenedNoteIndex: ['Loading...'],
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
      secretKeyStatus: 'locked',
      creatingNewDocument: false,
      documentToRename: '',
      displayError: false,
      errorMessage: 'An Error has occured. Please try again or submit an Issue.',
      ready: false,
      progress: 0,
      showSettings: false,
      tabToOpen: 'account',
      showNewSecretKeyConformation: false,
      showAccountDataDeletionConformation: false,
      showImportConformation: false
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
      this.activeDocumentIndex = this.shortenedNoteIndex.indexOf(item)
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
          'You cannot name a note the same as another note. Please rename the note. It will not be saved under this title.'
        this.displayError = true
        return
      }
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.userId, this.keyIndex[this.activeDocumentIndex])
      console.log('Saved: ' + this.activeDocument)
      //save document to firebase
      setDoc(docRef, {
        note: this.encryptString(this.activeDocumentContent),
        title: this.encryptString(this.activeDocument)
      })
      this.noteIndex[this.activeDocumentIndex] = this.activeDocument
      this.shorten('1')
    },
    autoSave() {
      //save active document after 3 seconds of inactivity
      this.activeSavingProcesses++
      this.shorten('1')
      setTimeout(() => {
        if (this.activeSavingProcesses == 1) {
          this.saveActiveDocument()
        }
        this.activeSavingProcesses--
      }, 3000)
    },
    async deleteDocument(item) {
      //initialize firebase and the index of the item to remove
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const indexToRemove = this.shortenedNoteIndex.indexOf(item)
      //remove item from lastNote in localStorage if it is the lastNote
      if (
        localStorage.getItem('lastNote') === this.noteIndex[this.shortenedNoteIndex.indexOf(item)]
      ) {
        localStorage.removeItem('lastNote')
      }
      //if indexToRemove is valid
      if (indexToRemove !== -1) {
        //remove item from noteIndex and shorten
        this.noteIndex.splice(indexToRemove, 1)
        this.shorten('1')
        //delete document from firebase
        await deleteDoc(doc(db, this.userId, this.keyIndex[indexToRemove]))
        //remove key from keyIndex and push new keyIndex to firebase
        this.keyIndex.splice(indexToRemove, 1)
        let docRef = doc(db, this.userId, 'keyIndex')
        setDoc(docRef, { index: this.keyIndex })
        location.reload()
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
      this.shorten('1')
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
      this.activeDocumentIndex = this.shortenedNoteIndex.indexOf(newNoteName)
      localStorage.setItem('lastNote', this.activeDocument)
      this.creatingNewDocument = false
    },
    async renameDocument(item, attempt) {
      if (attempt == 1) {
        //show input for new note name
        this.documentToRename = this.noteIndex[this.shortenedNoteIndex.indexOf(item)]
      } else if (attempt == 2) {
        //get a new name for the note
        let newName = document.getElementById('changeNoteNameInput').value
        //delete old document
        //initialize firebase and the index of the item to remove
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        const indexToRename = this.noteIndex.indexOf(this.documentToRename)
        //remove item from lastNote in localStorage if it is the lastNote
        if (
          localStorage.getItem('lastNote') === this.noteIndex[this.shortenedNoteIndex.indexOf(item)]
        ) {
          localStorage.removeItem('lastNote')
        }
        //if indexToRename is valid
        if (indexToRename !== -1) {
          //rename item in noteIndex and shorten
          this.noteIndex[indexToRename] = newName
          this.shorten('1')
          //backup content of note
          let docRef = doc(db, this.userId, this.keyIndex[indexToRename])
          let docSnap = await getDoc(docRef)
          let backupContent = docSnap.data().note
          //rename document in firebase
          docRef = doc(db, this.userId, this.keyIndex[indexToRename])
          setDoc(docRef, {
            note: backupContent,
            title: this.encryptString(newName)
          })
        } else {
          this.errorMessage = 'An Error has occured. Please try again or create an Issue on GitHub.'
          this.displayError = true
        }
        this.documentToRename = ''
        location.reload()
      }
    },
    shorten(par) {
      let charLimit = 30
      if (par === '1') {
        for (let i = 0; i < this.noteIndex.length; ) {
          if (this.noteIndex[i].length > charLimit) {
            let shortenedString = this.noteIndex[i].split('')
            shortenedString.splice(charLimit, this.noteIndex[i].length - charLimit, '...')
            this.shortenedNoteIndex[i] = shortenedString.join('')
          } else {
            this.shortenedNoteIndex[i] = this.noteIndex[i]
          }
          i++
        }
      } else {
        if (par.length > charLimit) {
          let shortenedString = par.split('')
          shortenedString.splice(charLimit, par.length - charLimit, '...')
          return shortenedString.join('')
        } else {
          return par
        }
      }
    },
    encryptString(string) {
      //verify that input is a string
      string = string.toString()
      //encrypt with AES-256
      let encrypted = CryptoJS.Rabbit.encrypt(string, this.secretKey)
      //encode to base64
      return btoa(encrypted)
    },
    decryptString(string) {
      //verify that input is a string
      string = string.toString()
      //decode from base64
      string = atob(string)
      //decrypt with AES-256
      const bytes = CryptoJS.Rabbit.decrypt(string, this.secretKey)
      return bytes.toString(CryptoJS.enc.Utf8)
    },
    generateKey(length) {
      //generate a random key
      let key = CryptoJS.lib.WordArray.random(length / 8)
      key.toString(CryptoJS.enc.Base64)
      //encode to base64 and return
      return btoa(key)
    },
    changeSecretKey() {
      if (this.secretKeyStatus == 'locked') {
        //changed status
        this.secretKeyStatus = 'unlocked'
      } else {
        //changed status
        this.secretKeyStatus = 'locked'
        //if trust is true, push new key to firebase
        if (this.trust) {
          const app = initializeApp(this.firebaseConfig)
          const db = getFirestore(app)
          let docRef = doc(db, this.userId, 'secretKey')
          setDoc(docRef, { key: this.secretKey })
        }
        //save new key to localStorage
        localStorage.setItem('secretKey', this.secretKey)
      }
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
      location.reload()
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
      location.reload()
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
            docRef = doc(db, this.userId, parsedData.keyIndex[i])
            let key = parsedData.keyIndex[i]
            setDoc(docRef, {
              note: parsedData[key].note,
              title: parsedData[key].title
            })
          }
          location.reload()
        } catch (error) {
          console.error('Error parsing JSON:', error)
        }
      }
      reader.readAsText(file)
      this.showImportConformation = false
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
    this.progress = 20
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
    this.progress = 100
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
        //shorten, obvi
        this.shorten('1')
        //recover last note or select first in array if equal to null
        if (
          localStorage.getItem('lastNote') == undefined ||
          localStorage.getItem('lastNote') === 'undefined'
        ) {
          localStorage.setItem('lastNote', this.noteIndex[0])
          console.log('changed last note')
        }
        this.getDocument(localStorage.getItem('lastNote'))
      } else {
        //if keyIndex does not exist, create a new one and push it to firebase
        let key = this.generateKey(128)
        this.keyIndex = [key]
        setDoc(docRef, { index: this.keyIndex })
        //create default note
        this.noteIndex = ['Welcome!']
        this.shorten('1')
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
        <Progress class="w-1/3 mb-3" :model-value="progress"></Progress>
        <Button variant="destructive" @click="deleteAccountData()">Delete My Data</Button>
      </div>
    </div>
    <div v-else>
      <div v-if="!showSettings">
        <div class="relative flex flex-col h-screen w-100">
          <!--Menubar-->
          <Menubar class="h-11 mx-2.5 my-2.5">
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
                <MenubarItem> Delete </MenubarItem>
                <MenubarSeparator />
                <MenubarSub>
                  <MenubarSubTrigger>Share</MenubarSubTrigger>
                  <MenubarSubContent>
                    <MenubarItem>Link</MenubarItem>
                    <MenubarItem>Email</MenubarItem>
                    <MenubarItem>SMS</MenubarItem>
                  </MenubarSubContent>
                </MenubarSub>
              </MenubarContent>
            </MenubarMenu>
            <MenubarMenu>
              <MenubarTrigger>Edit</MenubarTrigger>
              <MenubarContent>
                <MenubarItem> Undo <MenubarShortcut>⌘Z</MenubarShortcut> </MenubarItem>
                <MenubarItem> Redo <MenubarShortcut>⇧⌘Z</MenubarShortcut> </MenubarItem>
                <MenubarSeparator />
                <MenubarSub>
                  <MenubarSubTrigger>Find</MenubarSubTrigger>
                  <MenubarSubContent>
                    <MenubarItem>Search the web</MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem>Find...</MenubarItem>
                    <MenubarItem>Find Next</MenubarItem>
                    <MenubarItem>Find Previous</MenubarItem>
                  </MenubarSubContent>
                </MenubarSub>
                <MenubarSeparator />
                <MenubarItem>Cut</MenubarItem>
                <MenubarItem>Copy</MenubarItem>
                <MenubarItem>Paste</MenubarItem>
              </MenubarContent>
            </MenubarMenu>
            <MenubarMenu>
              <MenubarTrigger>View</MenubarTrigger>
              <MenubarContent>
                <MenubarCheckboxItem>Always Show Bookmarks Bar</MenubarCheckboxItem>
                <MenubarCheckboxItem checked> Always Show Full URLs </MenubarCheckboxItem>
                <MenubarSeparator />
                <MenubarItem inset> Reload <MenubarShortcut>⌘R</MenubarShortcut> </MenubarItem>
                <MenubarItem disabled inset>
                  Force Reload <MenubarShortcut>⇧⌘R</MenubarShortcut>
                </MenubarItem>
                <MenubarSeparator />
                <MenubarItem inset> Toggle Fullscreen </MenubarItem>
                <MenubarSeparator />
                <MenubarItem inset> Hide Sidebar </MenubarItem>
              </MenubarContent>
            </MenubarMenu>
          </Menubar>

          <div class="flex flex-1 h-full mb-2.5">
            <!--Note List-->
            <ScrollArea class="h-full rounded-md border mx-2.5 w-1/4 overflow-y-auto">
              <div class="p-4">
                <div v-for="item in shortenedNoteIndex" :key="item">
                  <a class="text-sm flex justify-between items-center" style="cursor: pointer">
                    <div
                      v-if="shortenedNoteIndex.indexOf(item) === activeDocumentIndex"
                      class="font-semibold"
                    >
                      {{ this.shorten(activeDocument) }}
                    </div>
                    <div v-else @click="getDocument(item)">
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
                        <DropdownMenuItem @click="deleteDocument(item)"> Delete </DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
                  </a>
                  <Separator class="my-2" />
                </div>
              </div>
            </ScrollArea>

            <!--Editor-->
            <div class="flex flex-col w-3/4 mr-2.5">
              <!--Title Editor-->
              <Input
                class="h-11 justify-between text-base font-semibold"
                type="text"
                v-model="activeDocument"
                @click="autoSave()"
              />

              <!--Content Editor-->
              <Textarea
                class="h-11 mt-2.5 justify-between flex-1 h-screen"
                type="text"
                v-model="activeDocumentContent"
                @input="autoSave()"
              ></Textarea>
            </div>
          </div>
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
                    <Button variant="outline" @click="changeSecretKey()" id="lockButton">
                      <div v-if="secretKeyStatus == 'locked'"><i class="bi bi-lock-fill"></i></div>
                      <div v-else><i class="bi bi-unlock-fill"></i></div>
                    </Button>
                    <Input
                      type="text"
                      v-model="secretKey"
                      aria-label="Secret Key"
                      :disabled="secretKeyStatus === 'locked'"
                      :readonly="secretKeyStatus === 'locked'"
                    />
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
