<template>
  <v-sheet border rounded>
    <v-data-table :headers="headers" :hide-default-footer="filteredStudents.length < 11"
                  :items="filteredStudents">
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>
            <v-icon color="medium-emphasis" icon="mdi-account-group" size="x-small" start></v-icon>
            Students
          </v-toolbar-title>

          <!-- Search Bar -->
          <v-text-field
            v-model="search"
            prepend-inner-icon="mdi-magnify"
            label="Search students"
            single-line
            hide-details
            clearable
            class="mx-4"
            style="max-width: 300px;"
          ></v-text-field>

          <v-btn
            class="me-2"
            prepend-icon="mdi-plus"
            rounded="lg"
            text="Add a student"
            border
            @click="add"
          ></v-btn>
        </v-toolbar>
      </template>

      <template v-slot:item.actions="{ item }">
        <div class="d-flex ga-2 justify-end">
          <v-icon
            color="medium-emphasis"
            icon="mdi-pencil"
            size="small"
            @click="edit(item.fullName)"
          ></v-icon>

          <v-icon
            color="medium-emphasis"
            icon="mdi-delete"
            size="small"
            @click="confirmDelete(item.fullName)"
          ></v-icon>
        </div>
      </template>

      <template v-slot:no-data>
        <v-btn
          prepend-icon="mdi-backup-restore"
          rounded="lg"
          text="Reset data"
          variant="text"
          border
          @click="reset"
        ></v-btn>
      </template>
    </v-data-table>
  </v-sheet>

  <v-dialog v-model="dialog" max-width="500">
    <v-card
      :subtitle="`${isEditing ? 'Update' : 'Create'} student`"
      :title="`${isEditing ? 'Edit' : 'Add'} a Student`"
    >
      <template v-slot:text>
        <v-row>
          <v-col cols="12">
            <v-text-field
              prepend-icon="mdi-account"
              v-model="record.fullName"
              label="Full Name"
            ></v-text-field>
          </v-col>

          <v-col cols="12">
            <!-- Compact Date Picker -->
            <v-menu
              v-model="dateMenu"
              :close-on-content-click="false"
              transition="scale-transition"
            >
              <template v-slot:activator="{ props }">
                <v-text-field
                  v-model="record.dateOfBirth"
                  label="Date of Birth"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-bind="props"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="datePickerValue"
                @update:model-value="updateDateOfBirth"
                no-title
                scrollable
              ></v-date-picker>
            </v-menu>
          </v-col>

          <v-col cols="12">
            <v-text-field
              prepend-icon="mdi-phone"
              v-model="record.mobilePhone"
              label="Mobile Phone"
            ></v-text-field>
          </v-col>

          <v-col cols="12">
            <v-text-field
              prepend-icon="mdi-email"
              v-model="record.email"
              label="Email"
            ></v-text-field>
          </v-col>
        </v-row>
      </template>

      <v-divider></v-divider>

      <v-card-actions class="bg-surface-light">
        <v-btn text="Cancel" variant="plain" @click="dialog = false"></v-btn>

        <v-spacer></v-spacer>

        <v-btn text="Save" @click="save"></v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <!-- Delete Confirmation Dialog -->
  <v-dialog v-model="deleteDialog" max-width="400">
    <v-card>
      <v-card-title class="text-h6"> Confirm Deletion </v-card-title>
      <v-card-text>
        Are you sure you want to delete <strong>{{ studentToDelete }}</strong
        >? This action cannot be undone.
      </v-card-text>
      <v-card-actions>
        <v-btn text="Cancel" variant="plain" @click="deleteDialog = false"></v-btn>
        <v-spacer></v-spacer>
        <v-btn color="error" text="Delete" @click="removeConfirmed"></v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { onMounted, ref, shallowRef, computed } from 'vue'
import { useDate } from 'vuetify'

const DEFAULT_RECORD = { fullName: '', dateOfBirth: '', mobilePhone: '', email: '' }
const students = ref([])
const record = ref(DEFAULT_RECORD)
const dialog = shallowRef(false)
const isEditing = shallowRef(false)
// Date picker
const adapter = useDate();
const dateMenu = ref(false) // Controls the date picker menu
const datePickerValue = ref(null) // Temporary storage for date picker value
// Delete confirmation state
const deleteDialog = ref(false)
const studentToDelete = ref('')
// Search input
const search = ref('')

