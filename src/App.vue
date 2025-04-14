<template>
  <div class="container mx-auto px-4 sm:px-6 lg:px-8 py-8 sm:py-12 lg:py-16 flex-grow">
    <!-- Title Section -->
    <div class="text-center mb-8 sm:mb-12">
      <h1
        class="text-3xl sm:text-4xl lg:text-5xl font-extrabold mb-2 text-transparent bg-clip-text bg-gradient-to-b from-black/80 to-white/80">
        Manpower Allocation
      </h1>
      <p class="text-xs sm:text-xl text-black/80">
        Allocate manpower to the locations below
      </p>
    </div>

    <div class="flex flex-col lg:flex-row gap-8 justify-center">
      <!-- Filters Section -->
      <div class="w-full lg:w-1/4 flex flex-col space-y-8">
        <div class="rounded-lg bg-black/4 p-6">
          <h3 class="text-sm font-medium text-gray-700 mb-4">
            Choose Shift
          </h3>
          <select v-model="selectedShift"
            class="mt-4 h-10 w-full rounded-md border border-gray-300 px-3 py-2 text-sm text-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-400">
            <option value="">Choose Shift</option>
            <option value="AM">AM Shift</option>
            <option value="PM">PM Shift</option>
          </select>
          <button
            class="mt-4 w-full bg-black/80 text-white py-2 rounded-md hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-gray-400 mb-4"
            @click="applyShift(selectedShift)">
            Set Shift
          </button>
          <h3 class="text-sm font-medium text-gray-700 mb-4">
            Enter Total Manpower
          </h3>
          <input type="number" id="manpower"
            class="h-10 w-full rounded-md border border-gray-300 px-3 py-2 text-sm text-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-400"
            v-model="manpower" placeholder="Enter Total Manpower" />
          <button
            class="mt-4 w-full bg-black/70 text-white py-2 rounded-md hover:bg-black focus:outline-none focus:ring-2 focus:ring-gray-400"
            @click="allocateManpower">
            Allocate Manpower
          </button>
        </div>

        <div class="rounded-lg bg-black/4 p-6">
          <h3 class="text-sm font-medium text-gray-700 mb-4">Filters</h3>
          <input type="text" v-model="searchQuery"
            class="h-10 w-full rounded-md border border-gray-300 px-3 py-2 text-sm text-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-400"
            placeholder="Search Location" />
          <select v-model="selectedType"
            class="mt-4 h-10 w-full rounded-md border border-gray-300 px-3 py-2 text-sm text-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-400">
            <option value="">Filter by Type</option>
            <option value="Vital">Vital</option>
            <option value="Non-Vital">Non-Vital</option>
            <option value="Normal">Normal</option>
          </select>
          <button
            class="mt-4 w-full bg-black/80 text-white py-2 rounded-md hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-gray-400"
            @click="applyFilters">
            Apply Filters
          </button>


        </div>
      </div>

      <!-- Allocation Table Section -->
      <div class="w-full lg:w-3/4 bg-black/4 rounded-lg p-6">
        <div v-if="paginatedData.length">
          <!-- Table Section -->
          <table class="w-full table-auto text-sm">
            <thead class="bg-black/80 text-white">
              <tr>
                <th class="px-3 py-2 text-left font-medium">Location</th>
                <th class="px-3 py-2 text-left font-medium w-30">Type</th>
                <th class="px-3 py-2 text-left font-medium w-20">Allocated</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(location, index) in paginatedData" :key="index" @click="selectRow(index)"
                class="hover:bg-gray-100 cursor-pointer">
                <td class="border px-3 py-2 text-gray-700">
                  {{ location.name }}
                </td>
                <td class="border px-3 py-2 w-20">
                  <span v-if="location.type === 'Vital'"
                    class="bg-red-100 text-red-800 text-xs font-medium px-2 py-0.5 rounded-sm">Vital</span>
                  <span v-if="location.type === 'Non-Vital'"
                    class="bg-green-100 text-green-800 text-xs font-medium px-2 py-0.5 rounded-sm">Non-Vital</span>
                  <span v-if="location.type === 'Normal'"
                    class="bg-blue-100 text-blue-800 text-xs font-medium px-2 py-0.5 rounded-sm">Normal</span>
                </td>
                <td class="border px-3 py-2 text-center text-gray-700 w-20">
                  {{ location.manpower }}
                </td>
              </tr>
            </tbody>
          </table>

          <!-- Pagination and Manpower Section -->
          <tfoot v-if="totalPages > 1">
            <tr>
              <td colspan="6" class="px-4 py-3">
                <div class="w-full flex justify-between items-center">
                  <button @click="previousPage" :disabled="currentPage === 1"
                    class="px-4 py-2 bg-gray-200 text-black/80 rounded-md text-sm disabled:opacity-50 hover:bg-black/10 inline-block">
                    Previous
                  </button>
                  <span class="text-sm mx-4 text-black/80 inline-block">
                    Page {{ currentPage }} of {{ totalPages }}
                  </span>
                  <button @click="nextPage" :disabled="currentPage === totalPages"
                    class="px-4 py-2 bg-gray-200 text-black/80 rounded-md text-sm disabled:opacity-50 hover:bg-black/10 inline-block">
                    Next
                  </button>
                  <span class="text-sm pl-34 text-black/80 ml-auto inline-block">
                    Remaining Manpower: {{ remainingManpower }}
                  </span>
                </div>
              </td>
            </tr>
          </tfoot>

        </div>
        <div v-else class="flex items-center justify-center h-full">
          <p class="text-lg text-gray-500">
            Your manpower allocation results will be shown here.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firstShift from "./locations/first-shift.json";
