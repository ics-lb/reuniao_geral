<template lang="">
    <div>
        
    </div>
</template>
<script>
export default {
    
}
</script>
<style lang="">
    
</style>

<!-- <template>
    <div id="jaas-container" style="height: 100%;"></div>
  </template>
  
  <script>
  export default {
    name: 'JitsiMeet',
    mounted() {
      // Inicialize a API do Jitsi quando o componente for montado
      this.initJitsiMeet();
    },
    methods: {
      initJitsiMeet() {
        // Certifique-se de que a API externa foi carregada
        const script = document.createElement('script');
        script.src = 'https://8x8.vc/vpaas-magic-cookie-64b5867425374a14a07984a4999fc704/external_api.js';
        script.async = true;
        script.onload = () => {
          const domain = "8x8.vc";
          const options = {
            roomName: "vpaas-magic-cookie-64b5867425374a14a07984a4999fc704/SampleAppScaryRefusalsLastToday",
            parentNode: document.querySelector('#jaas-container'),
            // Incluir JWT se necessário (opcional)
            // jwt: "SEU_JWT_TOKEN",
          };
          const api = new JitsiMeetExternalAPI(domain, options);
          console.log("Jitsi API inicializada", api);
        };
        document.body.appendChild(script);
      }
    }
  }
  </script>
  
  <style scoped>
  #jaas-container {
    height: 100%;
    width: 100%;
  }
  </style>
   -->