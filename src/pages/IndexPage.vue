<template>
  <q-page class="map-container">
    <div id="map"></div>
    <map-search-box
      v-if="map"
      :map="map"
      @place-selected="handlePlaceSelected"
      @search-cleared="clearSearch"
    />
    <!-- Route options UI -->
    <div v-if="routeOptions.length > 0" class="route-options">
      <div
        v-for="(option, index) in routeOptions"
        :key="index"
        @click="selectRoute(index)"
      >
        Route {{ index + 1 }}: {{ option.duration.text }} ({{
          option.distance.text
        }})
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref, reactive, onMounted } from "vue";
import MapSearchBox from "C:/Users/Alejandro Cedillo/OneDrive - Carbiz Rentals/Desktop/REAL MAP CODE/New-Map-Code/src/components/MapSearchBox.vue";

export default defineComponent({
  name: "IndexPage",
  components: {
    MapSearchBox,
  },
  setup() {
    const map = ref(null);
    let directionsRenderer = null;
    let lastDirectionsResponse = null;
    const routeOptions = reactive([]);
    let polylines = [];
    let selectedPolyline = null;

    onMounted(() => {
      initMap();
    });

    const initMap = () => {
      const defaultLocation = { lat: -34.397, lng: 150.644 };

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const currentLocation = {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            };
            map.value = createMap(currentLocation);
            addMarker(map.value, currentLocation);
          },
          () => {
            map.value = createMap(defaultLocation);
            addMarker(map.value, defaultLocation);
          }
        );
      } else {
        map.value = createMap(defaultLocation);
        addMarker(map.value, defaultLocation);
      }
    };

    const createMap = (location) => {
      const mapElement = document.getElementById("map");
      if (!mapElement) {
        console.error("Map element not found");
        return null;
      }

      const createdMap = new google.maps.Map(mapElement, {
        center: location,
        zoom: 15,
        mapTypeControl: false,
        fullscreenControl: false,
        zoomControl: false,
        streetViewControl: false,
        draggable: true,
      });

      const trafficLayer = new google.maps.TrafficLayer();
      trafficLayer.setMap(createdMap);

      return createdMap;
    };

    const addMarker = (map, location) => {
      new google.maps.Marker({
        position: location,
        map: map,
        title: "Your Location",
      });
    };

    const handlePlaceSelected = (destination) => {
      clearPolylines();
      if (directionsRenderer) {
        directionsRenderer.setMap(null);
      }

      directionsRenderer = new google.maps.DirectionsRenderer({
        map: map.value,
        polylineOptions: {
          strokeOpacity: 0.4,
        },
        suppressPolylines: true,
      });

      const directionsService = new google.maps.DirectionsService();
      directionsService.route(
        {
          origin: map.value.center,
          destination: destination,
          travelMode: google.maps.TravelMode.DRIVING,
          provideRouteAlternatives: true,
        },
        (response, status) => {
          if (status === google.maps.DirectionsStatus.OK) {
            lastDirectionsResponse = response;
            directionsRenderer.setDirections(response);
            routeOptions.splice(
              0,
              routeOptions.length,
              ...response.routes.map((r) => r.legs[0])
            );
            renderPolylines(response);
          } else {
            alert("Directions request failed due to " + status);
          }
        }
      );
    };

    const renderPolylines = (response) => {
      response.routes.forEach((route, index) => {
        const polyline = new google.maps.Polyline({
          path: route.overview_path,
          strokeColor: "#00BFFF", // Darker shade of light blue
          strokeOpacity: 0.6, // Slightly more opaque
          strokeWeight: 6,
          map: map.value,
        });
        polyline.addListener("click", () => selectRoute(index));
        polylines.push(polyline);
      });
    };

    const clearPolylines = () => {
      polylines.forEach((polyline) => polyline.setMap(null));
      polylines = [];
      if (selectedPolyline) {
        selectedPolyline.setMap(null);
        selectedPolyline = null;
      }
    };

    const selectRoute = (routeIndex) => {
      if (selectedPolyline) {
        selectedPolyline.setMap(null);
      }
      const selectedRoute = lastDirectionsResponse.routes[routeIndex];
      selectedPolyline = new google.maps.Polyline({
        path: selectedRoute.overview_path,
        strokeColor: "#0000FF", // Dark blue for selected route
        strokeOpacity: 1.0, // Non-transparent
        strokeWeight: 6,
        map: map.value,
      });

      if (lastDirectionsResponse) {
        directionsRenderer.setDirections(lastDirectionsResponse);
        directionsRenderer.setRouteIndex(routeIndex);
      }
    };

    const clearSearch = () => {
      clearPolylines();
      if (directionsRenderer) {
        directionsRenderer.setMap(null);
      }
      routeOptions.splice(0, routeOptions.length);
    };

    return {
      map,
      handlePlaceSelected,
      clearSearch,
      routeOptions,
      selectRoute,
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

.route-options {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  background-color: white;
  border-radius: 8px;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  text-align: center;
}
</style>
