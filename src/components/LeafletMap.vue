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
            v-model.number="circle.radius"
            type="number"
            hide-details
            single-line
        ></v-text-field>
      </v-col>
      <v-btn
          v-if="mapModifier === 3 && dotMarkers.length !== 0"
          icon
          @click="finishPolygon"
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
      <v-simple-checkbox
          v-model="isDrawForArea"
          label="Отображать маркеры только для выделенной области"
      ></v-simple-checkbox>
    </v-toolbar>
    <div class="leaflet-map">
      <l-map
          :center="center"
          :options="mapOptions"
          :zoom="zoom"
          @click="drawArea"
          @update:bounds="mapBoundsUpdate"
          @update:center="mapCenterUpdate"
          @update:zoom="mapZoomUpdate"
      >
        <l-tile-layer
          :url="url"
        />
        <!--    markers    -->
        <l-marker
            v-for="(item, index) in markers"
            :key="'marker-' + index"
            :lat-lng="item.latlng"
            :icon="cellphoneIcon"
        />
        <!--    markers for drawing    -->
        <l-marker
          v-for="(item, index) in dotMarkers"
          :key="'marker-' + index"
          :lat-lng="item.latlng"
          :icon="dotIcon"
        />
        <l-circle
            :lat-lng="circle.center"
            :radius="circle.radius"
            :color="circle.style.color"
            :weight="circle.style.weight"
        />
        <l-polygon
            :lat-lngs="polygon.latlngs"
            :color="polygon.style.color"
            :weight="polygon.style.weight"
        ></l-polygon>
        <l-rectangle
            :bounds="rect.bounds"
            :color="rect.style.color"
            :weight="rect.style.weight"
        ></l-rectangle>
      </l-map>
    </div>
  </v-col>
</template>

<script>
import { latLng, divIcon } from 'leaflet'
import { LMap, LTileLayer, LCircle, LMarker, LPolygon, LRectangle } from 'vue2-leaflet'

const queryTimeout = 1000

