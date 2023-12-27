<template>
  <q-page class="map-container">
    <div id="map"></div>
    <div class="search-bar">
      <input id="destination-search" placeholder="Enter a destination" />
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";

export default defineComponent({
  name: "IndexPage",
  setup() {
    const map = ref(null);
    const currentLocation = ref({ lat: -34.397, lng: 150.644 }); // Default location

    onMounted(() => {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            currentLocation.value = {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            };
            initMap();
          },
          () => {
            console.warn("Geolocation failed. Using default location.");
            initMap();
          }
        );
      } else {
        console.warn("Geolocation is not supported by this browser.");
        initMap();
      }
    });

    const initMap = () => {
      map.value = new google.maps.Map(document.getElementById("map"), {
        center: currentLocation.value,
        zoom: 8,
      });
      const directionsService = new google.maps.DirectionsService();
      const directionsRenderer = new google.maps.DirectionsRenderer();
      directionsRenderer.setMap(map.value);

      const input = document.getElementById("destination-search");
      const searchBox = new google.maps.places.SearchBox(input);
      map.value.controls[google.maps.ControlPosition.TOP_LEFT].push(input);

      searchBox.addListener("places_changed", () => {
        const places = searchBox.getPlaces();
        if (places.length === 0) return;
        const destination = places[0].geometry.location;

        directionsService.route(
          {
            origin: currentLocation.value,
            destination: destination,
            travelMode: google.maps.TravelMode.DRIVING,
          },
          (response, status) => {
            if (status === google.maps.DirectionsStatus.OK) {
              directionsRenderer.setDirections(response);
            } else {
              window.alert("Directions request failed due to " + status);
            }
          }
        );
      });
    };

    return {
      map,
    };
  },
});
</script>

<style>
.map-container {
  width: 100%;
  height: 90vh;
}

#map {
  width: 100%;
  height: 100%;
}

.search-bar {
  position: absolute;
  top: 20px; /* Adjust this as needed */
  left: 50%;
  transform: translate(-50%, 0);
  z-index: 5;
  background-color: white;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  width: 400px;
}

#destination-search {
  width: 100%;
  border: 1px solid #ddd;
  padding: 12px 15px;
  border-radius: 4px;
  box-sizing: border-box;
}
</style>
