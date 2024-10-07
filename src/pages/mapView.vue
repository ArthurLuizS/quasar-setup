<template>
  <q-page>
    <div style="height: calc(100vh - 50px); width: 100%">
      <l-map
        ref="map"
        style="height: 100%; width: 100%"
        v-model:zoom="zoom"
        v-model:center="center"
        @ready="onMapReady"
        @click="onMapClick"
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
        <!-- <l-marker
          v-for="marker in pontos"
          :key="marker.hostId"
          :icon="getIcon(marker)"
          :lat-lng="filterLatLog(marker)"
          @click="showDetail(marker)"
        >
          <l-tooltip>{{ marker.name }}</l-tooltip>
        </l-marker> -->

        <!-- <div v-if="showLines">
          <l-polyline
            v-for="link in polyLines"
            :key="link.id"
            :lat-lngs="link.latLong"
            :color="link.color"
          >
          </l-polyline>
        </div> -->
        <!-- </l-marker-cluster-group> -->
      </l-map>
      <q-dialog persistent v-model="showDetailModal">
        <q-card
          class="full-width full-height"
          :style="!maxDialog ? 'max-height: 600px; max-width: 1000px' : ''"
        >
          Teste
          <q-bar>
            <q-btn
              dense
              flat
              icon="close"
              v-close-popup
              @click="graphData = null"
            >
              <q-tooltip content-class="bg-white text-primary"
                >Fechar</q-tooltip
              >
            </q-btn>
          </q-bar>
        </q-card>
      </q-dialog>
      <q-dialog persistent v-model="showDialog">
        <q-card
          class="full-width full-height"
          :style="!maxDialog ? 'max-height: 600px; max-width: 1000px' : ''"
        >
          Criar um marker no mapa
          <q-bar>
            <q-btn
              dense
              flat
              icon="close"
              v-close-popup
              @click="graphData = null"
            >
              <q-tooltip content-class="bg-white text-primary"
                >Fechar</q-tooltip
              >
            </q-btn>
          </q-bar>
          <q-btn @click="criarMarker" label="novo ponto" />
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import L, { icon } from "leaflet";
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
import "leaflet.markercluster/dist/MarkerCluster.css";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";
import "leaflet.markercluster";
import "leaflet-draw/dist/leaflet.draw.css";
import "leaflet-draw";

import IconNormal from "src/assets/quasar-logo-vertical.svg";
import IconNotClassified from "src/assets/quasar-logo-vertical.svg";
import IconInformation from "src/assets/quasar-logo-vertical.svg";
import IconWarning from "src/assets/quasar-logo-vertical.svg";
import IconMajor from "src/assets/quasar-logo-vertical.svg";
import IconCritical from "src/assets/quasar-logo-vertical.svg";
import IconDisaster from "src/assets/quasar-logo-vertical.svg";

