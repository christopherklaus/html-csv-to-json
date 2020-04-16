<template>
  <div id="app" class="container">
    <div
      class="success-overlay"
      v-if="successfullySubmitted">
      <div class="container">
        <div class="row">
          <div class="col">
            <div class="alert alert-success text-center w-50 p-5 mx-auto mt-5">
              <h4>Thanks for sharing 🎉</h4>
              <p>Your data was uploaded successfully and will help improving response to the desease!</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <div class="import-data card mb-3">
          <div class="input-group">
            <div class="input-group-prepend">
              <div class="input-group-text">Upload from</div>
              <button
                class="btn btn-secondary"
                :class="{active: selectedInput === 'csv'}"
                @click="selectedInput = 'csv'"
                type="button">CSV</button>
            </div>
            <div class="input-group-append">
              <button
                class="btn btn-secondary"
                :class="{active: selectedInput === 'html'}"
                @click="selectedInput = 'html'"
                type="button">HTML Table</button>
            </div>
          </div>
          <div class="selection-wrapper mt-3">
            <div
              class="import-from-csv"
              v-if="selectedInput === 'csv'">
              <div class="import-from-file">
                <h6>Import from File from CSV</h6>
                <div class="input-group">
                  <div class="custom-file">
                    <input
                      type="file"
                      class="custom-file-input"
                      id="inputGroupFile01"
                      @change="uploadFile"
                      accept=".csv">
                    <label
                      class="custom-file-label"
                      for="inputGroupFile01">Choose file</label>
                  </div>
                </div>
              </div>
            </div>
            <div
              v-else
              class="import-html">
              <div class="import-from-html-url">
                <h6>Import from HTML URL</h6>
                <div class="import-from-url">
                  <div class="input-group">
                    <input
                      type="text"
                      v-model="tableURL"
                      class="form-control"
                      placeholder="https://example.com/siteWithDataTable">
                    <div class="input-group-append">
                      <button class="btn btn-secondary" @click="importTableFromURL">Import first table from URL</button>
                    </div>
                  </div>
                </div>
              </div>
              <div class="import-from-html mt-3">
                <h6>Import from HTML</h6>
                <textarea class="form-control mb-3" v-model="tableData"></textarea>
                <button class="btn btn-secondary" @click="convertHTML">Process Data</button>
              </div>
            </div>
          </div>
        </div>
        <div class="card my-4">
          <h5>Upload new data for location</h5>
          <div class="input-group mt-2">
            <div class="input-group-prepend">
              <label class="input-group-text">Available Locations</label>
            </div>
            <select
              class="custom-select"
              v-model="selectedLocation">
              <option selected disabled>Choose Location to upload data for ...</option>
              <option value="create">Create New Location</option>
              <option
                v-for="location in availableLocations"
                :value="location.id"
                :key="location.id">{{ location.province }} in {{ location.country }}</option>
            </select>
          </div>
          <div
            class="create-location-wrapper"
            v-if="selectedLocation === 'create'">
            <h5 class="mt-4 mb-0" >Create new location</h5>
            <p class="text-muted mt-1">In case you cannot find the location you want to upload data for, please create a new location.</p>

            <label for="country">Country</label>
            <input type="text" class="form-control mb-3" id="country" v-model="newLocation.country">
            <label for="country-code">Country Code</label>
            <input type="text" class="form-control mb-3" id="country-code" v-model="newLocation.country_code">
            <label for="county">County</label>
            <input type="text" class="form-control mb-3" id="county" v-model="newLocation.county">
            <label for="province">Province</label>
            <input type="text" class="form-control" id="province" v-model="newLocation.province">
            <button
              class="btn btn-success mt-3"
              @click="createLocation">Create Location</button>
          </div>
        </div>
        <div class="card mb-3 mt-5" v-if="readyToEditJsonData">
          <h5>Normalize Data</h5>
          <h6>Expected output structure</h6>
          <div class="result-data-wrapper d-flex justify-content-between align-items-center">
            <div
              class="badge badge-pill badge-secondary"
              v-for="field in requiredFields"
              :key="field.name"
              :class="{ 'badge-success': jsonDataFields.includes(field.name) }"
              @dragstart.stop="draggedField = field.name"
              draggable>{{ field.name }} ({{ field.type }})</div>
          </div>
          <h6 class="mt-4 mb-0">Your uploaded data for {{ availableLocations[selectedLocation] }}</h6>
          <p class="text-muted">Only one entry is shown</p>
          <div class="wrapper">
            <button
              @click="addStaticField = true"
              class="btn btn-secondary btn-sm mb-1"><font-awesome-icon icon="plus" /> Add static field</button>
            <p class="text-muted">This might be helpful for creating a static value for e.g. date.</p>
            <div
              class="input-group input-group-sm mb-3"
              v-if="addStaticField">
              <div class="input-group-prepend">
                <div class="input-group-text">Field Name</div>
              </div>
              <input
                v-model="newFieldName"
                type="text"
                placeholder="location"
                class="form-control">
              <div class="input-group-append">
                <div class="input-group-text">Value</div>
              </div>
              <input
                v-model="newFieldValue"
                type="text"
                placeholder="Berlin"
                class="form-control">
              <div class="input-group-append">
                <button
                  class="btn btn-success"
                  @click="addField">Create</button>
              </div>
            </div>
          </div>
          <div class="result-data-wrapper">
            <table class="table mt-2">
              <tr>
                <th
                  v-for="field in jsonDataFields"
                  @click="renameFieldSource = field; renameFieldTarget = field"
                  @drop.stop="renameField(field, draggedField)"
                  @dragover.prevent
                  :droppable="true"
                  :key="field">
                  <div
                    class="field-name d-flex justify-content-start align-items-center"
                    v-if="renameFieldSource !== field">
                    {{ field }}
                    <button
                      class="ml-2 color-red"
                      @click.stop="deleteField(field)"><font-awesome-icon icon="trash" /></button>
                  </div>
                  <input
                    v-else
                    class="form-control"
                    type="text"
                    v-model="renameFieldTarget"
                    @keydown.esc.stop="renameFieldSource = ''; renameFieldTarget = ''"
                    @keydown.enter.stop="renameField(renameFieldSource, renameFieldTarget)">
                </th>
              </tr>
              <tr>
                <td
                  v-for="field in jsonDataFields"
                  :key="field">
                  {{ jsonData[0][field] }}
                  <p class="text-muted mt-1 mb-0">({{ typeof jsonData[0][field] }})</p>
                </td>
              </tr>
            </table>
          </div>
        </div>
        <div class="upload-button-wrapper mt-3 mb-5" v-if="readyToEditJsonData">
          <button
            class="btn w-100"
            @click="submitEntry"
            :class="{ 'btn-success': allFieldsMatching, 'btn-secondary': !allFieldsMatching }"
            :disabled="!allFieldsMatching">Add entry to database</button>
        </div>
        <div class="card" v-if="jsonData.length">
          <button
            class="btn btn-secondary"
            @click="showJSON = !showJSON">Show JSON Preview</button>
          <div class="alert alert-secondary show-json-wrapper mb-0 mt-3" v-if="showJSON">
            <h6>JSON Preview</h6>
            <samp>
              <pre>{{ jsonData }}</pre>
            </samp>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import HTMLFromTable from 'html-table-to-json'
