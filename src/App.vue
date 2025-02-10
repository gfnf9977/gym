<template>
  <div id="app">
    <div class="container">
      <input v-model="inputValue" placeholder="Enter username" class="input-field" />
      <button @click="checkUsername" class="login-button">Login</button>
      <SheetDisplay
          v-if="showSheet"
          :headers="currentSheetHeaders"
          :data="currentSheetData"
      />
      <div v-else-if="inputValue" class="no-match">No match found or invalid input.</div>
    </div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';
import SheetDisplay from './components/SheetDisplay.vue';

export default {
  name: 'App',
  components: {
    SheetDisplay
  },
  data() {
    return {
      inputValue: '',
      sheet1Data: [],
      sheetsData: {},
      currentSheetHeaders: [],
      currentSheetData: [],
      showSheet: false,
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

        // Load data from other sheets dynamically
        for (const sheetName in this.workbook.Sheets) {
          if (sheetName !== 'Лист1') {
            const sheet = this.workbook.Sheets[sheetName];
            const sheetJson = XLSX.utils.sheet_to_json(sheet, { header: 1 });
            if (sheetJson.length > 0) {
              this.sheetsData[sheetName] = {
                headers: sheetJson[0],
                data: sheetJson.slice(1)
              };
            }
          }
        }
      } catch (error) {
        console.error('Error fetching the Excel file:', error);
      }
    },
    checkUsername() {
      const usernameIndex = this.sheet1Data[0].indexOf('username');
      const idIndex = this.sheet1Data[0].indexOf('id');
      if (usernameIndex === -1 || idIndex === -1) return; // If 'username' or 'id' column is not found

      const usernames = this.sheet1Data.slice(1).map(row => row[usernameIndex]);
      const ids = this.sheet1Data.slice(1).map(row => row[idIndex]);

      const userIndex = usernames.indexOf(this.inputValue);
      if (userIndex !== -1) {
        const sheetId = ids[userIndex];
        const sheetName = `Лист${sheetId}`;
        if (this.sheetsData[sheetName]) {
          this.currentSheetHeaders = this.sheetsData[sheetName].headers;
          this.currentSheetData = this.sheetsData[sheetName].data;
          this.showSheet = true;
        }
      } else {
        this.showSheet = false;
      }
    }
  }
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f7f7f7;
}

.container {
  background: white;
  padding: 40px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.input-field {
  margin: 20px;
  padding: 15px;
  width: 240px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

.login-button {
  margin: 20px;
  padding: 15px 30px;
  font-size: 16px;
  color: white;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.login-button:hover {
  background-color: #0056b3;
}

.no-match {
  color: red;
  margin-top: 20px;
}
</style>