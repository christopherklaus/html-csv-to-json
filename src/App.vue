<template>
  <div id="app" class="center">
    <div class="import-data">
      <div class="import-from-csv">
        <div class="import-from-url">
          <input type="text" v-model="csvURL" placeholder="https://example.com/file.csv">
          <button @click="importCSVFromUrl">Import File from URL</button>
        </div>
        <div class="import-from-file">

        </div>
      </div>
      <div class="import-from-html">
        <textarea v-model="tableData">
        </textarea>
        <div class="button-wrapper">
          <button @click="convertHTML">Convert to JSON</button>
        </div>
      </div>
    </div>
    <samp>
      <pre>{{ jsonData }}</pre>
    </samp>
    <h2>Rename fields</h2>
    <ul v-if="jsonData.length">
      <li
        v-for="field in jsonDataFields"
        :key="field"
        @dblclick="renameFieldSource = field; renameFieldTarget = field">
        <div class="field-name" v-if="renameFieldSource !== field">{{ field }}</div>
        <input type="text" v-model="renameFieldTarget" @keydown.enter.stop="renameField(renameFieldSource, renameFieldTarget)" v-else>
      </li>
    </ul>
  </div>
</template>

<script>

import HTMLFromTable from 'html-table-to-json'
import { mapKeys } from 'lodash'

export default {
  name: 'App',
  data() {
    return {
      rename: false,
      renameFieldSource: '',
      renameFieldTarget: '',
      csvURL: '',
      csvData: undefined,
      tableData: `
      <table>
        <tr>
          <th>Animal</th>
          <th>Color</th>
          <th>Name</th>
        </tr>
        <tr>
          <td>Unicorn</td>
          <td>Pink</td>
          <td>Billy</td>
        </tr>
        <tr>
          <td>Walrus</td>
          <td>Orange</td>
          <td>Sue</td>
        </tr>
      </table>`,
      jsonData: []
    }
  },
  components: {
  },
  computed: {
    jsonDataFields() {
      return Object.keys(this.jsonData[0]).map((key) => key)
    }
  },
  methods: {
    triggerInputChangeFor(element, event) {
      console.log({ element, event })
    },
    importCSVFromUrl() {

    },
    convertCSV() {

    },
    convertHTML() {
      this.jsonData = (HTMLFromTable.parse(this.tableData) || {}).results[0]
    },
    renameField(fieldName, newFieldName) {
      this.jsonData = this.jsonData.map(element => mapKeys(element, (value, key) => key === fieldName ? newFieldName : key))
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.center {
  width: 50%;
  margin: 0 auto;
}

textarea {
  border: 1px solid #ccc;
  padding: 1em;
  height: 300px;
  display: block;
  width: 100%
}

pre {
  text-align: left
}
</style>
