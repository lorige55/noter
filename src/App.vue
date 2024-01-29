<!-- eslint-disable vue/require-v-for-key -->
<script>
import '@passageidentity/passage-elements/passage-auth'
export default {
  data() {
    return {
      isLoggedIn: false,
      appId: 'JlXUGO3ZcoTO3pK2BSb38cc2',
      authToken: '',
      notes: [],
      noteIndex: ['Loading...'],
      activeDocumentContent: 'Select a note to edit.',
      activeDocument: ''
    }
  },
  methods: {
    async getDocument(item) {
      const app = initializeApp(firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.authToken, item)
      let docSnap = await getDoc(docRef)
      this.activeDocumentContent = docSnap.data().note
      this.activeDocument = item
    },
    saveActiveDocument() {
      this.activeDocumentContent
      const app = initializeApp(firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.authToken, this.activeDocument)
      setDoc(docRef, { note: this.activeDocumentContent })
    },
    async deleteDocument(item) {
      const app = initializeApp(firebaseConfig)
      const db = getFirestore(app)
      const indexToRemove = this.noteIndex.indexOf(item)

      if (indexToRemove !== -1) {
        this.noteIndex.splice(indexToRemove, 1)

        let docRef = doc(db, this.authToken, 'noteIndex')
        setDoc(docRef, { index: this.noteIndex })
      }

      console.log(this.noteIndex)

      await deleteDoc(doc(db, this.authToken, item))
    },
    createNewDocument() {
      const app = initializeApp(firebaseConfig)
      const db = getFirestore(app)
      const newNoteName = prompt('Enter a name for your new note:')

      this.noteIndex.push(newNoteName)

      let docRef = doc(db, this.authToken, 'noteIndex')
      setDoc(docRef, { index: this.noteIndex })

      docRef = doc(db, this.authToken, newNoteName)
      setDoc(docRef, { note: 'This is a note.' })

      this.activeDocumentContent = 'This is a note.'
      this.activeDocument = newNoteName
    }
  },
  beforeMount() {
    if (
      localStorage.getItem('psg_auth_token') !== null &&
      localStorage.getItem('psg_last_login') !== null
    ) {
      this.isLoggedIn = true
      this.authToken = localStorage.getItem('psg_auth_token').split('.')[0]
    }
  },
  async mounted() {
    const app = initializeApp(firebaseConfig)
    const db = getFirestore(app)
    const docRef = doc(db, this.authToken, 'noteIndex')
    let docSnap = await getDoc(docRef)

    if (docSnap.exists()) {
      this.noteIndex = docSnap.data().index

      let i = 0

      do {
        let docRef = doc(db, this.authToken, this.noteIndex[i])
        let docSnap = await getDoc(docRef)
        this.notes.push(docSnap.data().note)
        i++
      } while (i < this.noteIndex.length)
    } else {
      console.log('No noteIndex document!')
    }
  }
}

// Import the functions you need from the SDKs you need
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, getDoc, setDoc, deleteDoc } from 'firebase/firestore'

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: 'AIzaSyBV9FOnKKOBNLuQsCn9T4OdfxT39cRhF6g',
  authDomain: 'noter-6e08f.firebaseapp.com',
  projectId: 'noter-6e08f',
  storageBucket: 'noter-6e08f.appspot.com',
  messagingSenderId: '967538971502',
  appId: '1:967538971502:web:6d31288c8ade545465277f'
}
</script>

<template>
  <div v-if="!isLoggedIn" class="position-absolute top-50 start-50 translate-middle">
    <div class="authContainer">
      <passage-auth :app-id="appId"></passage-auth>
    </div>
  </div>

  <div v-else>
    <div class="position-absolute top-0 start-0" style="width: 200px">
      <ul class="list-group">
        <li class="list-group-item" v-for="item in noteIndex">
          <button class="btn btn-dark" @click="getDocument(item)">{{ item }}</button>
          <buttton class="btn btn-danger" @click="deleteDocument(item)"
            ><i class="bi bi-trash3"></i
          ></buttton>
        </li>
      </ul>
    </div>

    <div class="position-absolute top-50 end-0 translate-middle-y">
      <input
        class="form-control"
        type="text"
        v-model="activeDocumentContent"
        style="width: 1000px; height: 500px"
      />
    </div>
    <div class="position-absolute top-0 end-0">
      <button class="btn btn-dark" @click="saveActiveDocument()">Save</button>
      <button class="btn btn-success" @click="createNewDocument()">
        <i class="bi bi-plus-circle"></i>
      </button>
    </div>
  </div>
</template>
