<template>
    <div>
        <div id="map" class="tw-w-[600px] tw-h-[400px]"></div>
    </div>
</template>

<script setup>
import { onMounted } from 'vue'
import 'leaflet-draw/dist/leaflet.draw.css'
import 'leaflet/dist/leaflet.css'
import L from 'leaflet'
import 'leaflet-draw/dist/leaflet.draw.css'
import 'leaflet-draw'

onMounted(() => {
  const map = L.map('map').setView([-8.063151, -34.871127], 13);

  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);

  const drawnItems = new L.FeatureGroup()
  map.addLayer(drawnItems)

    // Configurando o controle de desenho
    const drawControl = new L.Control.Draw({
    edit: {
      featureGroup: drawnItems // Define a camada editável
    },
    draw: {
      polygon: false, // Desativar polígonos se necessário
      rectangle: false, // Desativar retângulos se necessário
      circle: false, // Desativar círculos se necessário
      marker: false // Desativar marcadores se necessário
    }
  })
  map.addControl(drawControl)

  // Evento quando o usuário desenha uma polilinha
  map.on(L.Draw.Event.CREATED, (event) => {
    const layer = event.layer
    drawnItems.addLayer(layer) // Adiciona a polilinha desenhada à camada
    console.log(layer.getLatLngs())
  })
})
</script>