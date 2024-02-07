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
      shortenedNoteIndex: ['Loading...'],
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
      errorMessage: 'An Error has occured. Please try again or create an Issue on GitHub.',
      conformationMessage: '',
      itemToDelete: '',
      activeSavingProcesses: 0
    }
  },
  methods: {
    async getDocument(item) {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      this.activeDocument = this.noteIndex[this.shortenedNoteIndex.indexOf(item)]
      let docRef = doc(db, this.authToken, this.activeDocument)
      let docSnap = await getDoc(docRef)
      this.activeDocumentContent = docSnap.data().note
    },
    saveActiveDocument() {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.authToken, this.activeDocument)
      setDoc(docRef, { note: this.activeDocumentContent })
      console.log('Document saved.')
    },
    autoSave() {
      this.activeSavingProcesses++
      console.log(this.activeSavingProcesses)
      setTimeout(() => {
        console.log(this.activeSavingProcesses)
        if (this.activeSavingProcesses == 1) {
          this.saveActiveDocument()
          console.log('saved')
        }
        this.activeSavingProcesses--
      }, 3000)
    },
    async deleteDocument(item, attempt) {
      if (attempt == 1) {
        this.conformationMessage =
          'Are you sure you want to delete "' +
          this.noteIndex[this.shortenedNoteIndex.indexOf(item)] +
          '"? This action cannot be undone.'

        this.conformationModal.show()

        this.itemToDelete = item
      } else if (attempt == 2) {
        item = this.itemToDelete
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        const indexToRemove = this.shortenedNoteIndex.indexOf(item)

        if (indexToRemove !== -1) {
          this.noteIndex.splice(indexToRemove, 1)
          this.shortenNoteIndex()

          let docRef = doc(db, this.authToken, 'noteIndex')
          setDoc(docRef, { index: this.noteIndex })
        }

        await deleteDoc(doc(db, this.authToken, item))
        console.log('Document deleted.')
      }
    },
    createNewDocument() {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const newNoteName = prompt('Enter a name for your new note:')

      this.noteIndex.push(newNoteName)
      this.shortenNoteIndex()

      let docRef = doc(db, this.authToken, 'noteIndex')
      setDoc(docRef, { index: this.noteIndex })

      docRef = doc(db, this.authToken, newNoteName)
      setDoc(docRef, { note: 'This is a note.' })

      this.activeDocumentContent = 'This is a note.'
      this.activeDocument = newNoteName
    },
    async renameDocument(item) {
      let newName = prompt('Enter a new name for this note:', item)

      if (newName.length < 1) {
        this.errorMessage = 'You cannot name a note nothing!'

        this.errorModal.show()
      } else {
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)

        const indexToRemove = this.shortenedNoteIndex.indexOf(item)

        if (indexToRemove !== -1) {
          await deleteDoc(doc(db, this.authToken, this.noteIndex[indexToRemove]))
          console.log('Document deleted.')

          this.noteIndex.splice(indexToRemove, 1, newName)
          this.shortenNoteIndex()

          let docRef = doc(db, this.authToken, 'noteIndex')
          setDoc(docRef, { index: this.noteIndex })

          docRef = doc(db, this.authToken, newName)
          setDoc(docRef, { note: this.activeDocumentContent })
        }
      }
    },
    shortenNoteIndex() {
      let charLimit = 30
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

    if (docSnap.exists() && docSnap.data().index[0].length > 0) {
      this.noteIndex = docSnap.data().index
      this.shortenNoteIndex()
    } else {
      console.log('No noteIndex document! Creating one plus a default note.')
      setDoc(docRef, { index: ['Welcome!'] })
      this.noteIndex = ['Welcome!']
      this.shortenNoteIndex()
      docRef = doc(db, this.authToken, 'Welcome!')
      setDoc(docRef, { note: 'This is your first note! Have fun!' })
      this.activeDocumentContent = 'This is your first note! Have fun!'
    }
    this.errorModal = new bootstrap.Modal(this.$refs.errorModal)

    this.conformationModal = new bootstrap.Modal(this.$refs.conformationModal)
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
    <nav class="navbar navbar-expand-lg bg-body-tertiary text-light">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Noter</a>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <button class="btn btn-dark" @click="saveActiveDocument()">Save</button>
          <button class="btn btn-outline-success" @click="createNewDocument()">
            <i class="bi bi-plus-circle"></i>
          </button>
        </div>
      </div>
    </nav>

    <div class="container-fluid mt-3">
      <div class="row">
        <!-- Left Side: List of Cards -->
        <div class="col-md-3">
          <ul class="list-group">
            <li v-for="item in shortenedNoteIndex" :key="item" class="list-group-item">
              <div class="row">
                <div class="col-9">
                  <button
                    class="btn"
                    :class="{
                      bold: noteIndex[shortenedNoteIndex.indexOf(item)] === activeDocument
                    }"
                    @click="getDocument(item)"
                  >
                    {{ item }}
                  </button>
                </div>
                <div class="col-3 text-right">
                  <div class="btn-group ms-auto">
                    <button
                      type="button"
                      class="btn"
                      data-bs-toggle="dropdown"
                      aria-expanded="false"
                    >
                      <i class="bi bi-three-dots"></i>
                    </button>
                    <ul class="dropdown-menu">
                      <li @click="renameDocument(item)" class="dropdown-item">
                        <a>Rename Note</a>
                      </li>
                      <li @click="deleteDocument(item, 1)" class="dropdown-item">
                        <a>Delete Note</a>
                      </li>
                    </ul>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </div>

        <!-- Right Side: TextArea -->
        <div class="col-md-9">
          <div class="form-group">
            <textarea
              @input="autoSave()"
              class="form-control"
              v-model="activeDocumentContent"
              rows="10"
            ></textarea>
          </div>
        </div>
      </div>
    </div>

    <!--Error Modal-->
    <div ref="errorModal" class="modal" id="errorModal" tabindex="-1">
      <div class="modal-dialog modal-dialog-centered">
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
            <p>{{ errorMessage }}</p>
          </div>
          <div class="modal-footer">
            <button data-bs-dismiss="modal" type="button" class="btn btn-danger">Dismiss</button>
          </div>
        </div>
      </div>
    </div>

    <!--Conformation Modal-->
    <div id="conformationModal" ref="conformationModal" class="modal" tabindex="-1">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Are you sure?</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <p>
              {{ conformationMessage }}
            </p>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-success" data-bs-dismiss="modal">Cancel</button>
            <button
              type="button"
              class="btn btn-danger"
              data-bs-dismiss="modal"
              @click="deleteDocument('thisStringDoesntMatter', 2)"
            >
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
