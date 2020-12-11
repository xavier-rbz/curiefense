<template>
  <div class="card">
    <div class="card-content">
      <div class="content">
        <div class="columns">
          <div class="column is-5" style="border-right:solid 2px #f8f8f8; ">
            <div class="field">
              <label class="label is-small">
                Name
                <span class="has-text-grey is-pulled-right document-id" title="Flow control id">
                    {{ selectedDoc.id }}
                  </span>
              </label>
              <div class="control">
                <input class="input is-small document-name"
                       placeholder="Flow control name"
                       v-model="selectedDoc.name"/>
              </div>
            </div>
            <div class="field">
              <label class="checkbox is-size-7">
                <input type="checkbox"
                       class="document-active"
                       v-model="selectedDoc.active">
                Active
              </label>
            </div>
            <div class="field">
              <label class="label is-small has-text-left form-label">TTL</label>
              <div class="control">
                <input class="input is-small document-ttl" type="text" placeholder="New rate limit rule name"
                       v-model="selectedDoc.ttl">
              </div>
            </div>
            <div class="field">
              <label class="label is-small has-text-left form-label">Count by</label>
              <div class="group-key">
                <limit-option
                    v-for="(option, idx) in selectedDoc.key"
                    show-remove
                    @remove="removeKey(idx)"
                    @change="updateKeyOption"
                    :removable="selectedDoc.key.length > 1"
                    :index="idx"
                    :option="generateOption(option)"
                    :key="getOptionTextKey(option, idx)"/>
                <a title="Add new option rule"
                   class="is-text is-small is-size-7 ml-4 add-key-button"
                   @click="addKey()">
                  New entry
                </a>
                <br>
                <p class="has-text-danger pl-3 mt-3 is-size-7 key-invalid" v-if="!keysAreValid">
                  Count-by entries must be unique
                </p>
              </div>
            </div>
            <div class="field">
              <limit-action :action.sync="selectedDoc.action"/>
            </div>
            <div class="field">
              <label class="label is-small">Notes</label>
              <div class="control">
                <textarea class="is-small textarea document-notes" v-model="selectedDoc.notes" rows="2"></textarea>
              </div>
            </div>
            <div class="columns">
              <div class="column is-6 filter-column" v-for="filter in filters" :key="filter"
                   :class="filter + '-filter-column'">
                <p class="title is-7 is-uppercase">{{ titles[filter] }}</p>
                <hr :style="barStyle[filter]"/>
                <table class="table is-narrow is-fullwidth">
                  <tbody>
                  <tr v-for="(tag, tagIndex) in selectedDoc[filter]" :key="tagIndex">
                    <td class="tag-cell"
                        :class=" duplicateTags[tag] ? 'has-text-danger' : '' ">
                      {{ tag }}
                    </td>
                    <td class="is-size-7 is-18-px">
                      <a title="remove entry"
                         class="is-small has-text-grey remove-filter-entry-button"
                         @click="removeTag(filter, tagIndex)">
                        &ndash;
                      </a>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <tag-autocomplete-input v-if="addNewTagColName === filter"
                                              ref="tagAutocompleteInput"
                                              :clear-input-after-selection="true"
                                              :selection-type="'single'"
                                              :auto-focus="true"
                                              @keydown.esc="cancelAddNewTag"
                                              @tagSubmitted="addNewTag(filter, $event)">
                      </tag-autocomplete-input>
                    </td>
                    <td class="is-size-7 is-18-px">
                      <a title="add new entry"
                         class="is-size-7 is-18-px is-small has-text-grey add-new-filter-entry-button"
                         @click="openTagInput(filter)">
                        +
                      </a>
                    </td>
                  </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
          <div class="column is-7">
            <div class="sequence-wrapper">
              <div v-for="(sequenceItem, sequenceIndex) in localDoc.sequence"
                   :key="sequenceIndex"
                   class="sequence">
                <div class="sequence-entries">
                  <table class="table is-narrow is-size-7 sequence-entries-table">
                    <tbody>
                    <tr class="sequence-entry-row method-entry-row">
                      <td class="is-size-7 is-48-px sequence-entries-relation"></td>
                      <td class="is-80-px is-vcentered">
                        Method
                      </td>
                      <td colspan="2">
                        <div class="control has-icons-left is-fullwidth">
                          <input class="input is-small method-entry-input" v-model="sequenceItem.method"
                                 @input="emitDocUpdate"/>
                          <span class="icon is-small is-left has-text-grey-light"><i class="fa fa-code"></i></span>
                        </div>
                      </td>
                      <td class="is-80-px"></td>
                    </tr>
                    <tr class="sequence-entry-row uri-entry-row">
                      <td class="is-size-7 is-48-px has-text-centered is-vcentered has-text-grey-light has-text-weight-medium sequence-entries-relation">
                        AND
                      </td>
                      <td class="is-80-px is-vcentered">
                        URI
                      </td>
                      <td colspan="2">
                        <div class="control has-icons-left is-fullwidth">
                          <input class="input is-small uri-entry-input" v-model="sequenceItem.uri"
                                 @input="emitDocUpdate"/>
                          <span class="icon is-small is-left has-text-grey-light"><i class="fa fa-code"></i></span>
                        </div>
                      </td>
                      <td class="is-80-px"></td>
                    </tr>
                    <tr class="sequence-entry-row host-entry-row">
                      <td class="is-size-7 is-48-px has-text-centered is-vcentered has-text-grey-light has-text-weight-medium sequence-entries-relation">
                        AND
                      </td>
                      <td class="is-80-px is-vcentered">
                        Header
                      </td>
                      <td class="is-100-px is-vcentered">
                        host
                      </td>
                      <td>
                        <div class="control has-icons-left is-fullwidth">
                          <input class="input is-small host-entry-input" v-model="sequenceItem.headers.host"
                                 @input="emitDocUpdate"/>
                          <span class="icon is-small is-left has-text-grey-light"><i class="fa fa-code"></i></span>
                        </div>
                      </td>
                      <td class="is-80-px"></td>
                    </tr>
                    <tr v-for="(sequenceEntry, sequenceEntryIndex) in sequenceItemEntries(sequenceIndex)"
                        :key="sequenceEntryIndex"
                        class="sequence-entry-row">
                      <td class="is-size-7 is-48-px has-text-centered is-vcentered has-text-grey-light has-text-weight-medium sequence-entries-relation">
                        AND
                      </td>
                      <td class="is-80-px is-vcentered">
                        {{ listEntryTypes[sequenceEntry[0]].title }}
                      </td>
                      <td class="is-100-px">
                        {{ sequenceEntry[1][0] }}
                      </td>
                      <td>
                        {{ sequenceEntry[1][1] }}
                      </td>
                      <td class="is-80-px">
                        <a class="is-small has-text-grey remove-entry-button"
                           title="Remove sequence entry"
                           @click="removeSequenceItemEntry(sequenceIndex, sequenceEntry[0], sequenceEntry[1][0])">
                          remove
                        </a>
                      </td>
                    </tr>
                    <tr v-if="newEntrySectionIndex !== sequenceIndex">
                      <td>
                        <a class="is-size-7 light add add-entry-button" title="add new row"
                           @click="setNewEntryIndex(sequenceIndex)"><i class="fas fa-plus"></i></a>
                        &nbsp;&middot;&nbsp;
                        <a class="is-size-7 light remove remove-section-button" title="remove entire section"
                           @click="removeSequenceItem(sequenceIndex)"><i class="fas fa-trash"></i></a>
                      </td>
                      <td colspan="4">
                      </td>
                    </tr>
                    <tr v-if="newEntrySectionIndex === sequenceIndex" class="new-entry-row">
                      <td class="is-size-7" colspan="2">
                        <div class="select is-small is-fullwidth">
                          <select v-model="newEntryType" class="select new-entry-type-selection">
                            <option v-for="(entryType, category) in listEntryTypes" :key="category" :value="category">
                              {{ entryType.title }}
                            </option>
                          </select>
                        </div>
                      </td>
                      <td class="is-size-7 is-100-px">
                        <div class="control has-icons-left is-fullwidth new-entry-name">
                          <input class="input is-small new-entry-name-input"
                                 placeholder="Name"
                                 v-model="newEntryItem.name"/>
                          <span class="icon is-small is-left has-text-grey-light"><i class="fa fa-code"></i></span>
                        </div>
                      </td>
                      <td class="is-size-7">
                        <div class="control has-icons-left is-fullwidth new-entry-value">
                          <input class="input is-small new-entry-value-input"
                                 placeholder="Value"
                                 v-model="newEntryItem.value"/>
                          <span class="icon is-small is-left has-text-grey-light"><i class="fa fa-code"></i></span>
                        </div>
                      </td>
                      <td class="is-size-7 is-80-px">
                        <a class="is-size-7 x-has-text-grey grey add confirm-add-entry-button" title="add new row"
                           @click="addSequenceItemEntry(sequenceIndex)"><i class="fas fa-check"></i> Add</a>
                        <br/>
                        <a class="is-size-7 x-has-text-grey grey remove" title="cancel add new row"
                           @click="setNewEntryIndex(-1)"><i class="fas fa-times"></i> Cancel</a>
                      </td>
                    </tr>
                    </tbody>
                  </table>
                </div>
                <div v-if="localDoc.sequence.length > 1 && sequenceIndex !== localDoc.sequence.length - 1"
                     class="control is-expanded relation-wrapper">
              <span class="tag is-small is-relative">
                THEN
              </span>
                </div>
              </div>
              <button class="button is-small new-sequence-button"
                      @click="addSequenceItem()">
                Create new sequence section
              </button>
            </div>
          </div>
        </div>
        <span class="is-family-monospace has-text-grey-lighter">{{ apiPath }}</span>
      </div>
    </div>
  </div>
