<!-- eslint-disable no-undef -->
<script>
import '@passageidentity/passage-elements/passage-auth'
import { PassageUser } from '@passageidentity/passage-auth/passage-user'
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, getDoc, setDoc, deleteDoc } from 'firebase/firestore'

export default {
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
      this.activeDocument = this.noteIndex[this.shortenedNoteIndex.indexOf(item)]
      localStorage.setItem('lastNote', this.activeDocument)
      //get document from firebase and set activeDocumentContent to content
      let docRef = doc(db, this.userId, this.keyIndex[this.shortenedNoteIndex.indexOf(item)])
      let docSnap = await getDoc(docRef)
      this.activeDocumentContent = this.decryptString(docSnap.data().note)
      this.$refs.editor.textContent = this.activeDocumentContent
    },
    saveActiveDocument() {
      //initialize firebase
      const app = initializeApp(this.firebaseConfig)
      const db = getFirestore(app)
      let docRef = doc(db, this.userId, this.keyIndex[this.noteIndex.indexOf(this.activeDocument)])
      //save document to firebase
      setDoc(docRef, {
        note: this.encryptString(this.activeDocumentContent),
        title: this.encryptString(this.activeDocument)
      })
    },
    autoSave() {
      //bind input to activeDocumentContent
      this.activeDocumentContent = this.$refs.editor.textContent
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
    createNewDocument(attempt) {
      //show input for new note name
      if (attempt == 1) {
        this.creatingNewDocument = true
      } else if (attempt == 2) {
        //initialize firebase
        const app = initializeApp(this.firebaseConfig)
        const db = getFirestore(app)
        //get new note name and push it to noteIndex, then shorten it
        const newNoteName = document.getElementById('noteNameInput').value
        this.noteIndex.push(newNoteName)
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
          note: this.encryptString('This is a note.'),
          title: this.encryptString(newNoteName)
        })
        //set activeDocument and lastNote to new note
        this.activeDocumentContent = 'This is a note.'
        this.$refs.editor.textContent = this.activeDocumentContent
        this.activeDocument = newNoteName
        localStorage.setItem('lastNote', this.activeDocument)
        this.creatingNewDocument = false
      }
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
    },
    changeEmail() {
      //change email
      this.user.changeEmail(document.getElementById('emailInput').value)
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
      this.errorModal.show()
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
        if (localStorage.getItem('lastNote') == null) {
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
        this.shortenNoteIndex()
        docRef = doc(db, this.userId, key)
        setDoc(docRef, {
          note: this.encryptString('This is your first note! Have fun!'),
          title: this.encryptString('Welcome!')
        })
        //set default note to active Note
        this.activeDocument = 'Welcome!'
        this.activeDocumentContent = 'This is your first note! Have fun!'
        this.$refs.editor.textContent = this.activeDocumentContent
      }
    }
    //initialize modals
    this.errorModal = new bootstrap.Modal(this.$refs.errorModal)
    this.conformationModal = new bootstrap.Modal(this.$refs.conformationModal)
    this.profileModal = new bootstrap.Modal(this.$refs.profileModal)
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
          <button class="btn btn-outline-success" @click="createNewDocument(1)">
            <i class="bi bi-plus-circle"></i>
          </button>
          <div class="dropdown">
            <button class="btn btn-outline-dark" data-bs-toggle="dropdown" aria-expanded="false">
              <i class="bi bi-person-circle"></i>
            </button>
            <ul class="dropdown-menu">
              <li><a class="dropdown-item" @click="profileModal.show()">My Profile</a></li>
              <li><a class="dropdown-item" @click="logout()">Logout</a></li>
            </ul>
          </div>
        </div>
      </div>
    </nav>

    <div class="container-fluid mt-3">
      <div class="row">
        <!-- Left Side: List of Cards -->
        <div class="col-md-3">
          <ul class="list-group">
            <li v-for="item in shortenedNoteIndex" :key="item" class="list-group-item w-100">
              <div v-if="documentToRename !== noteIndex[shortenedNoteIndex.indexOf(item)]">
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
                        <li @click="renameDocument(item, 1)" class="dropdown-item">
                          <a>Rename Note</a>
                        </li>
                        <li @click="deleteDocument(item, 1)" class="dropdown-item">
                          <a>Delete Note</a>
                        </li>
                      </ul>
                    </div>
                  </div>
                </div>
              </div>
              <div v-else>
                <div class="row">
                  <div class="col-9 w-100">
                    <div class="input-group w-100">
                      <input
                        type="text"
                        class="form-control w-80"
                        :placeholder="noteIndex[shortenedNoteIndex.indexOf(item)]"
                        aria-label="Change the name of your note"
                        aria-describedby="renameButton"
                        id="changeNoteNameInput"
                      />
                      <button
                        @click="renameDocument(item, 2)"
                        class="btn btn-outline-success"
                        type="button w-20"
                        id="renameButton"
                      >
                        Rename
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </li>
            <li class="list-group-item w-100" v-if="creatingNewDocument">
              <div class="col-9 w-100">
                <div class="input-group w-100">
                  <input
                    type="text"
                    class="form-control w-80"
                    placeholder="Enter a name for your new note"
                    aria-label="Enter a name for your new note"
                    aria-describedby="createButton"
                    id="noteNameInput"
                  />
                  <button
                    @click="createNewDocument(2)"
                    class="btn btn-outline-success"
                    type="button w-20"
                    id="createButton"
                  >
                    Create
                  </button>
                </div>
              </div>
            </li>
          </ul>
        </div>

        <!-- Right Side: TextArea -->
        <div class="col-md-9">
          <div class="form-group">
            <div
              id="editor"
              class="form-control"
              contenteditable="true"
              @input="autoSave()"
              ref="editor"
              style="height: 90vh"
            ></div>
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

    <!--My Profile Modal-->
    <div class="modal" tabindex="-1" ref="profileModal">
      <div class="modal-dialog modal-dialog-centered modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h4 class="modal-title">My Profile</h4>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <h5>Account</h5>
            <form>
              <label for="emailInput" class="form-label">Email address</label>
              <div class="input-group mb-3 w-50">
                <input
                  type="text"
                  class="form-control"
                  :value="user.email"
                  aria-label="Your Email Adress"
                  aria-describedby="saveNewEmail"
                  id="emailInput"
                />
                <button
                  class="btn btn-outline-danger"
                  type="button"
                  id="saveNewEmail"
                  @click="changeEmail()"
                >
                  Save
                </button>
              </div>
              <label for="userId" class="form-label">User ID:</label>
              <div class="mb-3 w-50">
                <input
                  type="text"
                  class="form-control"
                  :value="userId"
                  aria-label="Your User ID"
                  id="userId"
                  disabled
                />
              </div>
            </form>
            <h5>Privacy & Security</h5>
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
                @change="trustChanged()"
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
