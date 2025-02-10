<template>
  <div id="app">
    <div v-if="sheetNames.length">
      <label for="sheetSelect">Select Sheet:</label>
      <select id="sheetSelect" v-model="selectedSheet" @change="loadSheetData">
        <option v-for="(sheet, index) in sheetNames" :key="index" :value="sheet">{{ sheet }}</option>
      </select>
    </div>
    <table v-if="data.length">
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
    <div v-else>Loading data...</div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  name: 'App',
  data() {
    return {
      headers: [],
      data: [],
      sheetNames: [],
      selectedSheet: '',
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
        this.sheetNames = this.workbook.SheetNames;

        if (this.sheetNames.length > 0) {
          this.selectedSheet = this.sheetNames[0];
          this.loadSheetData();
        }
      } catch (error) {
        console.error('Error fetching the Excel file:', error);
      }
    },
    loadSheetData() {
      const worksheet = this.workbook.Sheets[this.selectedSheet];
      const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

      if (jsonData.length > 0) {
        this.headers = jsonData[0];
        this.data = jsonData.slice(1);
      } else {
        this.headers = [];
        this.data = [];
      }
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

table {
  margin: 20px auto;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

select {
  margin: 20px;
  padding: 10px;
}
</style>