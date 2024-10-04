<template>
  <div>
    <div id="map" class="tw-w-[600px] tw-h-[400px]"></div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import "leaflet-draw/dist/leaflet.draw.css";
import "leaflet/dist/leaflet.css";
import L, { polyline } from "leaflet";
import "leaflet-draw/dist/leaflet.draw.css";
import "leaflet-draw";

// Personalizações de mensagens

//botões
L.drawLocal.edit.handlers.edit.tooltip.text =
  "Clique em cancelar para desfazer alterações";
L.drawLocal.edit.handlers.edit.tooltip.subtext =
  "Arraste os pontos para editar";

L.drawLocal.edit.handlers.remove.tooltip.text =
  "Clique na linha ou marcador que deseja remover";

L.drawLocal.draw.toolbar.buttons = {
  polyline: "Desenhar Linha",
  marker: "Adicionar Ponto",
};

L.drawLocal.edit.toolbar.buttons.edit = "Editar";
//  Linha
L.drawLocal.draw.handlers.polyline.tooltip.start = "Clique no local de inicio";
L.drawLocal.draw.handlers.polyline.tooltip.cont =
  "Clique para adicionar novo ponto";
L.drawLocal.draw.handlers.polyline.tooltip.end =
  "Clique para adicionar novo ponto ou finalize";

// marcador
L.drawLocal.draw.handlers.marker.tooltip.start = "Clique no local desejado";

// Controle do modo de desenho
const isDrawingLine = ref(false);

onMounted(() => {
  const map = L.map("map").setView([-8.063151, -34.871127], 13);

  L.tileLayer(
    "https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png",
    {
      maxZoom: 19,
    }
  ).addTo(map);

  // Documentação: https://leaflet.github.io/Leaflet.draw/docs/leaflet-draw-latest.html#l-draw-toolbar

  var drawnItems = new L.FeatureGroup();
  map.addLayer(drawnItems);
  var drawControl = new L.Control.Draw({
    edit: {
      featureGroup: drawnItems,
    },
    //  desativar opções da barra de edição
    draw: {
      polygon: false,
      rectangle: false,
      circle: false,
      circlemarker: false,
      polyline: true,
      marker: true,
    },
  });
  map.addControl(drawControl);

  // Quando o modo de desenho de linha é ativado
  map.on(L.Draw.Event.DRAWSTART, function (e) {
    if (e.layerType === "polyline") {
      isDrawingLine.value = true;
      console.log("Modo de linha ativado");
    }
  });

  // Quando o modo de desenho é finalizado
  map.on(L.Draw.Event.DRAWSTOP, function () {
    isDrawingLine.value = false;
    console.log("Modo de linha desativado");
  });

  const markers = [];
  const marker1 = L.marker([-8.063151, -34.871127]).addTo(map);
  const marker2 = L.marker([-8.063151, -34.881127]).addTo(map);
  markers.push(marker1, marker2);

  markers.forEach((marker) => {
    marker.on("click", function () {
      if (isDrawingLine.value) {
        // Iniciar a linha a partir do marker clicado
        const markerPosition = marker.getLatLng();
        console.log(`Iniciando linha a partir do marcador: ${markerPosition}`);

        // Aqui você pode adicionar lógica para começar a linha
        // e fazer com que as coordenadas iniciais sejam as do marcador
      } else {
        // Outros modos podem ser definidos aqui
        console.log("Outro evento para o marcador");
      }
    });
  });

  // Criação de linha e marcador
  // Evento para captura de criação de polilinha
  map.on(L.Draw.Event.CREATED, function (e) {
    const layer = e.layer;
    drawnItems.addLayer(layer);

    if (e.layerType === "polyline") {
      console.log("Linha criada", e.layer.getLatLngs());
    }
    if (e.layerType === "marker") {
      console.log("Ponto adicionado");
    }
  });

  // edição:
  map.on("draw:edited", function (e) {
    console.log("editar");
    var layers = e.layers;
    layers.eachLayer(function (layer) {
      console.log(layer);
    });
  });
});
</script>

<style>
#map > div.leaflet-control-container > div.leaflet-bottom.leaflet-right > div {
  display: none;
}
</style>
