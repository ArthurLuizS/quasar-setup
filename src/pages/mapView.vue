<template>
  <q-page>
    <div style="height: calc(100vh - 50px); width: 100%">
      <l-map
        ref="map"
        style="height: 100%; width: 100%"
        v-model:zoom="zoom"
        v-model:center="center"
      >
        <l-control-layers position="topright" />
        <l-control position="bottomleft">
          <q-card>
            <q-card-section class="q-py-none">
              <q-checkbox left-label v-model="showLines" label="Linhas" />
            </q-card-section>
          </q-card>
        </l-control>
        <!-- <l-control-polyline-measure
          :options="{ showUnitControl: true }"
          position="topleft"
        /> -->
        <l-tile-layer
          ref="layer"
          v-for="tileProvider in tileProviders"
          :key="tileProvider.name"
          :name="tileProvider.name"
          :visible="tileProvider.visible"
          :url="tileProvider.url"
          :attribution="tileProvider.attribution"
          layer-type="base"
        />
        <!-- <l-mini-map :options="options" :layer="layer"></l-mini-map> -->
        <!-- <l-marker-cluster-group> -->
        <l-marker
          v-for="marker in markers"
          :key="marker.hostId"
          :icon="getIcon(marker)"
          :lat-lng="filterLatLog(marker)"
          @click="showDetail(marker)"
        >
          <l-tooltip>{{ marker.name }}</l-tooltip>
        </l-marker>

        <div v-if="showLines">
          <l-polyline
            v-for="link in polyLines"
            :key="link.id"
            :lat-lngs="link.latLong"
            :color="link.color"
          ></l-polyline>
        </div>
        <!-- </l-marker-cluster-group> -->
      </l-map>
    </div>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import {
  LMap,
  LTileLayer,
  LControl,
  LControlLayers,
  LTooltip,
  LMarker,
  LPolyline,
} from "@vue-leaflet/vue-leaflet";

const zoom = ref(4);
const center = ref([-10, -50]);
const markers = ref([
  {
    name: "Arthur 1",
    hostId: "123",
  },
]);
const showLines = ref(true);
const polyLines = ref([
  {
    id: "123",
    latLong: [],
    color: "red",
  },
]);
const tileProviders = ref([
  {
    name: "Claro",
    visible: false,
    attribution:
      '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
    url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
  },
  {
    name: "Escuro",
    visible: false,
    url: "https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png",
    attribution:
      '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
  },
  {
    name: "Simplificado",
    visible: true,
    url: "https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png",
    attribution:
      '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
  },
]);
const options = ref({
  position: "bottomright",
  width: 200,
  height: 175,
});

const layer = ref(
  new L.TileLayer("http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
);

const getIcon = (marker) => {
  if (marker.priority === 0) {
    return L.icon({
      iconUrl: IconNotClassified,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  } else if (marker.priority === 1) {
    return L.icon({
      iconUrl: IconInformation,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  } else if (marker.priority === 2) {
    return L.icon({
      iconUrl: IconWarning,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  } else if (marker.priority === 3) {
    return L.icon({
      iconUrl: IconMajor,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  } else if (marker.priority === 4) {
    return L.icon({
      iconUrl: IconCritical,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  } else if (marker.priority === 5) {
    return L.icon({
      iconUrl: IconDisaster,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  } else {
    return L.icon({
      iconUrl: IconNormal,
      iconSize: [30, 40],
      iconAnchor: [15, 40],
    });
  }
};

const filterLatLog = (mark) => {
  return [mark.latitude, mark.longitude];
};

const tab = ref();
const lastMarker = ref();
const showDetailModal = ref();
const graphId = ref();
const graphRange = ref();
const showDetail = (marker) => {
  tab.value = "detail";
  lastMarker.value = marker;
  showDetailModal.value = true;
  graphId.value = null;
  graphRange.value = 1;
};
</script>
