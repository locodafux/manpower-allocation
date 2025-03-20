<script setup>
import { ref } from 'vue';
import * as XLSX from 'xlsx';

const manpower = ref(0);
const remainingManpower = ref(0);
const allocationTable = ref([]);

const locations = [
  { name: 'Vital Location 1', type: 'Vital' },
  { name: 'Vital Location 2', type: 'Vital' },
  { name: 'Vital Location 3', type: 'Vital' },
  { name: 'Controlled Location 1', type: 'Controlled' },
  { name: 'Controlled Location 2', type: 'Controlled' },
  { name: 'Controlled Location 3', type: 'Controlled' },
  { name: 'Normal Location 1', type: 'Normal' },
  { name: 'Normal Location 2', type: 'Normal' },
  { name: 'Normal Location 3', type: 'Normal' }
];

const allocateManpower = () => {
  let totalManpower = parseInt(manpower.value);
  if (isNaN(totalManpower) || totalManpower < 1) {
    alert('Please enter a valid manpower number.');
    return;
  }

  let allocation = [];
  let remaining = totalManpower;

  locations.filter(loc => loc.type === 'Vital').forEach(loc => {
    let assigned = Math.min(3, remaining);
    allocation.push({ ...loc, manpower: assigned });
    remaining -= assigned;
  });

  locations.filter(loc => loc.type === 'Controlled').forEach(loc => {
    if (remaining > 0) {
      allocation.push({ ...loc, manpower: 1 });
      remaining -= 1;
    }
  });

  locations.filter(loc => loc.type === 'Normal').forEach(loc => {
    if (remaining > 0) {
      let assigned = Math.min(3, remaining);
      allocation.push({ ...loc, manpower: assigned });
      remaining -= assigned;
    }
  });

  allocationTable.value = allocation;
  remainingManpower.value = remaining;
};

const exportToExcel = () => {
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
    allocationTable.value = json.map(row => ({
      name: row['Location'] || '',
      type: row['Type'] || '',
      manpower: row['Allocated Manpower'] || 0
    }));
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
    <button @click="exportToExcel">Export to Excel</button>
    <input type="file" accept=".xlsx" @change="importFromExcel" />

    <table>
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
