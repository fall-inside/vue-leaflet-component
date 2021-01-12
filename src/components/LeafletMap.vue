<template>
  <v-col cols="auto">
    <v-toolbar
        class="btns-toolbar"
        flat
        dense>
      <v-btn-toggle
        dense
        group
        mandatory
      >
        <v-btn>
          <v-icon>mdi-shape-polygon-plus</v-icon>
        </v-btn>
        <v-btn>
          <v-icon>mdi-shape-square-plus</v-icon>
        </v-btn>
        <v-btn>
          <v-icon>mdi-shape-circle-plus</v-icon>
        </v-btn>
      </v-btn-toggle>
      <v-col
          cols="12"
          sm="2"
          md="1"
      >
        <v-text-field
            v-if="circleDrawing"
            v-model="circleRadius"
            type="number"
            hide-details
            single-line
        ></v-text-field>
      </v-col>
      <v-btn
        icon
      >
        <v-icon>
          mdi-content-save
        </v-icon>
      </v-btn>
      <v-btn
          icon
      >
        <v-icon>
          mdi-eraser
        </v-icon>
      </v-btn>

    </v-toolbar>
    <div class="leaflet-map">
      <l-map
        :zoom="zoom"
        :center="center"
        :options="mapOptions"
        @update:center="centerUpdate"
        @update:zoom="zoomUpdate"
      >
        <l-tile-layer
          :url="url"
        />
        <l-marker
          v-for="(item, index) in markers"
          :key="'marker-' + index"
          :lat-lng="item.location"
          :icon="getIcon(item)">
        </l-marker>
      </l-map>
    </div>
  </v-col>
</template>

<script>
import { latLng } from 'leaflet'
import { LMap, LTileLayer, LMarker } from 'vue2-leaflet'

export default {
  name: 'LeafletMap',
  components: {
    LMap,
    LTileLayer,
    LMarker
  },
  data () {
    return {
      zoom: 13,
      center: latLng(47.41322, -1.219482),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      currentZoom: 11.5,
      currentCenter: latLng(47.41322, -1.219482),
      mapOptions: {
        zoomSnap: 0.5
      },
      mapModificator: 0,
      markers: [],
      showRadiusEditor: false
    }
  },
  methods: {
    addMarker (markerInfo) {
      console.debug(markerInfo);
    },

    getIcon (iconName) {
      console.debug(iconName);
    },

    zoomUpdate (zoom) {
      this.currentZoom = zoom
    },

    centerUpdate (center) {
      this.currentCenter = center
    }
  }
}
</script>

<style scoped>
 .leaflet-map {
   width: 600px;
   height: 600px;
 }

 .btns {
   flex-direction: column;
 }
</style>
