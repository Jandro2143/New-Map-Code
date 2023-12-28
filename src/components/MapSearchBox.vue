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
  padding-right: 30px; /* Space for the clear button */
  background-color: white;
  border-radius: 20px; /* Rounded edges */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2); /* Optional: adds subtle shadow */
}

.search-input {
  width: 300px;
  padding: 10px;
  padding-right: 40px; /* Increased right padding to prevent text overlap */
  border: none;
  border-radius: 20px; /* Rounded edges */
  outline: none; /* Removes the default focus outline */
  overflow: hidden; /* Prevent text overflow */
  white-space: nowrap; /* No wrapping of text */
  text-overflow: ellipsis; /* Add ellipsis for overflowed text */
}

button {
  position: absolute;
  right: 10px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  font-size: 16px;
  color: #666; /* Adjust color as needed */
}

button:hover {
  color: #333; /* Darker color on hover */
}

/* Hide the button when searchQuery is empty */
button[disabled] {
  display: none;
}

/* Optional: Add a transition effect for hover */
.search-input,
button {
  transition: all 0.3s ease;
}
</style>
