<script setup lang="ts">
import { ref, reactive } from 'vue'
import axios from 'axios'
import DataModelForm from './DataModelForm.vue'
import DataModelTable from './DataModelTable.vue'

interface Field {
  name: string;
  type: string;
  comment: string;
}

interface Table {
  name: string;
  fields: Field[];
}

interface Relation {
  fromTable: string;
  fromField: string;
  toTable: string;
  toField: string;
  type: 'one-to-one' | 'one-to-many' | 'many-to-many';
}

interface Message {
  text: string;
  isUser: boolean;
  type: 'text' | 'form' | 'table';
  data?: any;
}

const messages = ref<Message[]>([])
const userInput = ref('')
const showForm = ref(false)
const showTable = ref(false)
const dataModel = reactive({
  tables: [] as Table[],
  relations: [] as Relation[]
})
const currentTable = ref('')

const sendMessage = async () => {
  if (userInput.value.trim() === '') return

  messages.value.push({ text: userInput.value, isUser: true, type: 'text' })

  const response = await getLLMResponse(userInput.value)
  messages.value.push(response)

  if (response.type === 'form') {
    showForm.value = true
  } else if (response.type === 'table') {
    showTable.value = true
  }

  userInput.value = ''
}

const simulateLLMResponse = (input: string): Message => {
  if (input.toLowerCase().includes('create table')) {
    return {
      text: 'Sure, I can help you create a new table. Please provide the details for the new table.',
      isUser: false,
      type: 'form'
    }
  } else if (input.toLowerCase().includes('show tables') || input.toLowerCase().includes('list tables')) {
    return {
      text: 'Here are all the tables in your data model:',
      isUser: false,
      type: 'table',
      data: dataModel.tables
    }
  } else {
    return {
      text: 'I understand you want to work on your data model. What would you like to do? You can create a new table, add fields to an existing table, or view all tables.',
      isUser: false,
      type: 'text'
    }
  }
}

const getLLMResponse = async (input: string): Promise<Message> => {
  try {
    const response = await axios.post('http://192.168.1.110:11434/api/generate', {
      model: 'llama3.2:latest',
      prompt: `You are a helpful AI assistant for building data models. The user's input is: "${input}". Based on this, provide a response that helps with creating or modifying data models. If the user wants to create a table, respond with a message of type 'form'. If the user wants to see all tables, respond with a message of type 'table'. Otherwise, respond with a message of type 'text'. Your response should be in JSON format with 'text', 'type', and optionally 'data' fields.`,
      stream: false
    })

    const llmResponse = JSON.parse(response.data.response)

    if (llmResponse.type === 'form' && input.toLowerCase().includes('create table')) {
      const tableName = input.split(' ').pop()
      if (tableName && !dataModel.tables.find(t => t.name === tableName)) {
        dataModel.tables.push({
          name: tableName,
          fields: [
            { name: 'id', type: 'string', comment: 'Unique identifier' },
            { name: 'created_at', type: 'date', comment: 'Creation timestamp' },
            { name: 'updated_at', type: 'date', comment: 'Last update timestamp' }
          ]
        })
        currentTable.value = tableName
      }
    } else if (llmResponse.type === 'table') {
      llmResponse.data = dataModel.tables
    }

    return llmResponse as Message
  } catch (error) {
    console.error('Error calling LLM:', error)
    console.log('Falling back to simulated response')
    return simulateLLMResponse(input)
  }
}

const handleFormSubmit = (formData: Field) => {
  const table = dataModel.tables.find(t => t.name === currentTable.value)
  if (table) {
    table.fields.push(formData)
    messages.value.push({
      text: `Field "${formData.name}" of type "${formData.type}" has been added to the "${currentTable.value}" table.`,
      isUser: false,
      type: 'text'
    })
  }
}

const addRelation = (relation: Relation) => {
  dataModel.relations.push(relation)
  messages.value.push({
    text: `Relation added: ${relation.fromTable}.${relation.fromField} ${relation.type} ${relation.toTable}.${relation.toField}`,
    isUser: false,
    type: 'text'
  })
}
</script>

<template>
  <div class="chat-interface">
    <div class="messages">
      <div v-for="(message, index) in messages" :key="index" :class="{ 'user-message': message.isUser, 'ai-message': !message.isUser }">
        <p>{{ message.text }}</p>
        <DataModelForm
          v-if="message.type === 'form' && showForm"
          :currentTable="currentTable"
          :tables="dataModel.tables"
          @submit="handleFormSubmit"
          @addRelation="addRelation"
        />
        <DataModelTable
          v-if="message.type === 'table' && showTable"
          :data="message.data"
        />
      </div>
    </div>
    <div class="input-area">
      <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Type your message here..." />
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<style scoped>
.chat-interface {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #2c3e50;
  color: #ecf0f1;
}

.messages {
  flex-grow: 1;
  overflow-y: auto;
  padding: 20px;
}

.user-message, .ai-message {
  margin-bottom: 15px;
  padding: 10px;
  border-radius: 5px;
}

.user-message {
  background-color: #34495e;
  align-self: flex-end;
}

.ai-message {
  background-color: #2980b9;
  align-self: flex-start;
}

.input-area {
  display: flex;
  padding: 20px;
}

input {
  flex-grow: 1;
  padding: 10px;
  border: none;
  border-radius: 5px 0 0 5px;
  background-color: #34495e;
  color: #ecf0f1;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 0 5px 5px 0;
  background-color: #27ae60;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #2ecc71;
}
</style>