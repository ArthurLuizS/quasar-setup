<template>
    <div>
      login !
      <div id="buttonDiv"></div>
    </div>
  </template>
  
  <script setup>
  import { onUnmounted, onMounted, ref } from 'vue'
  import { loadScript, removeScript } from '../utils/load-script'
  
  const scriptGoogle = 'https://accounts.google.com/gsi/client'
  const scriptJwt = 'https://unpkg.com/jwt-decode/build/jwt-decode.js'
  
  onMounted(() => {
    loadScript(scriptJwt)
    loadScript(scriptGoogle)
      .then(() => {
         // Retirado da pagina de documentação do google: https://developers.google.com/identity/gsi/web/guides/display-button?hl=pt-br#javascript 
        function handleCredentialResponse(response) {
          const decodedCredential = jwtDecode(response.credential)
          console.log(response)
          console.log(decodedCredential)
        }
  
        if (typeof google !== 'undefined' && google.accounts) { 
          google.accounts.id.initialize({
            client_id: "ID DA APLICAÇÃO NO GOOGLE",
            callback: handleCredentialResponse
          })
  
          // Renderiza o botão de login do Google
          google.accounts.id.renderButton(document.getElementById("buttonDiv"), {
            theme: "outline", 
            size: "large",
            type: "standard",
            shape: "pill",
            text: "continue_with",
            size: "medium",
            logo_alignment: "left",
          })

          google.accounts.id.prompt(); 
        }
      })
      .catch(error => {
        console.error('Erro ao carregar o script do Google:', error)
      })
  })
  
  onUnmounted(() => {
    removeScript(scriptGoogle)
  })
  </script>