<template>
  <div>
    <div id="map" class="tw-w-[600px] tw-h-[400px]"></div>
  </div>
</template>

<script setup>
import { onMounted } from "vue";
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

  // Criação de linha e marcador
  map.on(L.Draw.Event.CREATED, function (e) {
    var type = e.layerType,
      //  Não sei por que a IDE avisa de depreciado se foi retirado da documentação
      layer = e.layer;
    if (type === "polyline") {
      console.log("Linha desenhada!");
    }
    if (type === "marker") {
      console.log("Adicionado ponto");
    }
    drawnItems.addLayer(layer);
    map.addLayer(layer);
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
