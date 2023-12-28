<template>
  <div class="search-box">
    <input
      ref="searchInput"
      class="search-input"
      type="text"
      placeholder="Enter a destination"
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
  mounted() {
    this.initSearchBox();
  },
  methods: {
    initSearchBox() {
      const input = this.$refs.searchInput;
      const searchBox = new google.maps.places.SearchBox(input);

      searchBox.addListener("places_changed", () => {
        const places = searchBox.getPlaces();
        if (places.length === 0) {
          return;
        }
        this.searchQuery = input.value; // Update searchQuery with the selected place
        this.$emit("place-selected", places[0].geometry.location);
      });
    },
    clearSearch() {
      this.searchQuery = "";
      this.$refs.searchInput.value = ""; // Clear the input field
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