export default {
  name: 'LeafletMap',
  components: {
    LMap,
    LTileLayer,
    LCircle,
    LMarker,
    LPolygon,
    LRectangle
  },

  props: {
    // show toolbar
    showToolbar: {
      type: Boolean,
    }
  },

  data () {
    return {
      // map
      zoom: 13,
      center: latLng(47.41322, -1.219482),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      currentZoom: 11.5,
      currentCenter: latLng(47.41322, -1.219482),
      mapOptions: {
        zoomSnap: 0.5
      },

      // circle area
      circle: {
        center: [0, 0],
        radius: 100,
        style: { color: 'red', weight: 0 }
      },

      // icons
      cellphoneIcon: divIcon({
        iconSize: [30, 30],
        iconAnchor: [15, 26],
        className: "marker",
        html: `<svg version="1.1" id="cellphone" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                viewBox="0 0 512 512" style="enable-background:new 0 0 512 512;" xml:space="preserve">
                  <g>
                      <path d="M352,0H160c-35.296,0-64,28.704-64,64v384c0,35.296,28.704,64,64,64h192c35.296,0,64-28.704,64-64V64
                              C416,28.704,387.296,0,352,0z M208,32h96c8.832,0,16,7.168,16,16c0,8.832-7.168,16-16,16h-96c-8.832,0-16-7.168-16-16
                              C192,39.168,199.168,32,208,32z M192,448h-32v-32h32V448z M192,384h-32v-32h32V384z M272,448h-32v-32h32V448z M272,384h-32v-32h32
                              V384z M352,448h-32v-32h32V448z M352,384h-32v-32h32V384z M384,288H128V96h256V288z"/>
                  </g>
                </svg>`
      }),
      baseStationIcon: divIcon({
        iconSize: [30, 30],
        iconAnchor: [15, 26],
        className: "marker",
        html: `<svg version="1.1" id="cell_tower" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                viewBox="0 0 512 512" style="enable-background:new 0 0 512 512;" xml:space="preserve">
                  <g>
                      <path d="M375.405,499.121l-40.474-137.969c-0.002-0.008-0.005-0.016-0.007-0.025l-14.443-49.232
                              c-0.001-0.005-0.003-0.01-0.004-0.014l-19.958-68.033c-0.004-0.013-0.007-0.026-0.011-0.038l-31.754-108.244
                              c13.978-5.193,23.965-18.664,23.965-34.427c0-20.244-16.47-36.713-36.713-36.713s-36.713,16.47-36.713,36.713
                              c0,15.763,9.987,29.235,23.965,34.427l-66.17,225.563c-0.002,0.008-0.005,0.016-0.007,0.025l-40.474,137.968
                              c-1.562,5.324,1.488,10.907,6.812,12.47c5.331,1.561,10.907-1.488,12.47-6.813l6.154-20.978l93.964-62.067l93.964,62.067
                              l6.154,20.978c1.285,4.381,5.292,7.222,9.638,7.222c0.936,0,1.889-0.132,2.832-0.409
                              C373.917,510.029,376.967,504.446,375.405,499.121z M239.387,101.137c0-9.163,7.455-16.618,16.618-16.618
                              s16.618,7.455,16.618,16.618s-7.455,16.618-16.618,16.618S239.387,110.301,239.387,101.137z M256.005,163.498l21.492,73.265
                              h-42.985L256.005,163.498z M228.617,256.858h54.776l14.071,47.968h-82.918L228.617,256.858z M170.805,453.928l21.742-74.112
                              l45.229,29.875L170.805,453.928z M198.465,359.642l10.186-34.721h94.709l10.186,34.721l-57.541,38.008L198.465,359.642z
                              M274.235,409.692l45.229-29.875l21.742,74.112L274.235,409.692z"
                      />
                  </g>
                  <g>
                      <path d="M208.24,53.373c3.924-3.924,3.924-10.286,0-14.209c-3.924-3.924-10.285-3.924-14.21,0
                              tc-34.169,34.172-34.169,89.775,0.001,123.947c1.962,1.962,4.533,2.943,7.105,2.943c2.571,0,5.142-0.982,7.104-2.943
                              c3.924-3.924,3.924-10.286,0-14.209C181.902,122.565,181.902,79.71,208.24,53.373z"/>
                  </g>
                  <g>
                      <path d="M317.979,39.165c-3.924-3.924-10.284-3.924-14.21,0c-3.924,3.924-3.924,10.286,0,14.209
                              c26.337,26.337,26.337,69.192,0,95.529c-3.923,3.922-3.923,10.285,0.001,14.208c1.962,1.962,4.533,2.943,7.105,2.943
                              s5.142-0.982,7.105-2.943C352.15,128.939,352.15,73.336,317.979,39.165z"/>
                  </g>
                  <g>
                      <path d="M172.02,17.152c3.924-3.924,3.924-10.286,0-14.209c-3.924-3.924-10.285-3.924-14.21,0
                              c-26.228,26.229-40.673,61.101-40.673,98.195c0,37.093,14.445,71.966,40.673,98.195c1.963,1.962,4.533,2.943,7.106,2.943
                              s5.142-0.982,7.105-2.943c3.924-3.924,3.924-10.286,0-14.209C125.709,138.813,125.709,63.462,172.02,17.152z"/>
                  </g>
                  <g>
                      <path d="M374.322,28.38c-2.91-4.724-9.1-6.196-13.825-3.286c-4.723,2.911-6.194,9.1-3.285,13.824
                              c28.95,46.994,21.867,107.116-17.222,146.204c-3.924,3.924-3.924,10.286,0,14.21c1.962,1.962,4.533,2.943,7.105,2.943
                              c2.571,0,5.142-0.981,7.105-2.943c22.222-22.223,36.365-51.631,39.823-82.806C397.426,85.834,390.43,54.529,374.322,28.38z"/>
                  </g>
                  <g>
                      <path d="M347.095,0c-12.93,0-12.951,20.095,0,20.095C360.025,20.095,360.046,0,347.095,0z"/>
                  </g>
                </svg>`
      }),
      dotIcon: divIcon({
        iconSize: [12, 12],
        iconAnchor: [6, 6],
        className: "marker",
        html: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 6 6" height="12" width="12">
                    <circle cx="3" cy="3"  r="3" style="fill:red; fill-opacity:0.8;" />
              </svg>`
      }),

      // array for rect and polygon dots
      dotMarkers:[],

      isDrawForArea: false, // false - show markers everywhere, true - show markers in selected area
      isDrawing: false,
      lastQueryTimeStamp: 0,
      mapModifier: 0, // 1 - rect, 2 - circle, 3 - polygon
      noSelectedArea: true,

      // polygon area
      polygon: {
        latlngs: [],
        style: { color: 'red', weight: 0 }
      },

      // rect area
      rect: {
        bounds: [[0, 0], [0, 0]],
        style: { color: 'red', weight: 0 }
      },

      showRadiusEditor: false,
      toggledBtn: undefined
    }
  },
  watch: {
    /**
     * Наблюдает за выбором фигуры для рисования на тулбаре
     */
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

    /**
     * Метод очистки карты от всех нарисованных на ней объектов
     */
    clearMapArea () {
      console.debug('Remove all drawings');

      this.dotMarkers = [];
      this.isDrawing = false;
      this.mapModifier = 0;
      this.noSelectedArea = true;
      this.toggledBtn = undefined;

      // clear circle
      this.circle.center = [0, 0];
      this.circle.radius = 0;
      this.circle.style.weight = 0;

      // clear rect
      for (let i = 0; i < this.rect.bounds.length; i++) {
        this.rect.bounds.splice(i, 1, [0, 0]);
      }
      this.rect.style.weight = 0;

      // clear polygon
      this.polygon.latlngs = [];
    },

    /**
     * Метод рисования на карте.
     * Для прямоугольника сначала рисует две вершины, а затем - сам прямоугольник.
     * Для полигона рисует массив точек, которые можно соединить в фигуру.
     * Для окружности - рисует окружность с заданным радиусом и центром в точке клика
     * @param event - событие клика мыши на карте, которое содержит в себе координаты, необходимые для отрисовки
     */
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
              for (let i = 0; i < this.dotMarkers.length; i++) {
                this.rect.bounds.splice(i, 1, [this.dotMarkers[i].latlng.lat, this.dotMarkers[i].latlng.lng]);
              }
              this.rect.style.weight = 2;
              this.dotMarkers = [];
              this.noSelectedArea = false;

              // signal for getting markers
              this.emit("showMarkersForRect",this.rect);
            }
          }
          break;
        case 2: {
            this.circle.center = [marker.latlng.lat, marker.latlng.lng];
            this.circle.style.weight = 2;
            this.noSelectedArea = false;

            // signal for getting markers
            this.emit("showMarkersForCircle",this.circle);
          }
          break;
        case 3: {
            this.dotMarkers.push(marker);
          }
          break;
      }
    },


    // create polygon based on dotMarkers
    /**
     * Метод объединяет отображенные на карте вершины полигона
     */
    finishPolygon () {
      let latlngArray = [];
      for (let i = 0; i < this.dotMarkers.length; i++) {
        latlngArray.push([this.dotMarkers[i].latlng.lat, this.dotMarkers[i].latlng.lng]);
      }
      this.polygon.latlngs.push(latlngArray);
      this.polygon.style.weight = 2;
      this.dotMarkers = [];
      this.noSelectedArea = false;

      // signal for getting markers
      this.emit("showMarkersForPolygon",this.polygon);
    },

    /**
     * Метод очищает карту от других фигур и определяет начало отрисовки окружности
     */
    startDrawCircle () {
      console.debug('Start draw circle');
      if (this.isDrawing) {
        this.clearMapArea();
      }
      this.isDrawing = true;
      this.mapModifier = 2;
    },

    /**
     * Метод очищает карту от других фигур и определяет начало отрисовки полигона
     */
    startDrawPolygon () {
      console.debug('Start draw polygon');
      if (this.isDrawing) {
        this.clearMapArea();
      }
      this.isDrawing = true;
      this.mapModifier = 3;
    },

    /**
     * Метод очищает карту от других фигур и определяет начало отрисовки прямоугольника
     */
    startDrawRectangle () {
      console.debug('Start draw rect');
      if (this.isDrawing) {
        this.clearMapArea();
      }
      this.isDrawing = true;
      this.mapModifier = 1
    },

    //MAP EVENTS

    /**
     * Метод получения границ видимой области карты.
     * @param bounds
     */
    mapBoundsUpdate (bounds) {
      if (Date.now() - this.lastQueryTimeStamp < queryTimeout) {
        return;
      }
      if (!this.noSelectedArea && this.isDrawForArea) {
        return;
      }

      // signal for getting markers
      this.emit("showAllMarkers", bounds);
      this.lastQueryTimeStamp = Date.now();
    },

    mapCenterUpdate (center) {
      this.currentCenter = center
    },

    mapZoomUpdate (zoom) {
      this.currentZoom = zoom
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
