<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer" app>
      <v-list class="py-0">
        <v-list-item v-for="item in items" :key="item.title" :to="item.to">
            <v-icon>{{ item.icon }}</v-icon>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar
      color="teal-darken-4"
      image="https://picsum.photos/1920/1080?random"
      app
    >
      <template v-slot:image>
        <v-img
          gradient="to top right, rgba(19,84,122,.8), rgba(128,208,199,.8)"
        ></v-img>
      </template>

      <template v-slot:prepend>
        <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
      </template>

      <v-app-bar-title>Application</v-app-bar-title>

      <v-btn icon>
        <v-icon>mdi-magnify</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>mdi-heart</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>mdi-dots-vertical</v-icon>
      </v-btn>
    </v-app-bar>
    <v-main>
      <router-view></router-view>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'App',
  data: () => ({
    drawer: false,
    items: [
      { title: 'List', icon: 'mdi-format-list-checks', to: '/' },
      { title: 'About', icon: 'mdi-help-box', to: '/about' },
      { title: 'MyGroup', icon: 'mdi-account-group', to: '/mygroup' },
    ],
    right: null as null | boolean,
  }),
});
</script>

<style scoped>
/* Hide scrollbar for Webkit browsers (Chrome, Safari, Edge) */
v-main::-webkit-scrollbar {
  width: 0; /* Removes the scrollbar width */
}

/* Ensure the content is still scrollable */
v-main {
  overflow-y: auto; /* Enables vertical scrolling */
  height: 100vh; /* Ensures the main area takes full viewport height */
}

/* Optional: Hide scrollbar for Firefox */
v-main {
  scrollbar-width: none; /* Firefox */
}

/* Optional: Hide scrollbar for IE/Edge (older versions) */
v-main {
  -ms-overflow-style: none; /* IE and Edge */
}
</style>