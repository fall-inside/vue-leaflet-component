<template>
  <v-col cols="auto">
    <v-toolbar
        class="btns-toolbar"
        flat
        dense>
      <v-btn-toggle
          v-model = "toggledBtn"
          dense
          group
      >
        <v-btn>
          <v-icon small> mdi-shape-square-plus </v-icon>
        </v-btn>
        <v-btn>
          <v-icon small> mdi-shape-circle-plus </v-icon>
        </v-btn>
        <v-btn>
          <v-icon small> mdi-shape-polygon-plus </v-icon>
        </v-btn>
      </v-btn-toggle>
      <v-col
          v-if="mapModifier === 2"
          cols="12"
          sm="2"
          md="1"
      >
        <v-text-field
            v-model="circleRadius"
            type="number"
            hide-details
            single-line
        ></v-text-field>
      </v-col>
      <v-btn
          v-if="isDrawing"
          icon
      >
        <v-icon small> mdi-check </v-icon>
      </v-btn>
      <v-btn
        icon
      >
        <v-icon small> mdi-content-save </v-icon>
      </v-btn>
      <v-btn
          icon
          @click="clearMapArea"
      >
        <v-icon small> mdi-eraser </v-icon>
      </v-btn>

    </v-toolbar>
    <div class="leaflet-map">
      <l-map
        :zoom="zoom"
        :center="center"
        :options="mapOptions"
        @click="drawArea"
        @update:center="centerUpdate"
        @update:zoom="zoomUpdate"

      >
        <l-tile-layer
          :url="url"
        />
        <l-marker
          v-for="(item, index) in dotMarkers"
          :key="'marker-' + index"
          :lat-lng="item.latlng"
          :icon="dotIcon"
        />
        <l-rectangle :bounds="mapArea" :l-style="mapAreaStyle"></l-rectangle>
      </l-map>
    </div>
  </v-col>
</template>

<script>
import { latLng, divIcon } from 'leaflet'
import { LMap, LTileLayer, LMarker, LRectangle } from 'vue2-leaflet'

export default {
  name: 'LeafletMap',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LRectangle
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
      circleRadius: 0,
      dotIcon: divIcon({
        iconUrl: 'circle-icon.png',
        iconSize: [16, 16],
        className: "dot-marker",
        html: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 6 6" height="12" width="12">
                    <circle cx="3" cy="3"  r="3" style="fill:red; fill-opacity:0.8;" />
              </svg>`
      }),
      dotMarkers:[],
      isDrawing: false,
      mapArea: [],
      mapAreaStyle: { color: 'red', weight: 3 },
      mapModifier: 0, // 1 - rect, 2 - circle, 3 - polygon
      toggledBtn: undefined,
      markers: [],
      showRadiusEditor: false
    }
  },
  watch: {
    toggledBtn: function () {
      console.debug(this.toggledBtn);
      if (this.toggledBtn === undefined) {
        console.debug('no btns clicked');
        this.clearMapArea();
      } else {
        this.mapModifier = this.toggledBtn + 1;
        switch (this.toggledBtn) {
          case 0: this.startDrawRectangle();
            break;
          case 1: this.startDrawCircle();
            break;
          case 2: this.startDrawPolygon();
            break;
        }
      }

    }
  },

  methods: {
    addMarker (markerInfo) {
      console.debug(markerInfo);
    },

    clearMapArea () {
      console.debug('Remove all drawings');
      this.dotMarkers = [];
      this.isDrawing = false;
      this.mapModifier = 0;
      this.mapArea = [];
      this.circleRadius = 0;
    },

    drawArea (event) {
      if (!this.isDrawing) {
        return;
      }
      console.debug('Draw on the map, coord: ' + event.latlng);
      let marker = {
                      latlng: event.latlng,
                      icon: this.dotIcon
                   };
      switch (this.mapModifier) {
        case 1: {
            if (this.dotMarkers.length >= 2) {
              this.dotMarkers = [];
            }
            this.dotMarkers.push(marker);

            // if there are 2 dots draw rect and remove dots
            if (this.dotMarkers.length === 2) {
              this.dotMarkers.forEach(item => this.mapArea.push(item.latlng));
              console.debug(this.mapArea);
            }
          }
          break;
        // case 2: {
        //
        // }
      }
    },

    getIcon (iconName) {
      console.debug(iconName);
    },

    startDrawCircle () {
      console.debug('Start draw circle');
      if (this.isDrawing) {
        this.clearMapArea();
      }
      this.isDrawing = true;
      this.mapModifier = 2;
    },

    startDrawPolygon () {
      console.debug('Start draw polygon');
      if (this.isDrawing) {
        this.clearMapArea();
      }
      this.isDrawing = true;
      this.mapModifier = 3;
    },

    startDrawRectangle () {
      console.debug('Start draw rect');
      if (this.isDrawing) {
        this.clearMapArea();
      }
      this.isDrawing = true;
      this.mapModifier = 1
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
 .dot-marker {
   background-color: transparent;
 }
</style>
