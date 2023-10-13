<script>
import { ref, computed, watch } from "vue"
import amortizationsList from "../utils/amortizations.json"

export default {
  name: "AmortizationView",
  setup() {
    // pagination
    const currentPage = ref(0)
    const itemsPerPage = ref(20)
    // data
    const amortizations = ref(amortizationsList)
    // filters
    const selectedStatus = ref("")
    const selectedProjectId = ref(0)

    // Computed

    const filteredAmortizations = computed(() => {
      // to manage filters
      let filtered = amortizations.value

      if (selectedStatus.value) {
        filtered = filtered.filter((a) => a.state === selectedStatus.value)
      }
      if (selectedProjectId.value) {
        filtered = filtered.filter(
          (a) => a.project_id === selectedProjectId.value
        )
      }

      return filtered
    })

    const maxPage = computed(
      // max page number based on the length of the amortizations list
      // used to disable nextpage button and show info to user
      () =>
        Math.ceil(filteredAmortizations.value.length / itemsPerPage.value) - 1
    )

    const displayedAmortizations = computed(() => {
      // to select the items to show per page
      const start = currentPage.value * itemsPerPage.value // 0 * 20 = 0
      const end = start + itemsPerPage.value // 0 + 20 = 20
      return filteredAmortizations.value.slice(start, end) // will return the first 20 items of the amortizations array
    })

    function formatDate(dateString) {
      // to show date in more user friendly way
      const options = {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "2-digit",
        minute: "2-digit",
        hour12: true,
      }
      return new Date(dateString).toLocaleString(undefined, options)
    }

    const nextPage = () => {
      if (currentPage.value < maxPage.value) {
        currentPage.value += 1
      }
    }

    const prevPage = () => {
      if (currentPage.value > 0) {
        currentPage.value -= 1
      }
    }

    // go to the first page whenever a filter changes.
    watch([selectedStatus, selectedProjectId], () => {
      currentPage.value = 0
    })

    // initialize list of sorted IDS
    // used to show options on filtering by id
    const projectIds = ref(
      [...new Set(amortizations.value.map((a) => a.project_id))].sort(
        (a, b) => a - b
      )
    )
    // Expose to template
    return {
      currentPage,
      maxPage,
      displayedAmortizations,
      nextPage,
      prevPage,
      projectIds,
      selectedStatus,
      selectedProjectId,
      formatDate,
    }
  },
}
</script>

<template>
  <h1 class="text-2xl mt-4 mb-8">Amortization payments view</h1>
  <div class="px-9 relative h-screen flex flex-col text-sm">
    <!-- Table -->
    <table
      class="min-w-full bg-white scroll-table flex-1 overflow-y-auto scroll-table"
    >
      <thead class="sticky top-0 bg-white">
        <tr class="border-b-2">
          <th class="py-1 px-1">State</th>
          <th class="py-1 px-1">Schedule Date</th>
          <th class="py-1 px-1">Amount</th>
          <th class="py-1 px-1">Project ID</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="amortization in displayedAmortizations"
          :key="amortization.schedule_date"
        >
          <td class="py-1 px-1 flex items-center">
            <span
              :class="[
                'w-4 h-4 rounded-full mr-2',
                amortization.state === 'paid' ? 'bg-green-500' : 'bg-gray-400',
              ]"
            ></span>
            {{ amortization.state }}
          </td>
          <td class="py-1 px-1">
            {{ formatDate(amortization.schedule_date) }}
          </td>
          <td class="py-1 px-1">$ {{ amortization.amount }}</td>
          <td class="py-1 px-1">{{ amortization.project_id }}</td>
        </tr>
      </tbody>
    </table>

    <!-- Footer pagination and filters -->
    <footer
      class="sticky bottom-0 flex justify-between items-center p-1 bg-gray-100"
    >
      <div class="ml-auto flex flex-row justify-between items-center">
        <select class="p-2 border rounded mr-2" v-model="selectedStatus">
          <option value="">Filter By State</option>
          <option value="paid">Paid</option>
          <option value="pending">Pending</option>
        </select>

        <select class="p-2 border rounded mr-6" v-model="selectedProjectId">
          <option value="">Filter By Project ID</option>
          <option
            v-for="projectId in projectIds"
            :key="projectId"
            :value="projectId"
          >
            {{ projectId }}
          </option>
        </select>

        <span class="mr-4">
          Page {{ currentPage + 1 }} of {{ maxPage + 1 }}
        </span>
        <button
          @click="prevPage"
          :disabled="currentPage <= 0"
          class="px-4 py-2 text-white bg-goparity-yellow disabled:opacity-50 rounded-lg m-2"
        >
          Previous
        </button>
        <button
          @click="nextPage"
          :disabled="currentPage >= maxPage"
          class="px-4 py-2 text-white bg-goparity-yellow disabled:opacity-50 rounded-lg m-2"
        >
          Next
        </button>
      </div>
    </footer>
  </div>
</template>

<style scoped>
h1 {
  color: #ffc84b;
  font-weight: bold;
}
.scroll-table {
  height: calc(100vh - 100px);
  overflow-y: auto;
}

thead {
  color: #154b64;
}
</style>
