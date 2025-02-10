<template>
  <div id="app">
    <input v-model="inputValue" placeholder="Enter username" />
    <button @click="checkUsername">Login</button>
    <Sheet2Display v-if="showSheet2" :headers="sheet2Headers" :data="sheet2Data" />
    <div v-else-if="inputValue">No match found or invalid input.</div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';
import Sheet2Display from './components/Sheet2Display.vue';

export default {
  name: 'App',
  components: {
    Sheet2Display
  },
  data() {
    return {
      inputValue: '',
      sheet1Data: [],
      sheet2Data: [],
      sheet2Headers: [],
      showSheet2: false,
      workbook: null
    };
  },
  mounted() {
    this.fetchExcelData();
  },
  methods: {
    async fetchExcelData() {
      try {
        const response = await fetch('/gymexp.xlsx'); // Path to the file in the public directory
        const arrayBuffer = await response.arrayBuffer();
        this.workbook = XLSX.read(arrayBuffer);

        // Load data from "Лист1"
        const sheet1 = this.workbook.Sheets['Лист1'];
        this.sheet1Data = XLSX.utils.sheet_to_json(sheet1, { header: 1 });

        // Load data from "Лист2"
        const sheet2 = this.workbook.Sheets['Лист2'];
        const sheet2Json = XLSX.utils.sheet_to_json(sheet2, { header: 1 });
        if (sheet2Json.length > 0) {
          this.sheet2Headers = sheet2Json[0];
          this.sheet2Data = sheet2Json.slice(1);
        }
      } catch (error) {
        console.error('Error fetching the Excel file:', error);
      }
    },
    checkUsername() {
      const usernameIndex = this.sheet1Data[0].indexOf('username');
      if (usernameIndex === -1) return; // If 'username' column is not found

      const usernames = this.sheet1Data.slice(1).map(row => row[usernameIndex]);
      this.showSheet2 = usernames.includes(this.inputValue);
    }
  }
};
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

input {
  margin: 20px;
  padding: 10px;
  width: 200px;
}

button {
  margin: 20px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}
</style>


<template>
  <div>
    <table>
      <thead>
      <tr>
        <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(row, rowIndex) in data" :key="rowIndex">
        <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'Sheet2Display',
  props: {
    headers: Array,
    data: Array
  }
};
</script>