const zoom = ref(4);
const center = ref([-10, -50]);
const markers = L.markerClusterGroup();
const showLines = ref(true);
const polyLines = ref([
  {
    id: "123",
    latLong: [
      [
        [-8.131432, -34.938023], // Recife
        [-7.996206, -34.838986], // Olinda
      ],
    ],
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

const layer = ref(
  new L.TileLayer("http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
);

const showDialog = ref(false);

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
  console.log("abrir algo");
  console.log(marker);
  tab.value = "detail";
  lastMarker.value = marker;
  showDetailModal.value = true;
  graphId.value = null;
  graphRange.value = 1;
};

const pontos = [
  {
    id: "1",
    name: "Recife",
    hostId: "123",
    latitude: "-8.131432",
    longitude: "-34.938023",
  },
  {
    id: "2",
    name: "Olinda",
    hostId: "1234",
    latitude: "-7.996206",
    longitude: "-34.838986",
  },
  {
    id: "3",
    name: "Camaragibe",
    hostId: "12345",
    latitude: "-8.022041",
    longitude: "-34.979277",
  },
];

const map = ref();

const loadingMarkers = () => {
  markers.clearLayers();
  // Adiciona os pontos como marcadores ao cluster
  pontos.forEach((ponto) => {
    const latLng = [parseFloat(ponto.latitude), parseFloat(ponto.longitude)];
    const marker = L.marker(latLng, {
      icon: getIcon(ponto),
      ponto,
    }).bindTooltip(ponto.name);
    markers.addLayer(marker); // Adiciona cada marcador ao grupo
  });
  markers.on("click", function (a) {
    showDetail(a.layer.options.ponto);
    console.log(a.layer.options.ponto);
  });
  // Adiciona o cluster ao mapa quando o mapa estiver pronto
  map.value.addLayer(markers);
};

const onMapReady = (leafletMap) => {
  map.value = leafletMap; // Atribui a instância do mapa do Leaflet
  loadingMarkers(); // Carrega os marcadores
  map.value.addLayer(markers); // Adiciona o cluster ao mapa

  const drawnItems = new L.FeatureGroup();
  map.value.addLayer(drawnItems);

  polyLines.value.forEach((polylineObj) => {
    const polylineCoords = polylineObj.latLong;
    const polyline = L.polyline(polylineCoords, {
      color: polylineObj.color, // Usa a cor definida no objeto
      weight: 4,
    }).addTo(map.value);
    drawnItems.addLayer(polyline); // Adiciona ao grupo de itens editáveis
  });

  const drawControl = new L.Control.Draw({
    draw: {
      polyline: {
        shapeOptions: {
          color: "#f357a1", // Cor da linha
          weight: 5, // Espessura da linha
        },
      },
      polygon: false, // Desabilita polígonos, por exemplo
      circle: false,
      rectangle: false,
      marker: true,
    },
    edit: {
      featureGroup: drawnItems, // Grupo de itens que podem ser editados
      edit: {
        selectedPathOptions: {
          maintainColor: true, // Mantém a cor da linha durante a edição
          opacity: 0.5,
        },
      },
      remove: true, // Habilita remoção
    },
  });

  map.value.addControl(drawControl);

  // Evento ao finalizar o desenho
  map.value.on(L.Draw.Event.CREATED, (event) => {
    const layer = event.layer;
    drawnItems.addLayer(layer); // Adiciona a linha desenhada ao grupo
    console.log("Nova linha desenhada:", layer.getLatLngs()); // Exibe as coordenadas
  });

  // Evento ao editar uma linha
  map.value.on(L.Draw.Event.EDITED, (event) => {
    const layers = event.layers;
    layers.eachLayer((layer) => {
      console.log("Linha editada:", layer.getLatLngs()); // Exibe as coordenadas da linha editada
    });
  });

  map.value.on("draw:drawstart", (event) => {
    toggleDrawingMode();
    const type = event.layerType; // Tipo de desenho ativado
    console.log(`Modo de desenho ativado para: ${type}`);
  });

  // Evento para detectar o término do modo de desenho
  map.value.on("draw:drawstop", (event) => {
    toggleDrawingMode();
    const type = event.layerType; // Tipo de desenho desativado
    console.log(`Modo de desenho desativado para: ${type}`);
  });
};

const clickedLatLng = ref();
const isDrawingMode = ref(false);
const onMapClick = (e) => {
  // Captura a latitude e longitude do clique no mapa
  if (!isDrawingMode.value) {
    clickedLatLng.value = e.latlng;
    console.log(clickedLatLng.value);

    // Abre o dialog
    showDialog.value = true;
  }
};

const toggleDrawingMode = () => {
  isDrawingMode.value = !isDrawingMode.value;
};

const criarMarker = () => {
  if (!isDrawingMode.value && clickedLatLng.value) {
    const latLng = clickedLatLng.value;
    const newMarker = {
      id: Date.now(),
      name: `Ponto ${pontos.length + 1}`,
      latitude: latLng.lat,
      longitude: latLng.lng,
    };
    pontos.push(newMarker);
    setTimeout(() => {
      loadingMarkers(); // Recarrega os marcadores
    }, 5000);
  }
};
</script>
