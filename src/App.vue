<template>
  <div class="container mx-auto px-4 sm:px-6 lg:px-8 py-8 sm:py-12 lg:py-16 flex-grow">
    <!-- Toast Notification -->
    <div v-if="toast.show" class="fixed top-4 right-4 z-50">
      <div :class="[
        'flex items-center p-4 rounded-lg shadow-lg text-white',
        toast.type === 'error' ? 'bg-red-500' : 'bg-[#041D56]',
      ]">
        <span>{{ toast.message }}</span>
        <button @click="toast.show = false" class="ml-4">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
          </svg>
        </button>
      </div>
    </div>

    <!-- Title Section -->
    <div class="text-center mb-8 sm:mb-12">
      <h1
        class="text-3xl sm:text-4xl lg:text-5xl font-extrabold mb-2 text-transparent bg-clip-text bg-gradient-to-b from-[#041D56] to-[#276DAB]">
        Manpower Allocation
      </h1>
      <p class="text-xs sm:text-xl text-[#041D56]/80">
        Allocate manpower to the locations below
      </p>
    </div>

    <div class="flex flex-col lg:flex-row gap-8 justify-center">
      <!-- Filters Section -->
      <div class="w-full lg:w-1/4 flex flex-col space-y-8">
        <div class="rounded-lg bg-[#276DAB]/10 p-6">
          <h3 class="text-sm font-medium text-[#041D56] mb-4">Choose Shift</h3>
          <select v-model="selectedShift"
            class="mt-4 h-10 w-full rounded-md border border-[#102574]/30 px-3 py-2 text-sm text-[#041D56] focus:outline-none focus:ring-2 focus:ring-[#276DAB] hover:border-[#276DAB]">
            <option value="">Choose Shift</option>
            <option value="AM">AM Shift</option>
            <option value="PM">PM Shift</option>
          </select>
          <button
            class="mt-4 w-full bg-[#041D56] text-white py-2 rounded-md hover:bg-[#102574] focus:outline-none focus:ring-2 focus:ring-[#276DAB] mb-4 transition-colors duration-200"
            @click="applyShift(selectedShift)">
            Set Shift
          </button>
          <h3 class="text-sm font-medium text-[#041D56] mb-4">
            Enter Total Manpower
          </h3>
          <input type="number" id="manpower"
            class="h-10 w-full rounded-md border border-[#102574]/30 px-3 py-2 text-sm text-[#041D56] focus:outline-none focus:ring-2 focus:ring-[#276DAB] hover:border-[#276DAB]"
            v-model="manpower" placeholder="Enter Total Manpower" />
          <button
            class="mt-4 w-full bg-[#102574] text-white py-2 rounded-md hover:bg-[#041D56] focus:outline-none focus:ring-2 focus:ring-[#276DAB] transition-colors duration-200"
            @click="allocateManpower">
            Allocate Manpower
          </button>
        </div>

        <div class="rounded-lg bg-[#276DAB]/10 p-6">
          <h3 class="text-sm font-medium text-[#041D56] mb-4">Filters</h3>
          <input type="text" v-model="searchQuery"
            class="h-10 w-full rounded-md border border-[#102574]/30 px-3 py-2 text-sm text-[#041D56] focus:outline-none focus:ring-2 focus:ring-[#276DAB] hover:border-[#276DAB]"
            placeholder="Search Location" />
          <select v-model="selectedType"
            class="mt-4 h-10 w-full rounded-md border border-[#102574]/30 px-3 py-2 text-sm text-[#041D56] focus:outline-none focus:ring-2 focus:ring-[#276DAB] hover:border-[#276DAB]">
            <option value="">Filter by Type</option>
            <option value="Vital">Vital</option>
            <option value="Non-Vital">Non-Vital</option>
          </select>
          <button
            class="mt-4 w-full bg-[#041D56] text-white py-2 rounded-md hover:bg-[#102574] focus:outline-none focus:ring-2 focus:ring-[#276DAB] transition-colors duration-200"
            @click="applyFilters">
            Apply Filters
          </button>
        </div>
      </div>

      <!-- Allocation Table Section -->
      <div class="w-full lg:w-3/4 bg-[#276DAB]/10 rounded-lg p-6">
        <div v-if="paginatedData.length">
          <!-- Table Section -->
          <table class="w-full table-auto text-sm">
            <thead class="bg-[#041D56] text-white">
              <tr>
                <th class="px-3 py-2 text-left font-medium w-30">Rank</th>
                <th class="px-3 py-2 text-left font-medium">Location</th>
                <th class="px-3 py-2 text-left font-medium w-20">Allocated</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(location, index) in paginatedData" :key="index" @click="selectRow(index)"
                class="hover:bg-[#276DAB]/20 cursor-pointer transition-colors duration-200">
                <td class="border px-3 py-2 text-[#041D56]">
                  {{ location.rank }}
                </td>
                <td class="border px-3 py-2 text-[#041D56]">
                  {{ location.name }}
                </td>

                <td class="border px-3 py-2 text-center text-[#041D56] w-20">
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
                    class="px-4 py-2 bg-[#276DAB]/20 text-[#041D56] rounded-md text-sm disabled:opacity-50 hover:bg-[#276DAB]/30 hover:text-[#102574] inline-block transition-colors duration-200">
                    Previous
                  </button>
                  <span class="text-sm mx-4 text-[#041D56] inline-block">
                    Page {{ currentPage }} of {{ totalPages }}
                  </span>
                  <button @click="nextPage" :disabled="currentPage === totalPages"
                    class="px-4 py-2 bg-[#276DAB]/20 text-[#041D56] rounded-md text-sm disabled:opacity-50 hover:bg-[#276DAB]/30 hover:text-[#102574] inline-block transition-colors duration-200">
                    Next
                  </button>
                  <span class="text-sm pl-34 text-[#041D56] ml-auto inline-block">
                    Remaining Manpower: {{ remainingManpower }}
                  </span>
                </div>
              </td>
            </tr>
          </tfoot>
        </div>
        <div v-else class="flex items-center justify-center h-full">
          <p class="text-lg text-[#041D56]/70">
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
      toast: {
        show: false,
        message: "",
        type: "info", // 'info' or 'error'
      },
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
    showToast(message, type = "info") {
      this.toast = {
        show: true,
        message,
        type,
      };
      setTimeout(() => {
        this.toast.show = false;
      }, 5000);
    },

    selectRow(index) {
      this.selectedRow = this.selectedRow === index ? null : index;
    },

    allocateManpower() {
      if (!this.selectedShift) {
        this.showToast("Please select a shift first.", "error");
        return;
      }

      if (parseInt(this.manpower) < 10) {
        this.showToast("Insufficient number of manpower", "error");
        return;
      }
      if (parseInt(this.manpower) > 60) {
        this.showToast("Maximum manpower deployed", "error");
        return;
      }
      const totalManpower = parseInt(this.manpower);
      if (isNaN(totalManpower) || totalManpower < 0) {
        this.showToast(
          "Please enter a valid positive manpower number.",
          "error"
        );
        return;
      }

      const allocation = [...this.locations].sort((a, b) => a.rank - b.rank);
      allocation.forEach((loc) => {
        if (loc) loc.manpower = 0;
      });

      let remaining = totalManpower;

      // Top 10
      if (totalManpower <= 10) {
        for (let i = 0; i < 10 && i < allocation.length && remaining > 0; i++) {
          allocation[i].manpower = 1;
          remaining -= 1;
        }
      }

      // Top 25
      if (totalManpower >= 11 && totalManpower <= 25) {
        for (let i = 0; i < 3 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(2 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 3; i < allocation.length && remaining > 0; i++) {
          if (allocation[i].manpower < 1) {
            allocation[i].manpower = 1;
            remaining -= 1;
          }
        }
      }

      // Top 45
      if (totalManpower >= 26 && totalManpower <= 30) {
        for (let i = 0; i < 3 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(3 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 3; i < 10 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(2 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 10; i < allocation.length && remaining > 0; i++) {
          if (allocation[i].manpower < 1) {
            allocation[i].manpower = 1;
            remaining -= 1;
          }
        }
      }
      // Top 45
      if (totalManpower >= 30 && totalManpower <= 45) {
        for (let i = 0; i < 5 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(3 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 5; i < 10 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(2 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 10; i < allocation.length && remaining > 0; i++) {
          if (allocation[i].manpower < 1) {
            allocation[i].manpower = 1;
            remaining -= 1;
          }
        }
      }

      if (totalManpower >= 46) {
        for (let i = 0; i < 5 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(3 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 5; i < 15 && i < allocation.length && remaining > 0; i++) {
          const add = Math.min(2 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }

        for (let i = 10; i < allocation.length && remaining > 0; i++) {
          if (allocation[i].manpower < 1) {
            allocation[i].manpower = 1;
            remaining -= 1;
          }
        }

        for (let i = 11; i < allocation.length && remaining > 0; i++) {
          const add = Math.min(2 - allocation[i].manpower, remaining);
          allocation[i].manpower += add;
          remaining -= add;
        }
      }

      this.locations = allocation;
      this.filteredLocations = allocation;
      this.remainingManpower = remaining;
      this.showToast("Manpower allocated successfully!");
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
      // Clear all data
      this.locations = [];
      this.filteredLocations = [];
      this.remainingManpower = 0;
      this.currentPage = 1;
      this.selectedType = "";
      this.searchQuery = "";
      this.manpower = "";

      if (shift === "AM") {
        this.locations = firstShift;
      } else if (shift === "PM") {
        this.locations = secondShift;
      }
    },
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
  opacity: 0.5;
}

/* Smooth transitions for hover effects */
button,
input,
select,
tr {
  transition: all 0.2s ease-in-out;
}

/* Custom hover effects */
button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(4, 29, 86, 0.2);
}

tr:hover {
  transform: translateX(2px);
}
</style>
