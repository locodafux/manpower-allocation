<script setup>
import { ref } from 'vue';
import * as XLSX from 'xlsx';

const manpower = ref(0);
const remainingManpower = ref(0);
const allocationTable = ref([]);
const locations = ref([]);
const dataUploaded = ref(false);

const allocateManpower = () => {
  if (!dataUploaded.value) {
    alert('Please upload data from Excel first.');
    return;
  }
  
  let totalManpower = parseInt(manpower.value);
  if (isNaN(totalManpower) || totalManpower < 1) {
    alert('Please enter a valid manpower number.');
    return;
  }

  let allocation = [];
  let remaining = totalManpower;

  locations.value.forEach(loc => allocation.push({ ...loc, manpower: 0 }));

  let vitalLocations = allocation.filter(loc => loc.type === 'Vital');
  let controlledLocations = allocation.filter(loc => loc.type === 'Controlled');
  let normalLocations = allocation.filter(loc => loc.type === 'Normal');

  vitalLocations.forEach(loc => {
    let assigned = Math.min(2, remaining);
    loc.manpower = assigned;
    remaining -= assigned;
  });

  controlledLocations.forEach(loc => {
    if (remaining > 0) {
      loc.manpower = 1;
      remaining -= 1;
    }
  });

  vitalLocations.forEach(loc => {
    if (remaining > 0 && loc.manpower < 3) {
      let extra = Math.min(3 - loc.manpower, remaining);
      loc.manpower += extra;
      remaining -= extra;
    }
  });


  normalLocations.forEach(loc => {
    if (remaining > 0) {
      let assigned = Math.min(3, remaining);
      loc.manpower = assigned;
      remaining -= assigned;
    }
  });

  allocationTable.value = allocation;
  remainingManpower.value = Math.max(remaining, 0);
};

const exportToExcel = () => {
  if (allocationTable.value.length === 0) {
    alert('No data to export. Please upload and allocate manpower first.');
    return;
  }
  let wb = XLSX.utils.book_new();
  let ws = XLSX.utils.json_to_sheet(allocationTable.value);
  XLSX.utils.book_append_sheet(wb, ws, 'Manpower Allocation');
  XLSX.writeFile(wb, 'Manpower_Allocation.xlsx');
};

const importFromExcel = (event) => {
  let file = event.target.files[0];
  if (!file) return;

  let reader = new FileReader();
  reader.onload = (e) => {
    let data = new Uint8Array(e.target.result);
    let workbook = XLSX.read(data, { type: 'array' });
    let sheet = workbook.Sheets[workbook.SheetNames[0]];
    let json = XLSX.utils.sheet_to_json(sheet);
    locations.value = json.map(row => ({
      name: row['Location'] || '',
      type: row['Type'] || '',
      manpower: 0
    }));
    allocationTable.value = locations.value;
    dataUploaded.value = true;
  };
  reader.readAsArrayBuffer(file);
};
</script>

<template>
  <div>
    <h2>Manpower Allocation</h2>
    <label for="manpower">Enter Total Manpower:</label>
    <input type="number" v-model="manpower" min="1" />
    <button @click="allocateManpower">Allocate</button>
    <button @click="exportToExcel" :disabled="allocationTable.length === 0">Export to Excel</button>
    <input type="file" accept=".xlsx" @change="importFromExcel" />

    <table v-if="allocationTable.length > 0">
      <thead>
        <tr>
          <th>Location</th>
          <th>Type</th>
          <th>Allocated Manpower</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in allocationTable" :key="index">
          <td>{{ row.name }}</td>
          <td>{{ row.type }}</td>
          <td>{{ row.manpower }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>No record found. Please upload data from Excel first.</p>

    <h3>Remaining Manpower: <span>{{ remainingManpower }}</span></h3>
  </div>
</template>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}
th, td {
  border: 1px solid black;
  padding: 8px;
  text-align: center;
}
</style>