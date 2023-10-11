<script>
import { ref, computed, watch } from "vue"
import amortizationsList from "../utils/amortizations.json"

export default {
  name: "AmortizationView",
  methods: {
    formatDate(dateString) {
      const options = {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "2-digit",
        minute: "2-digit",
        hour12: true,
      }
      return new Date(dateString).toLocaleString(undefined, options)
    },
  },
  setup() {
    // State
    // const currentPage = ref(0)
    // const itemsPerPage = ref(20)
    // const amortizations = ref(amortizationsList)

    // // Computed properties
    // const maxPage = computed(
    //   () => Math.ceil(amortizations.value.length / itemsPerPage.value) - 1
    // )
    // const displayedAmortizations = computed(() => {
    //   const start = currentPage.value * itemsPerPage.value
    //   const end = start + itemsPerPage.value
    //   return amortizations.value.slice(start, end)
    // })

    // // Methods
    // const nextPage = () => {
    //   if (currentPage.value < maxPage.value) {
    //     currentPage.value += 1
    //   }
    // }

    // const prevPage = () => {
    //   if (currentPage.value > 0) {
    //     currentPage.value -= 1
    //   }
    // }

    // return {
    //   currentPage,
    //   maxPage,
    //   displayedAmortizations,
    //   nextPage,
    //   prevPage,
    // }
    const currentPage = ref(0) // pagination
    const itemsPerPage = ref(20) // pagination
    const amortizations = ref(amortizationsList) // data
    const selectedStatus = ref("") // filter
    const selectedProjectId = ref(0) // filter

    const filteredAmortizations = computed(() => {
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
      () =>
        Math.ceil(filteredAmortizations.value.length / itemsPerPage.value) - 1
    )

    const displayedAmortizations = computed(() => {
      const start = currentPage.value * itemsPerPage.value
      const end = start + itemsPerPage.value
      return filteredAmortizations.value.slice(start, end)
    })

    // Reset currentPage whenever a filter changes.
    watch([selectedStatus, selectedProjectId], () => {
      currentPage.value = 0
    })
    // Computed properties
    // const maxPage = computed(
    //   () => Math.ceil(amortizations.value.length / itemsPerPage.value) - 1
    // )

    // const displayedAmortizations = computed(() => {

    //   const start = currentPage.value * itemsPerPage.value
    //   const end = start + itemsPerPage.value

    //   let filteredAmortizations = amortizations.value

    //   // Filtering logic can be applied here, e.g.:
    //   if (selectedStatus.value) {
    //     filteredAmortizations = filteredAmortizations.filter(
    //       (a) => a.state === selectedStatus.value
    //     )
    //   }
    //   if (selectedProjectId.value) {
    //     console.log(selectedProjectId.value)
    //     filteredAmortizations = filteredAmortizations.filter(
    //       (a) => a.project_id === selectedProjectId.value
    //     )
    //   }

    //   return filteredAmortizations.slice(start, end)
    // })

    // Methods
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

    const filterByStatus = (event) => {
      selectedStatus.value = event.target.value
    }

    const filterByProjectId = (event) => {
      selectedProjectId.value = Number(event.target.value)
    }

    const projectIds = ref([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]) // Initialize with project IDs.

    // Expose to template
    return {
      currentPage,
      maxPage,
      displayedAmortizations,
      nextPage,
      prevPage,
      filterByStatus,
      filterByProjectId,
      projectIds,
      selectedStatus,
      selectedProjectId,
    }
  },
}
</script>

<template>
  <h1 class="text-2xl mt-4 mb-8">Amortization payments view</h1>
  <div class="px-9 relative h-screen flex flex-col text-sm">
    <!-- Table -->
    <transition name="fade">
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
                  amortization.state === 'paid'
                    ? 'bg-green-500'
                    : 'bg-gray-400',
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
    </transition>

    <!-- Footer agination and filters -->
    <footer
      class="sticky bottom-0 flex justify-between items-center p-1 bg-gray-100"
    >
      <div class="ml-auto flex flex-row justify-between items-center">
        <select
          class="p-2 border rounded mr-2"
          @change="filterByStatus($event)"
        >
          <option value="">Filter By State</option>
          <option value="paid">Paid</option>
          <option value="pending">Pending</option>
        </select>

        <select
          class="p-2 border rounded mr-6"
          @change="filterByProjectId($event)"
        >
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
