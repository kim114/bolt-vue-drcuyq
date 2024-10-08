<script setup lang="ts">
defineProps<{
  data: Array<{ name: string; fields: Array<{ name: string; type: string; comment: string }> }>
}>()
</script>

<template>
  <div class="data-model-table">
    <div v-for="table in data" :key="table.name" class="table-container">
      <h3>{{ table.name }}</h3>
      <table>
        <thead>
          <tr>
            <th>Field Name</th>
            <th>Field Type</th>
            <th>Comment</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="field in table.fields" :key="field.name">
            <td>{{ field.name }}</td>
            <td>{{ field.type }}</td>
            <td>{{ field.comment }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.data-model-table {
  margin-top: 10px;
}

.table-container {
  margin-bottom: 20px;
}

h3 {
  color: var(--primary-color);
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #7f8c8d;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #34495e;
  font-weight: bold;
}

tr:nth-child(even) {
  background-color: #2c3e50;
}
</style>