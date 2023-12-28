<template>
  <div class="search-box">
    <input
      ref="searchInput"
      class="search-input"
      type="text"
      placeholder="Enter a destination"
      v-model="searchQuery"
    />
    <button v-if="searchQuery" @click="clearSearch">Clear</button>
  </div>
</template>

<script>
export default {
  name: "MapSearchBox",
  props: {
    map: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      searchQuery: "",
    };
  },
  watch: {
    map: {
      immediate: true,
      handler(map) {
        if (map && this.$refs.searchInput) {
          this.initSearchBox(map);
        }
      },
    },
  },
  methods: {
    initSearchBox(map) {
      const input = this.$refs.searchInput;
      const searchBox = new google.maps.places.SearchBox(input);
      map.controls[google.maps.ControlPosition.TOP_LEFT].push(input);

      searchBox.addListener("places_changed", () => {
        const places = searchBox.getPlaces();
        if (places.length === 0) {
          return;
        }
        this.$emit("place-selected", places[0].geometry.location);
      });
    },
    clearSearch() {
      this.searchQuery = "";
      this.$emit("search-cleared");
    },
  },
};
</script>

<style>
.search-box {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 5;
  display: flex;
  align-items: center;
}

.search-input {
  width: 300px;
  padding: 10px;
}

button {
  padding: 5px 10px;
  margin-left: 5px;
  cursor: pointer;
}
</style>
