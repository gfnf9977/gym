<template>
  <div>
    <div class="weight-display">
      <span v-if="!isEditing">Вага: {{ localWeight }}</span>
      <input v-else v-model.number="newWeight" type="number" class="weight-input" />
      <button v-if="!isEditing" @click="startEditing" class="edit-button">Edit</button>
      <button v-else @click="saveWeight" class="save-button">Save</button>
    </div>
    <div v-if="updateStatus" :class="['status-message', updateStatus.type]">
      {{ updateStatus.message }}
    </div>
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
    weight: {
      type: [String, Number],
      required: true
    },
    headers: {
      type: Array,
      required: true
    },
    data: {
      type: Array,
      required: true
    },
    username: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      isEditing: false,
      newWeight: null,
      localWeight: this.weight,
      updateStatus: null
    };
  },
  watch: {
    weight(newVal) {
      this.localWeight = newVal;
    }
  },
  methods: {
    startEditing() {
      this.newWeight = this.localWeight;
      this.isEditing = true;
    },
    async saveWeight() {
      try {
        const response = await fetch('http://localhost:5000/update-weight', {  // Update URL
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
          },
          body: JSON.stringify({
            weight: parseFloat(this.newWeight),  // Ensure weight is a number
            username: this.username
          })
        });

        const data = await response.json();
        console.log('Server response:', data);  // Debug log

        if (response.ok) {
          this.updateStatus = {
            type: 'success',
            message: 'Weight updated successfully!'
          };
          this.localWeight = this.newWeight;
          this.isEditing = false;
          this.$emit('weight-updated', this.newWeight);
        } else {
          this.updateStatus = {
            type: 'error',
            message: data.error || 'Failed to update weight'
          };
        }
      } catch (error) {
        console.error('Error details:', error);  // Detailed error logging
        this.updateStatus = {
          type: 'error',
          message: 'Error connecting to server. Check if the server is running on port 5000.'
        };
      }

      // Clear status message after 3 seconds
      setTimeout(() => {
        this.updateStatus = null;
      }, 3000);
    }
  }
};
</script>

<style scoped>
.weight-display {
  font-size: 20px;
  margin-bottom: 20px;
  color: #555;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.weight-input {
  margin-left: 10px;
  padding: 5px;
  font-size: 16px;
  width: 80px;
}

.edit-button, .save-button {
  padding: 5px 15px;
  font-size: 14px;
  cursor: pointer;
  border-radius: 4px;
  border: none;
  color: white;
  transition: background-color 0.3s;
}

.edit-button {
  background-color: #007bff;
}

.save-button {
  background-color: #28a745;
}

.status-message {
  margin: 10px 0;
  padding: 10px;
  border-radius: 4px;
  text-align: center;
}

.success {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.error {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.weight-display {
  font-size: 20px;
  margin-bottom: 20px;
  color: #555;
  font-weight: bold;
  display: flex;
  align-items: center;
}

.weight-input {
  margin-left: 10px;
  padding: 5px;
  font-size: 16px;
}

.edit-button, .save-button {
  margin-left: 10px;
  padding: 5px 10px;
  font-size: 14px;
  cursor: pointer;
}

.edit-button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
}

.save-button {
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
}

table {
  margin: 20px auto;
  border-collapse: collapse;
  width: 100%;
  max-width: 800px;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

th, td {
  border: 1px solid #ddd;
  padding: 16px;
  text-align: left;
}

th {
  background-color: #007bff;
  color: white;
  font-weight: bold;
}

tr:hover {
  background-color: #f1f1f1;
}

.alt-row {
  background-color: #f9f9f9;
}
</style>
