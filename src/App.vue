<template>
  <div id="app">
    <Notebook @change-page="changePage" @new-page="newPage" :pages="pages" :activePage="index" />
    <Page @save-page="savePage" @delete-page="deletePage" :page="pages[index]" />
  </div>
</template>

<script>
import Notebook from './components/Notebook'
import Page from './components/Page'
import Firebase from 'firebase'

let database = Firebase.initializeApp({
  apiKey: "AIzaSyB-Jc34qCTT3ohtEH0awR5PG59MJE08ySs",
  authDomain: "notebook-295ae.firebaseapp.com",
  databaseURL: "https://notebook-295ae.firebaseio.com",
  projectId: "notebook-295ae",
  storageBucket: "",
  messagingSenderId: "452619893152"
}).database().ref();

export default {
  name: 'App',
  components: {
    Notebook,
    Page
  },
  data: () => ({
    pages: [],
    index: 0
  }),
  mounted () {
    database.once('value', (pages) => {
      pages.forEach((page) => {
        this.pages.push({
          ref: page.ref,
          title: page.child('title').val(),
          content: page.child('content').val()
        })
      })
    })
  },
  methods: {
    newPage () {
      this.pages = [
        ...this.pages, {
          title: '',
          content: ''
        }
      ]
      this.index = this.pages.length - 1
    },
    changePage (index) {
      this.index = index
    },
    savePage () {
      const page = this.pages[this.index]
      if (page.ref) {
        this.updateExistingPage(page)
      } else {
        this.insertNewPage(page)
      }
    },
    updateExistingPage (page) {
      page.ref.update({
        title: page.title,
        content: page.content
      })
    },
    insertNewPage (page) {
      page.ref = database.push(page)
    },
    deletePage () {
      const ref = this.pages[this.index].ref
      ref && ref.remove()
      this.pages = this.pages.filter((page, i) => i !== this.index)
      this.index = Math.max(this.index - 1, 0)
    }
  }
}
</script>

<style>
  html, body, #app {
    height: 100%;
  }

  body {
    margin: 0;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    display: flex;
    flex-direction: row;
  }
</style>
