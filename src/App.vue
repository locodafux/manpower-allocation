<template>

  <div :class="{ 'dark bg-gray-900 text-white': darkMode }"
    class="min-h-screen p-4 md:p-10 transition-colors flex justify-center">
    <div :class="{ 'bg-gray-900 text-white': darkMode, 'bg-white text-black': !darkMode }"
      class="w-full max-w-4xl p-4 md:p-10 rounded-lg shadow-lg transition-colors overflow-auto">

      <div class="flex justify-between items-center mb-3">
        <h2 class="text-xl font-semibold">Manpower Allocation</h2>
        <label class="flex items-center cursor-pointer text-sm">
          <span class="mr-2">Dark Mode</span>
          <input type="checkbox" v-model="darkMode" class="hidden" />
          <div class="relative w-10 h-5 bg-gray-300 dark:bg-gray-700 rounded-full transition-all flex items-center">
            <div class="absolute w-4 h-4 bg-white rounded-full shadow-md transform transition-transform duration-300"
              :class="{ 'translate-x-5': darkMode, 'translate-x-1': !darkMode }">
            </div>
          </div>
        </label>
      </div>


      <div class="mb-3">
        <label for="manpower" class="block text-sm mb-3">Enter Total Manpower:</label>
        <input type="number" v-model="manpower" min="1"
          :class="{ 'bg-gray-100 text-black': !darkMode, 'bg-gray-700 text-white': darkMode }"
          class="w-full p-1.5 border rounded-md transition-colors text-sm" />
      </div>

      <div class="flex gap-2 mb-3">
        <button @click="allocateManpower"
          class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1.5 rounded-md text-sm">Allocate</button>
          <!-- 
        <button @click="exportToExcel" :disabled="allocationTable.length === 0"
          class="bg-green-500 hover:bg-green-600 text-white px-3 py-1.5 rounded-md text-sm disabled:opacity-50">Export</button>
           -->
      </div>
      <!-- <div class="mb-5">

        <div class="mb-5">
          <label for="excel-upload" class="block text-sm font-medium"
            :class="{ 'text-black': !darkMode, 'text-white': darkMode }">
            Upload Excel File
          </label>

          <div
            class="relative flex items-center justify-center w-full p-4 border-2 border-dashed border-gray-300 dark:border-gray-500 rounded-lg cursor-pointer bg-gray-50 dark:bg-gray-800 hover:border-blue-500 dark:hover:border-blue-400 transition">
            <input id="excel-upload" type="file" accept=".xlsx" @change="importFromExcel"
              class="absolute inset-0 w-full h-full opacity-0 cursor-pointer" />
            <span class="text-gray-600 dark:text-gray-300 text-sm">Click to upload or drag & drop</span>
          </div>
        </div>
      </div> -->

      <div  class="overflow-x-auto">
        <table class="w-full border-collapse border border-gray-300 dark:border-gray-700 text-sm">
          <thead class="bg-gray-200 dark:bg-gray-800">
            <tr>
              <th class="border border-gray-300 dark:border-gray-700 p-1.5">Location</th>
              <th class="border border-gray-300 dark:border-gray-700 p-1.5">Type</th>
              <th class="border border-gray-300 dark:border-gray-700 p-1.5">Allocated</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in allocationTable" :key="index" class="text-center">
              <td class="border border-gray-300 dark:border-gray-700 p-1.5">{{ row.name }}</td>
              <td class="border border-gray-300 dark:border-gray-700 p-1.5">{{ row.type }}</td>
              <td class="border border-gray-300 dark:border-gray-700 p-1.5">{{ row.manpower }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- <p
        class="bg-white text-black dark:bg-black dark:text-white p-3 border border-gray-300 dark:border-gray-700 rounded-md text-sm text-center">
        No records available. Please upload an Excel file and allocate manpower.
      </p> -->

      <h3 class="mt-3 text-base font-medium">Remaining Manpower: <span class="text-blue-500">{{ remainingManpower
      }}</span></h3>
    </div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      manpower: 0,
      remainingManpower: 0,
      allocationTable: [ ],
      locations: [
      { name: 'Location 1', type: 'Vital', manpower: 0 },
        { name: 'Location 2', type: 'Vital', manpower: 0 },
        { name: 'Location 3', type: 'Vital', manpower: 0 },
        { name: 'Location 4', type: 'Controlled', manpower: 0 },
        { name: 'Location 5', type: 'Controlled', manpower: 0 },
        { name: 'Location 6', type: 'Controlled', manpower: 0 },
        { name: 'Location 7', type: 'Normal', manpower: 0 },
        { name: 'Location 8', type: 'Normal', manpower: 0 },
        { name: 'Location 9', type: 'Normal', manpower: 0 },
        { name: 'Location 10', type: 'Vital', manpower: 0 },
        { name: 'Location 11', type: 'Vital', manpower: 0 },
        { name: 'Location 12', type: 'Vital', manpower: 0 },
        { name: 'Location 13', type: 'Controlled', manpower: 0 },
        { name: 'Location 14', type: 'Controlled', manpower: 0 },
        { name: 'Location 15', type: 'Controlled', manpower: 0 },
        { name: 'Location 16', type: 'Normal', manpower: 0 },
        { name: 'Location 17', type: 'Normal', manpower: 0 },
        { name: 'Location 18', type: 'Normal', manpower: 0 },
        { name: 'Location 19', type: 'Vital', manpower: 0 },
        { name: 'Location 20', type: 'Vital', manpower: 0 },
        { name: 'Location 21', type: 'Vital', manpower: 0 },
        { name: 'Location 22', type: 'Controlled', manpower: 0 },
        { name: 'Location 23', type: 'Controlled', manpower: 0 },
        { name: 'Location 24', type: 'Controlled', manpower: 0 },
        { name: 'Location 25', type: 'Normal', manpower: 0 },
        { name: 'Location 26', type: 'Normal', manpower: 0 },
        { name: 'Location 27', type: 'Normal', manpower: 0 },
        { name: 'Location 28', type: 'Vital', manpower: 0 },
        { name: 'Location 29', type: 'Vital', manpower: 0 },
        { name: 'Location 30', type: 'Controlled', manpower: 0 },
        { name: 'Location 31', type: 'Controlled', manpower: 0 },
        { name: 'Location 32', type: 'Normal', manpower: 0 },
        { name: 'Location 33', type: 'Normal', manpower: 0 }
      ],
      dataUploaded: false,
      darkMode: true 
    };
  },
  methods: {
    allocateManpower() {
      // if (!this.dataUploaded) {
      //   alert('Please upload data from Excel first.');
      //   return;
      // }

      let totalManpower = parseInt(this.manpower);
      if (isNaN(totalManpower) || totalManpower < 1) {
        alert('Please enter a valid manpower number.');
        return;
      }

      let allocation = this.locations.map(loc => ({ ...loc, manpower: 0 }));
      let remaining = totalManpower;

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

      this.allocationTable = allocation;
      this.remainingManpower = Math.max(remaining, 0);
    },
    exportToExcel() {
      if (this.allocationTable.length === 0) {
        alert('No data to export. Please upload and allocate manpower first.');
        return;
      }
      let wb = XLSX.utils.book_new();
      let ws = XLSX.utils.json_to_sheet(this.allocationTable);
      XLSX.utils.book_append_sheet(wb, ws, 'Manpower Allocation');
      XLSX.writeFile(wb, 'Manpower_Allocation.xlsx');
    },
    importFromExcel(event) {
      let file = event.target.files[0];
      if (!file) return;

      let reader = new FileReader();
      reader.onload = (e) => {
        let data = new Uint8Array(e.target.result);
        let workbook = XLSX.read(data, { type: 'array' });
        let sheet = workbook.Sheets[workbook.SheetNames[0]];
        let json = XLSX.utils.sheet_to_json(sheet);

        this.locations = json.map(row => ({
          name: row['Location'] || '',
          type: row['Type'] || '',
          manpower: 0
        }));
        this.allocationTable = this.locations;
        this.dataUploaded = true;
      };
      reader.readAsArrayBuffer(file);
    }
  }
};
</script>
