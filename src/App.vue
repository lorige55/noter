<!-- eslint-disable vue/require-v-for-key -->
<script>
import '@passageidentity/passage-elements/passage-auth'
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, getDoc, setDoc, deleteDoc } from 'firebase/firestore'

export default {
  data() {
    return {
      isLoggedIn: false,
      appId: 'JlXUGO3ZcoTO3pK2BSb38cc2',
      authToken: '',
      noteIndex: ['Loading...'],
      activeDocumentContent: 'Select a note to edit.',
      activeDocument: '',
      firebaseConfig: {
        apiKey: 'AIzaSyBV9FOnKKOBNLuQsCn9T4OdfxT39cRhF6g',
        authDomain: 'noter-6e08f.firebaseapp.com',
        projectId: 'noter-6e08f',
        storageBucket: 'noter-6e08f.appspot.com',
        messagingSenderId: '967538971502',
        appId: '1:967538971502:web:6d31288c8ade545465277f'
      },
      errorMessage: 'An Error has occured. Please try again or create an Issue on GitHub.'
    }
  },
  methods: {
    async getDocument(item) {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.authToken, item)
      let docSnap = await getDoc(docRef)
      this.activeDocumentContent = docSnap.data().note
      this.activeDocument = item
    },
    saveActiveDocument() {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.authToken, this.activeDocument)
      setDoc(docRef, { note: this.activeDocumentContent })
    },
    async deleteDocument(item) {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const indexToRemove = this.noteIndex.indexOf(item)

      if (indexToRemove !== -1) {
        this.noteIndex.splice(indexToRemove, 1)

        let docRef = doc(db, this.authToken, 'noteIndex')
        setDoc(docRef, { index: this.noteIndex })
      }

      console.log(this.noteIndex)

      await deleteDoc(doc(db, this.authToken, item))
      console.log('Document deleted.')
    },
    createNewDocument() {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const newNoteName = prompt('Enter a name for your new note:')

      this.noteIndex.push(newNoteName)

      let docRef = doc(db, this.authToken, 'noteIndex')
      setDoc(docRef, { index: this.noteIndex })

      docRef = doc(db, this.authToken, newNoteName)
      setDoc(docRef, { note: 'This is a note.' })

      this.activeDocumentContent = 'This is a note.'
      this.activeDocument = newNoteName
    },
    renameDocument(item) {
      let newName = prompt('Enter a new name for this note:')

      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)

      const indexToRemove = this.noteIndex.indexOf(item)

      if (indexToRemove !== -1) {
        this.noteIndex.splice(indexToRemove, 1, newName)

        let docRef = doc(db, this.authToken, 'noteIndex')
        setDoc(docRef, { index: this.noteIndex })

        docRef = doc(db, this.authToken, newName)
        setDoc(docRef, { note: this.activeDocumentContent })
      }
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
    const app = initializeApp(this.firebaseConfig)
    const db = getFirestore(app)
    let docRef = doc(db, this.authToken, 'noteIndex')
    let docSnap = await getDoc(docRef)
    console.log(docSnap.data())

    if (docSnap.exists()) {
      this.noteIndex = docSnap.data().index
    } else {
      console.log('No noteIndex document! Creating one plus a default note.')
      setDoc(docRef, { index: ['Welcome!'] })
      this.noteIndex = ['Welcome!']
      docRef = doc(db, this.authToken, 'Welcome!')
      setDoc(docRef, { note: 'This is your first note! Have fun!' })
      this.activeDocumentContent = 'This is your first note! Have fun!'
    }
  }
}
</script>

<template>
  <div v-if="!isLoggedIn" class="position-absolute top-50 start-50 translate-middle">
    <div class="authContainer">
      <passage-auth :app-id="appId"></passage-auth>
    </div>
  </div>

  <div v-else>
    <div class="position-absolute top-0 start-0" style="width: 25vb">
      <ul class="list-group">
        <li class="list-group-item" v-for="item in noteIndex" style="display: relative">
          <button class="btn" @click="getDocument(item)">{{ item }}</button>
          <button
            class="btn btn-outline-success btnToShowOnHoverOfItem"
            @click="renameDocument(item)"
            style="position: absolute; right: 60px; top: 50%; transform: translateY(-50%)"
          >
            <i class="bi bi-input-cursor-text"></i>
          </button>
          <buttton
            class="btn btn-outline-danger btnToShowOnHoverOfItem"
            @click="deleteDocument(item)"
            style="position: absolute; right: 10px; top: 50%; transform: translateY(-50%)"
            ><i class="bi bi-trash3"></i
          ></buttton>
        </li>
      </ul>
    </div>

    <div class="position-absolute top-50 end-0 translate-middle-y">
      <textarea
        class="form-control"
        v-model="activeDocumentContent"
        style="width: 1000px; height: 500px"
      ></textarea>
    </div>
    <div class="position-absolute top-0 end-0">
      <button class="btn btn-dark" @click="saveActiveDocument()">Save</button>
      <button class="btn btn-outline-success" @click="createNewDocument()">
        <i class="bi bi-plus-circle"></i>
      </button>
    </div>

    <!--Error Modal-->
    <div class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Error</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <p>{{ this.errorMessage }}</p>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-danger">Dismiss</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
