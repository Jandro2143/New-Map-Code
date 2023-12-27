<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title> Quasar App </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>

    <q-footer elevated>
      <div class="q-gutter-md">
        <div class="quick-links">
          <template v-for="(link, index) in essentialLinks" :key="index">
            <EssentialLink v-bind="link" />
            <!-- Add a separator div if it's not the last link -->
            <div
              v-if="index < essentialLinks.length - 1"
              class="separator"
            ></div>
          </template>
        </div>
      </div>
    </q-footer>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import EssentialLink from "components/EssentialLink.vue";

const linksList = [
  {
    title: "Explore",
    icon: "school",
    link: "https://quasar.dev",
  },
  {
    title: "Go",
    icon: "code",
    link: "https://github.com/quasarframework",
  },
  {
    title: "Saved",
    icon: "chat",
    link: "https://chat.quasar.dev",
  },
];

export default defineComponent({
  name: "MainLayout",

  components: {
    EssentialLink,
  },

  setup() {
    const leftDrawerOpen = ref(false);

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
    };
  },
});
</script>

<style scoped>
/* Custom CSS to style the Quick Links section */
.quick-links {
  display: flex;
  justify-content: center;
  align-items: center;
}

.quick-links .essential-link {
  margin-right: 10px; /* Adjust the margin as needed */
  padding-right: 10px; /* Add padding to separate link text from border */
  border-right: 1px solid #ccc; /* Add a border on the right */
}

.quick-links .essential-link:last-child {
  border-right: none; /* Remove border for the last link */
}

/* Style for the separator div */
.separator {
  width: 1px;
  height: 15px; /* Adjust the height as needed */
  background-color: #ccc; /* Separator color */
  margin: 0 10px; /* Adjust the margin to control the spacing */
}
</style>
