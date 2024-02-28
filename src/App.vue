<!-- eslint-disable no-undef -->
<script>
import '@passageidentity/passage-elements/passage-auth'
import { PassageUser } from '@passageidentity/passage-auth/passage-user'
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, getDoc, setDoc, deleteDoc } from 'firebase/firestore'
//shadecn Imports:
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

export default {
  components: {
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
    Textarea
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
      errorMessage: 'An Error has occured. Please try again or create an Issue on GitHub.',
      conformationMessage: '',
      itemToDelete: '',
      activeSavingProcesses: 0,
      secretKey: '',
      trust: true,
      secretKeyStatus: 'locked',
      creatingNewDocument: false,
      documentToRename: ''
    }
  },
  methods: {
    async getDocument(item) {
      //save active document before switching
      if (this.activeDocument) {
        this.saveActiveDocument()
        this.activeSavingProcesses = 0
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
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.userId, this.keyIndex[this.activeDocumentIndex])
      console.log(this.activeDocument)
      //save document to firebase
      setDoc(docRef, {
        note: this.encryptString(this.activeDocumentContent),
        title: this.encryptString(this.activeDocument)
      })
      //rename document in firebase if document has been renamed
      const indexOfItem = this.shortenedNoteIndex.indexOf(this.activeDocument)
      if (this.activeDocument !== this.noteIndex[indexOfItem]) {
        //remove item from lastNote in localStorage if it is the lastNote
        if (localStorage.getItem('lastNote') === this.noteIndex[this.activeDocumentIndex]) {
          localStorage.removeItem('lastNote')
        }
        //if indexOfItem is valid
        if (indexOfItem !== -1) {
          //rename item in noteIndex and shortenNoteIndex
          this.noteIndex[indexOfItem] = this.activeDocument
          this.shortenNoteIndex()
          //backup content of note
          let docRef = doc(db, this.userId, this.keyIndex[indexOfItem])
          let docSnap = await getDoc(docRef)
          let backupContent = docSnap.data().note
          //rename document in firebase
          docRef = doc(db, this.userId, this.keyIndex[indexOfItem])
          setDoc(docRef, {
            note: backupContent,
            title: this.encryptString(this.activeDocument)
          })
        }
      }
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
    async deleteDocument(item, attempt) {
      //if first attempt, show conformation modal
      if (attempt == 1) {
        this.conformationMessage =
          'Are you sure you want to delete "' +
          this.noteIndex[this.shortenedNoteIndex.indexOf(item)] +
          '"? This action cannot be undone.'

        this.conformationModal.show()

        this.itemToDelete = item
      } else if (attempt == 2) {
        //recover item from first attempt
        item = this.itemToDelete
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
          //remove item from noteIndex and shortenNoteIndex
          this.noteIndex.splice(indexToRemove, 1)
          this.shortenNoteIndex()
          //delete document from firebase
          await deleteDoc(doc(db, this.userId, this.keyIndex[indexToRemove]))
          //remove key from keyIndex and push new keyIndex to firebase
          this.keyIndex.splice(indexToRemove, 1)
          let docRef = doc(db, this.userId, 'keyIndex')
          setDoc(docRef, { index: this.keyIndex })
        } else {
          this.errorMessage = 'An Error has occured. Please try again or create an Issue on GitHub.'
          this.errorModal.show()
        }
        //reload page for the user to see the changes
        location.reload()
      }
    },
    createNewDocument() {
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      //push new Note to Index
      this.noteIndex.push('New Note')
      this.shortenNoteIndex()
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
        title: this.encryptString('New Note')
      })
      //set activeDocument and lastNote to new note
      this.activeDocumentContent = 'This is a new note. Feel free to edit it!'
      this.activeDocument = 'New Note'
      this.activeDocumentIndex = this.shortenedNoteIndex.indexOf('New Note')
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
          //rename item in noteIndex and shortenNoteIndex
          this.noteIndex[indexToRename] = newName
          this.shortenNoteIndex()
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
          this.errorModal.show()
        }
        this.documentToRename = ''
        location.reload()
      }
    },
    shortenNoteIndex() {
      //set char limit
      let charLimit = 30
      //loop through noteIndex and shorten each item and set it to its position in shrotenedNoteIndex
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
      this.errorMessage =
        'An Error has occured: ' + error + '. Please try again or create an Issue on GitHub.'
      //this.errorModal.show()
    }
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
        //shortenNoteIndex, obvi
        this.shortenNoteIndex()
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
        this.shortenNoteIndex()
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
    <div class="flex flex-col h-screen">
      <!--Menubar-->
      <Menubar class="h-11 mx-2.5 my-2.5">
        <MenubarMenu>
          <MenubarTrigger>General</MenubarTrigger>
          <MenubarContent>
            <MenubarItem disabled> My Profile </MenubarItem>
            <MenubarItem disabled> Settings </MenubarItem>
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
              <MenubarSubTrigger disabled>Share</MenubarSubTrigger>
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
              <a @click="getDocument(item)" class="text-sm" style="cursor: pointer">
                {{ item }}
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
    </div>
  </div>
</template>
