<template>
  <v-app>
    <v-main>
      <v-container fluid class="pa-6" style="padding-bottom: 0;">
        <div class="scroll-wrapper">

          <!-- 🏫 College Name Row -->
          <div class="d-flex align-center px-4 py-2 header-row">
            <div class="label-col">College Name</div>
            <div v-for="(college, index) in collegeNames" :key="'college-' + index"
              class="college-col d-flex align-center">
              <v-text-field v-model="collegeNames[index]" placeholder="College" variant="outlined" density="compact"
                hide-details class="school-input" />
              <v-btn v-if="index >= 3" icon size="small" color="red" class="ml-1" @click="removeCollege(index)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </div>

            <!-- ➕ Add Button Column (aligned like others) -->
            <div class="college-col d-flex align-center">
              <v-btn icon size="small" color="primary" @click="addCollege">
                <v-icon>mdi-plus</v-icon>
              </v-btn>
            </div>
          </div>

          <!-- 📘 Section Title Row -->
          <div class="d-flex px-4 py-2 section-title">
            <div class="label-col">Annual Costs</div>
            <div v-for="(_, index) in collegeNames.length + 1" :key="'section-' + index" class="college-col"></div>
          </div>

          <!-- 💸 Cost Input Rows -->
          <div v-for="(label, rowIndex) in costLabels" :key="'row-' + rowIndex" class="d-flex align-center px-4 py-2"
            :class="rowIndex % 2 === 0 ? 'bg-light' : 'bg-lighter'">
            <div class="label-col">{{ label }}</div>
            <div v-for="(college, colIndex) in collegeNames" :key="'input-' + rowIndex + '-' + colIndex"
              class="college-col">
              <v-text-field v-model="costMatrix[rowIndex][colIndex]" variant="outlined" density="compact" hide-details
                class="cost-input no-spinner" type="number" inputmode="numeric"
                :ref="el => setInputRef(rowIndex, colIndex, el)" @keydown.tab.prevent="handleTab(rowIndex, colIndex)" />
            </div>

            <!-- Empty cell to align with + column -->
            <div class="college-col"></div>
          </div>

          <!-- 🧮 Total Cost Row -->
          <div class="d-flex align-center px-4 py-2 bg-total font-weight-bold">
            <div class="label-col">Total Cost</div>
            <div v-for="(total, index) in totalCosts" :key="'total-' + index" class="college-col">
              ${{ total.toLocaleString() }}
            </div>

            <!-- Empty cell for + button column -->
            <div class="college-col"></div>
          </div>

        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, computed } from 'vue'

// Columns (colleges)
const collegeNames = ref(['', '', ''])

// Row labels
const costLabels = [
  'Tuition & Fees',
  'Food & Housing',
  'Books & Supplies',
  'Health Insurance',
  'Other Costs',
]

// Matrix of values
const costMatrix = ref(
  Array.from({ length: costLabels.length }, () => Array(collegeNames.value.length).fill(''))
)

// Add new column
const addCollege = () => {
  collegeNames.value.push('')
  costMatrix.value.forEach(row => row.push(''))
}

// Remove a column
const removeCollege = (index) => {
  collegeNames.value.splice(index, 1)
  costMatrix.value.forEach(row => row.splice(index, 1))
}

// Tab handling
const inputRefs = ref([])

const setInputRef = (row, col, el) => {
  if (!inputRefs.value[row]) inputRefs.value[row] = []
  inputRefs.value[row][col] = el
}

const handleTab = (row, col) => {
  const nextRow = row + 1
  if (nextRow < costLabels.length) {
    inputRefs.value[nextRow]?.[col]?.focus()
  } else {
    const nextCol = col + 1
    if (nextCol < collegeNames.value.length) {
      inputRefs.value[0]?.[nextCol]?.focus()
    }
  }
}

// Total cost calculation
const totalCosts = computed(() =>
  collegeNames.value.map((_, colIndex) =>
    costMatrix.value.reduce((sum, row) => {
      const val = parseFloat(row[colIndex])
      return sum + (isNaN(val) ? 0 : val)
    }, 0)
  )
)
</script>

<style scoped>
/* 🔄 Scroll container */
.scroll-wrapper {
  overflow-x: auto;
  overflow-y: hidden;
  white-space: nowrap;
  max-width: 100%;
  border: 1px solid #ccc;
}

/* 📐 Make background stretch fully */
.header-row,
.section-title,
.bg-light,
.bg-lighter,
.bg-total {
  min-width: max-content;
  width: fit-content;
}

/* 🧱 Column sizing */
.label-col {
  width: 200px;
  min-width: 200px;
  font-weight: bold;
}

.college-col {
  width: 200px;
  min-width: 200px;
  padding-right: 8px;
}

/* 🎨 Zebra striping */
.bg-light {
  background-color: #fcfdff;
}

.bg-lighter {
  background-color: #f2f4fb;
}

.bg-total {
  background-color: #e0e7ff;
}

/* 🎨 Header + section title */
.header-row {
  background-color: #4B5A94;
  color: white;
}

.section-title {
  background-color: #5E6AA8;
  color: white;
}

/* ✏️ Input appearance */
.school-input input {
  color: white;
}

.school-input input::placeholder {
  color: black;
  opacity: 1;
}

.school-input,
.cost-input {
  background-color: #f9faff;
  border-radius: 6px;
}

/* 🔕 Remove number spinners */
.no-spinner input::-webkit-outer-spin-button,
.no-spinner input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.no-spinner input[type="number"] {
  -moz-appearance: textfield;
}
</style>