import PapaParse from 'papaparse'
import { mapKeys, omit, assignIn, isEqual, sortBy } from 'lodash'
import { get, post } from 'axios'

export default {
  name: 'App',
  data() {
    return {
      draggedField: undefined,
      newLocation: {
        country_code: '',
        country: '',
        county: '',
        province: ''
      },
      requiredFields: [
        {
          name: 'occured_on',
          type: 'date'
        },
        {
          name: 'tested',
          type: 'integer'
        },
        {
          name: 'confirmed',
          type: 'integer'
        },
        {
          name: 'recovered',
          type: 'integer'
        },
        {
          name: 'deceased',
          type: 'integer'
        }
      ],
      newFieldName: '',
      newFieldValue: '',
      showJSON: false,
      rename: false,
      addStaticField: false,
      renameFieldSource: '',
      renameFieldTarget: '',
      tableURL: '',
      selectedInput: 'csv',
      csvData: undefined,
      tableData: '',
      jsonData: [],
      error: undefined,
      selectedLocation: undefined,
      availableLocations: [],
      showCreateNewLocation: false,
      successfullySubmitted: false
    }
  },
  computed: {
    jsonDataFields() {
      return Object.keys(this.jsonData[0]).map((key) => key)
    },
    readyToEditJsonData() {
      return this.jsonData.length && this.selectedLocation && this.selectedLocation !== 'create'
    },
    allFieldsMatching() {
      const requiredFieldsHeader = this.requiredFields.map(({ name }) => name)
      return isEqual(sortBy(this.jsonDataFields), sortBy(requiredFieldsHeader))
    }
  },
  mounted() {
    this.getLocations()
  },
  methods: {
    submitEntry() {
      post('https://api.test.ix-io.net/covid/covid/cases', {
        cases: this.jsonData,
        location_id: this.selectedLocation
      }).then(() => {
        this.successfullySubmitted = true
      }).catch((error) => {
        console.error(error)
      })
    },
    importTableFromURL() {
      get(`http://a05b81e417a6b11eaa621060268a248d-1111261182.eu-west-1.elb.amazonaws.com/table?url=${this.tableURL}`).then((response) => {
        this.tableData = response.data.tags[0]
      })
    },
    getLocations() {
      return get('https://api.test.ix-io.net/covid/covid/locations').then((response) => {
        this.availableLocations = response.data.locations
      })
    },
    uploadFile(event) {
      const files = event.target.files || event.dataTransfer.files;
      if (!files.length)
        return

      const reader = new FileReader()
      reader.onload = (event) => {
        this.csvData = event.target.result
      }
      reader.readAsText(files[0])
      setTimeout(() => {
        this.convertCSV()
      }, 500)
    },
    convertCSV() {
      this.jsonData = PapaParse.parse(this.csvData, { header: true }).data
    },
    createLocation() {
      post('https://api.test.ix-io.net/covid/covid/locations', {
        location: this.newLocation
      }).then(() => {
        this.getLocations().then(() => {
          this.selectedLocation = (this.availableLocations.find(({ province }) => province === this.newLocation.province) || {}).id
        })
      })
    },
    convertHTML() {
      this.jsonData = (HTMLFromTable.parse(this.tableData) || {}).results[0]
    },
    addField() {
      const newElement = {}
      newElement[this.newFieldName] = this.newFieldValue
      this.jsonData = this.jsonData.map(element => assignIn(element, newElement))

      this.newFieldValue = ''
      this.newFieldName = ''
    },
    renameField(fieldName, newFieldName) {
      this.jsonData = this.jsonData.map(element => mapKeys(element, (value, key) => key === fieldName ? newFieldName : key))
    },
    deleteField(fieldName) {
      this.jsonData = this.jsonData.map(element => omit(element, [fieldName]))
    }
  }
}
</script>

<style>
.success-overlay {
  position: fixed;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, .7);
  z-index: 1000
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  margin-top: 60px;
}

body {
  background-color: #f8f9fa
}

.card {
  box-shadow: 0 1px 1px rgba(0, 0, 0, .04), 0 4px 5px rgba(0, 0, 0, .02), 0 7px 9px rgba(0, 0, 0, .04), 0 25px 30px -15px rgba(0, 0, 0, .08), 0 15px 18px -30px rgba(0, 0, 0, .04);
  padding: 1em;
  border-radius: 15px
}

table th {
  min-width: 150px;
  margin: 0 5px;
}

.result-data-wrapper {
  min-width: 100%;
  overflow: auto
}

.field-name {
  cursor: text
}

button.color-red {
  color: #cc0000;
  border: none;
  background: transparent
}

textarea {
  border: 1px solid #ccc;
  padding: 1em;
  height: 300px;
  display: block;
  width: 100%
}

ul {
  list-style-type: none;
  padding: 0
}

.card {
  z-index: 100
}
</style>
