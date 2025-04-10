<template>
  <v-card class="mx-auto">
    <v-text-field
      v-model="newItemTitle"
      label="Add new item"
      append-icon="mdi-plus"
      clearable
      outlined
      class="pa-4"
      @click:append="addNewItem"
      @keyup.enter="addNewItem"
    ></v-text-field>

    <v-list v-model:selected="settingsSelection">
      <template v-for="(item, index) in shoppingList" :key="item.id">
        <v-list-item
          :title="item.title"
          @click="doneBought(item.id)"
          :class="{ 'bg-blue-lighten-5': item.bought }"
        >
          <template v-slot:prepend>
            <v-list-item-action start>
              <v-checkbox-btn v-model="item.bought"></v-checkbox-btn>
            </v-list-item-action>
          </template>
          <template v-slot:title>
            <v-list-item-title :class="{ 'text-decoration-line-through': item.bought }">
              {{ item.title }}
            </v-list-item-title>
          </template>
          <template v-slot:append>
            <v-list-item-action>
              <v-btn icon @click.stop="deleteItem(item.id)">
                <i class="mdi mdi-delete"></i>
              </v-btn>
            </v-list-item-action>
          </template>
        </v-list-item>
        <v-divider v-if="index < shoppingList.length - 1"></v-divider>
      </template>
    </v-list>
  </v-card>
</template>

<script setup>
import { ref } from 'vue'

const shoppingList = ref([
  {
    id: 'groceries',
    title: 'Groceries',
    bought: false,
  },
  {
    id: 'electronics',
    title: 'Electronics',
    bought: false,
  },
  {
    id: 'clothing',
    title: 'Clothing',
    bought: false,
  },
]);
const newItemTitle = ref('')

const settingsSelection = ref([]);

const doneBought = (id) => {
  const item = shoppingList.value.find((item) => item.id === id);
  if (item) {
    item.bought = !item.bought;
  }
};

const deleteItem = (id) => {
  shoppingList.value = shoppingList.value.filter((item) => item.id !== id);
};

const addNewItem = () => {
  if (newItemTitle.value.trim() === '') return;
  const newId = Date.now().toString();
  shoppingList.value.push({
    id: newId,
    title: newItemTitle.value.trim(),
    bought: false,
  });
  newItemTitle.value = ''; // Clear the input after adding
};
</script>



