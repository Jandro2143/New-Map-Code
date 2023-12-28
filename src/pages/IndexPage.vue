<template>
  <q-page class="map-container">
    <div id="map"></div>
    <map-search-box
      v-if="map"
      :map="map"
      @place-selected="handlePlaceSelected"
      @search-cleared="clearSearch"
    />
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";
import MapSearchBox from "C:/Users/Alejandro Cedillo/OneDrive - Carbiz Rentals/Desktop/REAL MAP CODE/New-Map-Code/src/components/MapSearchBox.vue"; // Adjust the path as needed

export default defineComponent({
  name: "IndexPage",
  components: {
    MapSearchBox,
  },
  setup() {
    const map = ref(null);
    let directionsRenderer = null; // Reference to the directions renderer

    onMounted(() => {
      initMap();
    });

    const initMap = () => {
      const defaultLocation = { lat: -34.397, lng: 150.644 }; // Default location

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const currentLocation = {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            };
            console.log("Current Location:", currentLocation); // Debug statement
            map.value = createMap(currentLocation);
            addMarker(map.value, currentLocation);
          },
          () => {
            console.log("Geolocation error"); // Debug statement
            map.value = createMap(defaultLocation); // Fallback to default location
            addMarker(map.value, defaultLocation);
          }
        );
      } else {
        console.log("Geolocation not supported"); // Debug statement
        map.value = createMap(defaultLocation); // Geolocation not supported
        addMarker(map.value, defaultLocation);
      }
    };

    const createMap = (location) => {
      const mapElement = document.getElementById("map");
      console.log("Map Element:", mapElement); // Debug statement

      if (!mapElement) {
        console.error("Map element not found"); // Debug statement
        return null;
      }

      return new google.maps.Map(mapElement, {
        center: location,
        zoom: 15,
        mapTypeControl: false, // Disable the map button
        fullscreenControl: false, // Disable the expand button
      });
    };

    const addMarker = (map, location) => {
      if (!map) {
        console.error("Map is null, cannot add marker"); // Debug statement
        return;
      }

      new google.maps.Marker({
        position: location,
        map: map,
        title: "Your Location",
      });
    };

    const handlePlaceSelected = (destination) => {
      if (directionsRenderer) {
        directionsRenderer.setMap(null); // Remove previous directions from the map
      }

      directionsRenderer = new google.maps.DirectionsRenderer();
      directionsRenderer.setMap(map.value);

      const directionsService = new google.maps.DirectionsService();
      directionsService.route(
        {
          origin: map.value.center,
          destination: destination,
          travelMode: google.maps.TravelMode.DRIVING,
        },
        (response, status) => {
          if (status === google.maps.DirectionsStatus.OK) {
            directionsRenderer.setDirections(response);
          } else {
            alert("Directions request failed due to " + status);
          }
        }
      );
    };

    const clearSearch = () => {
      if (directionsRenderer) {
        directionsRenderer.setMap(null); // Remove directions when clearing search
      }
    };

    return {
      map,
      handlePlaceSelected,
      clearSearch,
    };
  },
});
</script>

<style>
.map-container {
  width: 100%;
  height: 80vh;
}

#map {
  width: 100%;
  height: 100%;
}
</style>