</template>

<script>

import LimitAction from '@/components/LimitAction'
import LimitOption from '@/components/LimitOption'
import TagAutocompleteInput from '@/components/TagAutocompleteInput'
import DatasetsUtils from '@/assets/DatasetsUtils'

export default {
  name: 'FlowControl',

  props: {
    selectedDoc: Object,
    apiPath: String
  },

  components: {
    LimitAction,
    LimitOption,
    TagAutocompleteInput
  },

  data() {
    return {
      filters: ['include', 'exclude'],
      barStyle: {
        'include': 'background-color: hsl(141,  71%, 48%); margin: 1rem 0 0.5rem 0;',
        'exclude': 'background-color: hsl(348, 100%, 61%); margin: 1rem 0 0.5rem 0;',
      },
      addNewTagColName: null,
      titles: DatasetsUtils.Titles,
      defaultSequenceItem: {
        'method': 'GET',
        'uri': '/',
        'cookies': {},
        'headers': {
          'host': 'www.example.com'
        },
        'args': {}
      },
      listEntryTypes: {
        'headers': {'title': 'Header', 'pair': true},
        'args': {'title': 'Argument', 'pair': true},
        'cookies': {'title': 'Cookie', 'pair': true},
      },
      keysAreValid: true,
      newEntrySectionIndex: -1,
      newEntryType: 'args',
      newEntryItem: {
        name: '',
        value: ''
      }
    }
  },

  computed: {
    localDoc() {
      return JSON.parse(JSON.stringify(this.selectedDoc))
    },

    duplicateTags() {
      let doc = this.selectedDoc
      let allTags = this.ld.concat(doc['include'], doc['exclude'])
      let dupTags = this.ld.filter(allTags, (val, i, iteratee) => this.ld.includes(iteratee, val, i + 1))
      return this.ld.fromPairs(this.ld.zip(dupTags, dupTags))
    },
  },

  methods: {
    emitDocUpdate() {
      this.$emit('update', this.localDoc)
    },

    // Key

    getOptionTextKey(option, idx) {
      const [type] = Object.keys(option)
      return `${this.selectedDoc.id}_${type}_${idx}`
    },

    generateOption(data) {
      const [firstObjectKey] = Object.keys(data)
      const type = firstObjectKey
      const key = (data[firstObjectKey] || null)
      const value = null
      return {type, key, value}
    },

    addKey() {
      this.selectedDoc.key.push({attrs: 'ip'})
      this.checkKeysValidity()
    },

    removeKey(idx) {
      if (this.selectedDoc.key.length > 1) {
        this.selectedDoc.key.splice(idx, 1)
      }
      this.checkKeysValidity()
    },

    updateKeyOption(option, index) {
      this.selectedDoc.key.splice(index, 1, {
        [option.type]: option.key
      })
      this.checkKeysValidity()
    },

    checkKeysValidity() {
      const keysToCheck = this.ld.countBy(this.selectedDoc.key, item => {
        const key = Object.keys(item)[0]
        return `${key}_${item[key]}`
      })
      this.keysAreValid = true
      for (const key of Object.keys(keysToCheck)) {
        if (keysToCheck[key] > 1) {
          this.keysAreValid = false
          break
        }
      }
      return this.keysAreValid
    },

    // Sequence

    setNewEntryIndex(index) {
      this.newEntryItem = {
        name: '',
        value: ''
      }
      this.newEntryType = 'args'
      this.newEntrySectionIndex = index
    },

    sequenceItemEntries(sequenceIndex) {
      const sequenceItem = this.localDoc.sequence[sequenceIndex]
      const headersEntries = Object.entries(sequenceItem.headers)
      const cookiesEntries = Object.entries(sequenceItem.cookies)
      const argsEntries = Object.entries(sequenceItem.args)
      const mergedEntries = []
      for (let i = 0; i < headersEntries.length; i++) {
        if (headersEntries[i][0] !== 'host') {
          mergedEntries.push(['headers', headersEntries[i]])
        }
      }
      for (let i = 0; i < argsEntries.length; i++) {
        mergedEntries.push(['args', argsEntries[i]])
      }
      for (let i = 0; i < cookiesEntries.length; i++) {
        mergedEntries.push(['cookies', cookiesEntries[i]])
      }
      return mergedEntries
    },

    addSequenceItem() {
      if (this.localDoc.sequence.length > 0) {
        const firstSequenceItem = this.localDoc.sequence[0]
        this.defaultSequenceItem.headers.host = firstSequenceItem.headers.host
      }
      this.localDoc.sequence.push(this.defaultSequenceItem)
      this.emitDocUpdate()
    },

    removeSequenceItem(sequenceIndex) {
      this.localDoc.sequence.splice(sequenceIndex, 1)
      this.emitDocUpdate()
    },

    addSequenceItemEntry(sequenceIndex) {
      const sequenceItem = this.localDoc.sequence[sequenceIndex]
      const newEntryName = this.newEntryItem.name.trim().toLowerCase()
      const newEntryValue = this.newEntryItem.value.trim()
      if (newEntryName && newEntryValue && !Object.prototype.hasOwnProperty.call(sequenceItem[this.newEntryType], newEntryName)) {
        sequenceItem[this.newEntryType][newEntryName] = newEntryValue
      }
      this.setNewEntryIndex(-1)
      this.emitDocUpdate()
    },

    removeSequenceItemEntry(sequenceIndex, type, key) {
      const sequenceItem = this.localDoc.sequence[sequenceIndex]
      delete sequenceItem[type][key]
      this.emitDocUpdate()
    },

    // Tags filters

    addNewTag(filter, entry) {
      if (entry && entry.length > 2) {
        this.localDoc[filter].push(entry)
        this.emitDocUpdate()
      }
    },

    openTagInput(section) {
      this.addNewTagColName = section
    },

    cancelAddNewTag() {
      this.addNewTagColName = null
    },

    removeTag(section, idx) {
      this.localDoc[section].splice(idx, 1)
      this.addNewTagColName = null
      this.emitDocUpdate()
    }
  },
}
</script>

