<script setup lang="ts">
import { ref, computed } from 'vue'

const props = defineProps<{
  currentTable: string;
  tables: Array<{ name: string; fields: Array<{ name: string; type: string; comment: string }> }>;
}>()

const emit = defineEmits(['submit', 'addRelation'])

const fieldName = ref('')
const fieldType = ref('string')
const fieldComment = ref('')
const relationType = ref('one-to-many')
const relatedTable = ref('')
const relatedField = ref('')

const currentTableFields = computed(() => {
  const table = props.tables.find(t => t.name === props.currentTable)
  return table ? table.fields : []
})

const submitForm = () => {
  if (fieldName.value && fieldType.value) {
    emit('submit', { name: fieldName.value, type: fieldType.value, comment: fieldComment.value })
    fieldName.value = ''
    fieldType.value = 'string'
    fieldComment.value = ''
  }
}

const submitRelation = () => {
  if (relatedTable.value && relatedField.value) {
    emit('addRelation', {
      fromTable: props.currentTable,
      fromField: 'id',
      toTable: relatedTable.value,
      toField: relatedField.value,
      type: relationType.value
    })
    relatedTable.value = ''
    relatedField.value = ''
    relationType.value = 'one-to-many'
  }
}
</script>

<template>
  <div class="data-model-form">
    <h3>Add a new field to {{ currentTable }}</h3>
    <div class="form-group">
      <label for="fieldName">Field Name:</label>
      <input id="fieldName" v-model="fieldName" type="text" placeholder="Enter field name" />
    </div>
    <div class="form-group">
      <label for="fieldType">Field Type:</label>
      <select id="fieldType" v-model="fieldType">
        <option value="string">String</option>
        <option value="number">Number</option>
        <option value="boolean">Boolean</option>
        <option value="date">Date</option>
      </select>
    </div>
    <div class="form-group">
      <label for="fieldComment">Field Comment:</label>
      <input id="fieldComment" v-model="fieldComment" type="text" placeholder="Enter field comment" />
    </div>
    <button @click="submitForm">Add Field</button>

    <h3>Current Fields</h3>
    <ul>
      <li v-for="field in currentTableFields" :key="field.name">
        {{ field.name }} ({{ field.type }}): {{ field.comment }}
      </li>
    </ul>

    <h3>Add Relation</h3>
    <div class="form-group">
      <label for="relatedTable">Related Table:</label>
      <select id="relatedTable" v-model="relatedTable">
        <option v-for="table in tables" :key="table.name" :value="table.name">{{ table.name }}</option>
      </select>
    </div>
    <div class="form-group">
      <label for="relatedField">Related Field:</label>
      <select id="relatedField" v-model="relatedField">
        <option v-for="field in tables.find(t => t.name === relatedTable)?.fields || []" :key="field.name" :value="field.name">{{ field.name }}</option>
      </select>
    </div>
    <div class="form-group">
      <label for="relationType">Relation Type:</label>
      <select id="relationType" v-model="relationType">
        <option value="one-to-one">One-to-One</option>
        <option value="one-to-many">One-to-Many</option>
        <option value="many-to-many">Many-to-Many</option>
      </select>
    </div>
    <button @click="submitRelation">Add Relation</button>
  </div>
</template>

<style scoped>
.data-model-form {
  background-color: #34495e;
  padding: 15px;
  border-radius: 8px;
  margin-top: 10px;
}

.form-group {
  margin-bottom: 10px;
}

label {
  display: block;
  margin-bottom: 5px;
  color: var(--text-color);
}

input, select {
  width: 100%;
  padding: 5px;
  border: 1px solid #7f8c8d;
  border-radius: 4px;
  background-color: #2c3e50;
  color: var(--text-color);
}

button {
  background-color: var(--secondary-color);
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

button:hover {
  background-color: #27ae60;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
}
</style>