const headers = [
  { title: 'Full Name', key: 'fullName', align: 'start' },
  { title: 'Date of Birth', key: 'dateOfBirth' },
  { title: 'Mobile phone', key: 'mobilePhone' },
  { title: 'Email', key: 'email', align: 'end' },
  { title: 'Actions', key: 'actions', align: 'end', sortable: false },
]

const filteredStudents = computed(() => {
  if (!search.value) return students.value
  const searchLower = search.value.toLowerCase()
  return students.value.filter(
    (student) =>
      student.fullName.toLowerCase().includes(searchLower) ||
      student.dateOfBirth.toLowerCase().includes(searchLower) ||
      student.mobilePhone.toLowerCase().includes(searchLower) ||
      student.email.toLowerCase().includes(searchLower),
  )
})

// Manual parsing function for "MM/DD/YYYY" to Date object
function parseDate(dateStr) {
  if (!dateStr) return null;
  const [month, day, year] = dateStr.split('/').map(Number);
  return new Date(year, month - 1, day); // Month is 0-based in JS Date
}

onMounted(() => {
  reset()
})

function add() {
  isEditing.value = false
  record.value = DEFAULT_RECORD
  datePickerValue.value = null
  dialog.value = true
}

function edit(fullName) {
  isEditing.value = true

  const found = students.value.find((student) => student.fullName === fullName)

  record.value = { ...found }

  datePickerValue.value = found.dateOfBirth ? parseDate(found.dateOfBirth) : null;
  dialog.value = true
}

function confirmDelete(fullName) {
  studentToDelete.value = fullName
  deleteDialog.value = true
}

function removeConfirmed() {
  const index = students.value.findIndex((student) => student.fullName === studentToDelete.value)
  if (index !== -1) {
    students.value.splice(index, 1)
  }
  deleteDialog.value = false
  studentToDelete.value = ''
}

function updateDateOfBirth(date) {
  if (date) {
    // Manually format to MM/DD/YYYY
    const d = new Date(date);
    const month = String(d.getMonth() + 1).padStart(2, '0'); // Months are 0-based
    const day = String(d.getDate()).padStart(2, '0');
    const year = d.getFullYear();
    record.value.dateOfBirth = `${month}/${day}/${year}`;
  } else {
    record.value.dateOfBirth = '';
  }
  dateMenu.value = false; // Close the menu after selection
}

function save() {
  if (isEditing.value) {
    const index = students.value.findIndex((student) => student.fullName === record.value.fullName)
    students.value[index] = record.value
  } else {
    record.value.id = students.value.length + 1
    students.value.push(record.value)
  }

  dialog.value = false
}

function reset() {
  dialog.value = false
  record.value = DEFAULT_RECORD
  students.value = [
    { fullName: 'Іван Петренко', dateOfBirth: '12/04/1998', mobilePhone: '0501234567', email: 'ivan.petrenko@gmail.com' },
    { fullName: 'Олена Коваленко', dateOfBirth: '06/20/1997', mobilePhone: '0977654321', email: 'olena.kovalenko@ukr.net' },
    { fullName: 'Андрій Шевченко', dateOfBirth: '03/09/1995', mobilePhone: '0631122334', email: 'ashevchenko@mail.com' },
    { fullName: 'Марія Іванова', dateOfBirth: '10/11/1999', mobilePhone: '0669988776', email: 'm.ivanova@example.com' },
    { fullName: 'Сергій Бондар', dateOfBirth: '08/25/1996', mobilePhone: '0934433221', email: 'serhii.bondar@domain.com' },
    { fullName: 'Катерина Романюк', dateOfBirth: '01/17/2000', mobilePhone: '0685556677', email: 'k.romaniuk@edu.ua' },
  ]
  search.value = ''
}
</script>

<style scoped>
  .mx-4 {
    margin-left: 16px;
    margin-right: 16px;
  }
</style>