import secondShift from "./locations/second-shift.json";

export default {
  data() {
    return {
      manpower: "",
      locations: [],
      filteredLocations: [],
      currentPage: 1,
      perPage: 10,
      remainingManpower: 0,
      searchQuery: "",
      selectedType: "",
      selectedShift: "",
      selectedRow: null,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.filteredLocations.length / this.perPage);
    },
    paginatedData() {
      const start = (this.currentPage - 1) * this.perPage;
      const end = start + this.perPage;
      return this.filteredLocations.slice(start, end);
    },
  },
  methods: {
    selectRow(index) {
      this.selectedRow = this.selectedRow === index ? null : index;
    },
    allocateManpower() {
      if (!this.selectedShift) {
        alert("Please select a shift first.");
        return;
      }
      let totalManpower = parseInt(this.manpower);
      if (isNaN(totalManpower) || totalManpower < 1) {
        alert("Please enter a valid manpower number.");
        return;
      }

      let allocation = this.locations.map((loc) => ({ ...loc, manpower: 0 }));
      let remaining = totalManpower;

      let vitalLocations = allocation.filter((loc) => loc.type === "Vital");
      let controlledLocations = allocation.filter(
        (loc) => loc.type === "Non-Vital"
      );
      let normalLocations = allocation.filter((loc) => loc.type === "Normal");

      vitalLocations.forEach((loc) => {
        let assigned = Math.min(2, remaining);
        loc.manpower = assigned;
        remaining -= assigned;
      });

      controlledLocations.forEach((loc) => {
        if (remaining > 0) {
          loc.manpower = 1;
          remaining -= 1;
        }
      });

      vitalLocations.forEach((loc) => {
        if (remaining > 0 && loc.manpower < 3) {
          let extra = Math.min(3 - loc.manpower, remaining);
          loc.manpower += extra;
          remaining -= extra;
        }
      });

      normalLocations.forEach((loc) => {
        if (remaining > 0) {
          let assigned = Math.min(3, remaining);
          loc.manpower = assigned;
          remaining -= assigned;
        }
      });

      this.locations = allocation; // Update original locations with allocation
      this.filteredLocations = allocation; // Also update filteredLocations
      this.remainingManpower = Math.max(remaining, 0);
    },
    applyFilters() {
      let filteredData = this.locations.filter((location) => {
        const matchesSearch = location.name
          .toLowerCase()
          .includes(this.searchQuery.toLowerCase());
        const matchesType = this.selectedType
          ? location.type === this.selectedType
          : true;
        return matchesSearch && matchesType;
      });

      this.filteredLocations = filteredData;
      this.currentPage = 1; // Reset to first page when filters are applied
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    applyShift(shift) {
      if (shift === "AM") {
        this.locations = firstShift
      }
      if (shift === "PM") {
        this.locations = secondShift
      }
    }
  },
};
</script>

<style scoped>
/* Table and Pagination Styling */
table {
  border-collapse: collapse;
  width: 100%;
}

th,
td {
  padding: 0.75rem;
  text-align: left;
  border: 1px solid #e0e0e0;
}

button:disabled {
  cursor: not-allowed;
}

button:hover {
  background-color: #333;
}
</style>
