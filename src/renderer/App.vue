<template>
    <div id="app">
        <LandingPage @change-page="changePage" @new-page="newPage" :pages="pages" :activePage="index"/>
        <Page @delete-page="deletePage"
              :page="pages[index]"/>
    </div>
</template>

<script>
  import LandingPage from './components/LandingPage'
  import Page from './components/Page'

  var exec = require('child_process').exec

  const remote = require('electron').remote
  const app = remote.app
  const home = app.getPath('home')
  const appProjPath = home + '/Drupal8Projects/'
  const fs = require('fs')

  const low = require('lowdb')
  const FileAsync = require('lowdb/adapters/FileSync')
  const adapter = new FileAsync(appProjPath + 'db.json')
  const db = low(adapter)
  //
  db.defaults({
    pages: []
  }).write()

  export default {
    name: 'drupal-setup',
    components: {
      LandingPage,
      Page
    },
    data () {
      return {
        pages: [],
        index: 0
      }
    },
    mounted () {
      this.pages = this.readInitStorage()
    },
    methods: {
      readInitStorage () {
        var data = fs.readFileSync(appProjPath + 'db.json')
        var storageProjects = JSON.parse(data)
        return storageProjects.pages
      },
      newPage () {
        this.pages.push({
          title: '',
          drupalComposerMessage: '',
          composeGetMessage: '',
          composerInstallMessage: '',
          landoInitMessage: '',
          edit: '',
          projectStartMessage: '',
          projectInfoMessage: '',
          formErrors: [],
          projectCredentials: ''
        })
        this.index = this.pages.length - 1
      },
      changePage (index) {
        this.index = index
      },
      deletePage () {
        var self = this
        var projName = this.pages[this.index].title
        var projPath = appProjPath + projName
        var deleteThis = new Promise(function (resolve, reject) {
          exec('chmod -R 777 ' + projPath + ' && rm -rf ' + appProjPath + self.pages[self.index].title, (error, stdout, stderr) => {
            if (error) {
              console.error(`exec error: ${error}`)
            }
            var data = 'drupal clonedrrr'
            resolve(data)
          })
        })
        deleteThis.then(function (test) {
          db.get('pages')
            .remove(self.pages[self.index])
            .write()
          self.pages.splice(self.index, 1)
          self.index = Math.max(self.index - 1, 0)
        })
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
        height: 100%;
        width: 100%;
    }
</style>
