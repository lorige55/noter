<!-- eslint-disable no-undef -->
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
      noteIndex: [],
      shortenedNoteIndex: ['Loading...'],
      keyIndex: [],
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
      activeSavingProcesses: 0,
      secretKey: '',
      trust: true,
      secretKeyStatus: 'locked'
    }
  },
  methods: {
    async getDocument(item) {
      if (this.activeDocument !== '') {
        this.saveActiveDocument()
        this.activeSavingProcesses = 0
      }
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      this.activeDocument = this.noteIndex[this.shortenedNoteIndex.indexOf(item)]
      localStorage.setItem('lastNote', this.encryptString(this.activeDocument))
      let docRef = doc(db, this.authToken, this.keyIndex[this.shortenedNoteIndex.indexOf(item)])
      let docSnap = await getDoc(docRef)
      this.activeDocumentContent = this.decryptString(docSnap.data().note)
    },
    saveActiveDocument() {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(
        db,
        this.authToken,
        this.keyIndex[this.noteIndex.indexOf(this.activeDocument)]
      )
      setDoc(docRef, {
        note: this.encryptString(this.activeDocumentContent),
        title: this.encryptString(this.activeDocument)
      })
    },
    autoSave() {
      this.activeSavingProcesses++
      setTimeout(() => {
        if (this.activeSavingProcesses == 1) {
          this.saveActiveDocument()
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
          await deleteDoc(doc(db, this.authToken, this.keyIndex[indexToRemove]))
          this.keyIndex.splice(indexToRemove, 1)
          let docRef = doc(db, this.authToken, 'keyIndex')
          setDoc(docRef, { index: this.keyIndex })
        }
      }
    },
    createNewDocument() {
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      const newNoteName = prompt('Enter a name for your new note:', 'For Example: "Banana"')

      this.noteIndex.push(newNoteName)
      this.shortenNoteIndex()

      let key = this.generateKey(128)
      this.keyIndex.push(key)

      let docRef = doc(db, this.authToken, 'keyIndex')
      setDoc(docRef, { index: this.keyIndex })

      docRef = doc(db, this.authToken, key)
      setDoc(docRef, {
        note: this.encryptString('This is a note.'),
        title: this.encryptString(newNoteName)
      })

      this.activeDocumentContent = 'This is a note.'
      this.activeDocument = newNoteName
      localStorage.setItem('lastNote', this.activeDocument)
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
          await deleteDoc(
            doc(db, this.authToken, this.encryptString(this.noteIndex[indexToRemove]))
          )

          this.noteIndex.splice(indexToRemove, 1, newName)
          this.shortenNoteIndex()

          let docRef = doc(db, this.authToken, 'noteIndex')
          setDoc(docRef, { index: this.encryptArray(this.noteIndex) })

          docRef = doc(db, this.authToken, newName)
          setDoc(docRef, { note: this.encryptString(this.activeDocumentContent) })
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
    },
    encryptString(string) {
      string = string.toString()
      let encrypted = CryptoJS.Rabbit.encrypt(string, this.authToken).toString()
      return btoa(encrypted)
    },
    decryptString(string) {
      string = string.toString()
      string = atob(string)
      const bytes = CryptoJS.Rabbit.decrypt(string, this.authToken)
      return bytes.toString(CryptoJS.enc.Utf8)
    },
    generateKey(length) {
      let key = CryptoJS.lib.WordArray.random(length / 8)
      key.toString(CryptoJS.enc.Base64)
      return btoa(key)
    },
    changeSecretKey() {
      if (this.secretKeyStatus == 'locked') {
        this.secretKeyStatus = 'unlocked'
      } else {
        this.secretKeyStatus = 'locked'
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        let docRef = doc(db, this.authToken, 'secretKey')
        setDoc(docRef, { key: this.secretKey })
        localStorage.setItem('secretKey', this.secretKey)
      }
    },
    generateNewSecretKey() {
      this.secretKey = this.generateKey(256)
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.authToken, 'secretKey')
      setDoc(docRef, { key: this.secretKey })
      localStorage.setItem('secretKey', this.secretKey)
    }
  },
  async mounted() {
    if (
      localStorage.getItem('psg_auth_token') !== null &&
      localStorage.getItem('psg_last_login') !== null
    ) {
      this.isLoggedIn = true
      this.authToken = localStorage.getItem('psg_auth_token').split('.')[0]
    }

    const app = initializeApp(this.firebaseConfig)
    const db = getFirestore(app)

    let docRef = doc(db, this.authToken, 'secretKey')
    let docSnap = await getDoc(docRef)
    if (docSnap.exists()) {
      if (docSnap.data().key !== 'nah') {
        this.secretKey = docSnap.data().key
      } else {
        this.trust = false
        if (localStorage.getItem('secretKey') !== null) {
          this.secretKey = localStorage.getItem('secretKey')
        } else {
          this.$refs.securityModal.show()
        }
      }
    } else {
      let key = this.generateKey(256)
      docRef = doc(db, this.authToken, 'secretKey')
      setDoc(docRef, { key: key })
      this.secretKey = key
    }

    if (this.secretKey !== '') {
      docRef = doc(db, this.authToken, 'keyIndex')
      docSnap = await getDoc(docRef)
      if (docSnap.exists() && docSnap.data().index[0].length > 0) {
        this.keyIndex = docSnap.data().index
        for (let i = 0; i < this.keyIndex.length; i++) {
          let docRef = doc(db, this.authToken, this.keyIndex[i])
          let docSnap = await getDoc(docRef)
          this.noteIndex[i] = this.decryptString(docSnap.data().title)
        }
        this.shortenNoteIndex()
        if (localStorage.getItem('lastNote') !== null) {
          this.getDocument(this.decryptString(localStorage.getItem('lastNote')))
        }
      } else {
        let key = this.generateKey(128)
        this.keyIndex = [key]
        setDoc(docRef, { index: this.keyIndex })
        this.noteIndex = ['Welcome!']
        this.shortenNoteIndex()
        docRef = doc(db, this.authToken, key)
        setDoc(docRef, {
          note: this.encryptString('This is your first note! Have fun!'),
          title: this.encryptString('Welcome!')
        })
        this.activeDocument = 'Welcome!'
        this.activeDocumentContent = 'This is your first note! Have fun!'
      }
    }

    this.errorModal = new bootstrap.Modal(this.$refs.errorModal)
    this.conformationModal = new bootstrap.Modal(this.$refs.conformationModal)
    this.securityModal = new bootstrap.Modal(this.$refs.securityModal)
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
          <button class="btn btn-outline-success" @click="createNewDocument()">
            <i class="bi bi-plus-circle"></i>
          </button>
          <button class="btn btn-outline-dark" @click="securityModal.show()">
            <i class="bi bi-shield-lock-fill"></i>
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

    <!--Secret Key Modal-->
    <div class="modal" tabindex="-1" ref="securityModal">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Privacy & Security</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <p>
              To ensure privacy, Notes are encrypted using
              <a
                href="https://en.wikipedia.org/wiki/Advanced_Encryption_Standard"
                class="link-dark link-offset-2 link-underline-opacity-25 link-underline-opacity-100-hover"
                >AES-256</a
              >
              before saving to the cloud. The encryption key is crucial for data security and is
              stored by default in the cloud. For full privacy, store the key yourself, but remember
              you'll need to enter it every time you switch devices or browsers. Losing the key
              means losing all your data permanently.
            </p>
            <div v-if="trust">
              <p>This is your Secret Key:</p>
              <input
                class="form-control"
                type="text"
                v-model="secretKey"
                aria-label="Secret Key"
                disabled
                readonly
              />
            </div>
            <div v-else>
              <p>Please enter your Secret Key:</p>
              <div class="input-group mb-3">
                <button
                  @click="changeSecretKey()"
                  class="btn btn-outline-dark"
                  type="button"
                  id="lockButton"
                >
                  <div v-if="secretKeyStatus == 'locked'"><i class="bi bi-lock-fill"></i></div>
                  <div v-else><i class="bi bi-unlock-fill"></i></div>
                </button>
                <input
                  class="form-control"
                  type="text"
                  :value="secretKey"
                  aria-label="Secret Key"
                  :disabled="secretKeyStatus === 'locked'"
                  :readonly="secretKeyStatus === 'locked'"
                />
                <button class="btn btn-outline-danger" @click="generateNewSecretKey()">
                  Generate
                </button>
              </div>
              <p>
                Note: We strongly recommend to use a randomly generated Secret Key from us. However,
                if you want, you could edit the generated key or even set your completly own one.
              </p>
              <p class="text-danger">If you loose your Secret Key, all your data is lost!</p>
            </div>
            <br />
            <div class="form-check form-switch">
              <input
                class="form-check-input"
                type="checkbox"
                role="switch"
                id="flexSwitchCheckChecked"
                v-model="trust"
              />
              <label class="form-check-label" for="flexSwitchCheckChecked"
                >Store Secret Key in Cloud</label
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