<style scoped>

.sequence-entries {
  margin-bottom: 0.75rem
}

.sequence-entries-relation {
  margin-bottom: 1rem
}

.sequence-entries-table .select, .sequence-entries-table select {
  width: 100%
}

.relation-wrapper {
  text-align: center;
  margin-bottom: 1rem;
}

.relation-wrapper:before {
  content: '';
  position: absolute;
  top: 50%;
  left: 0;
  border-top: 1px solid black;
  background: black;
  width: 100%;
  transform: translateY(-50%);
}

.filter-column:first-of-type {
  padding-left: 0
}

.filter-column:last-of-type {
  padding-right: 0
}

/deep/ .limit-actions {
  padding: 0
}

/deep/ .tag-input {
  font-size: 0.58rem
}

.light {
  color: hsl(0, 0%, 86%)
}

.add:hover {
  color: hsl(0, 0%, 21%)
}

.remove:hover {
  color: hsl(348, 100%, 61%)
}

.is-18-px {
  min-width: 18px;
  max-width: 18px;
  width: 18px;
}

.is-48-px {
  min-width: 40px;
  max-width: 40px;
  width: 48px;
}

.is-80-px {
  min-width: 80px;
  max-width: 80px;
  width: 80px;
}

.is-100-px {
  min-width: 100px;
  max-width: 100px;
  width: 100px;
}

</style>