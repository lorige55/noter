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

//icon imports
import { MoreHorizontal, AlertCircle, User, Settings, Github, LifeBuoy } from 'lucide-vue-next'

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
    Settings,
    Github,
    LifeBuoy
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
      progress: 0
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
        //if trust changed to false remove secretKey from firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        setDoc(docRef, { key: 'nah' })
        localStorage.setItem('secretKey', this.secretKey)
      } else if (this.trust) {
        //if trust changed to true, push secretKey to firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.userId, 'secretKey')
        setDoc(docRef, { key: this.secretKey })
      }
    },
    async logout() {
      localStorage.removeItem('psg_auth_token')
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
      <div class="flex justify-center items-center h-screen">
        <Progress class="w-1/3" :model-value="progress"></Progress>
      </div>
    </div>
    <div v-else>
      <div class="relative flex flex-col h-screen w-100">
        <!--Menubar-->
        <Menubar class="h-11 mx-2.5 my-2.5">
          <MenubarMenu>
            <MenubarTrigger>Home</MenubarTrigger>
            <MenubarContent>
              <MenubarItem disabled> <User class="mr-2 h-4 w-4" /> Profile </MenubarItem>
              <MenubarItem disabled> <Settings class="mr-2 h-4 w-4" /> Settings </MenubarItem>
              <MenubarSeparator />
              <a href="https://github.com/lorige55/noter" target="_blank">
                <MenubarItem> <Github class="mr-2 h-4 w-4" /> GitHub </MenubarItem>
              </a>
              <MenubarItem disabled> <LifeBuoy class="mr-2 h-4 w-4" /> Support </MenubarItem>
              <MenubarSeparator />
              <MenubarItem @click="logout()"> Logout </MenubarItem>
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
              class="h-11 justify-between"
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
  </div>
</template>
