<template>
  <div>
    <table>
      <thead>
      <tr>
        <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(row, rowIndex) in data" :key="rowIndex" :class="{ 'alt-row': rowIndex % 2 === 1 }">
        <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'SheetDisplay',
  props: {
    headers: Array,
    data: Array
  }
};
</script>

<style scoped>
table {
  margin: 20px auto;
  border-collapse: collapse;
  width: 100%;
  max-width: 800px;
}

th, td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

th {
  background-color: #007bff;
  color: white;
}

tr:hover {
  background-color: #f1f1f1;
}

.alt-row {
  background-color: #f9f9f9;
}
</style>