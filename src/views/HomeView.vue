<template>
  <div>
    <Form />
  </div>
</template>

<script>
import axios from 'axios';
import { API_BASE_URL } from "@/config.js";
import Form from '@/components/Form.vue';

export default {
  components: {
    Form,
  },
  data() {
    return {
      visibilidade: null, // Inicializa com null
    };
  },
  methods: {
    async validarVisibilidade() {
      try {
        const response = await axios.get(`${API_BASE_URL}/reuniao/`);
        this.visibilidade = response.data[0].visivel;
      } catch (error) {
        console.error('Erro ao validar visibilidade:', error);
      }
    },
  },
  async created() {
    await this.validarVisibilidade();

    // Atualiza a visibilidade a cada 30 segundos, por exemplo
    this.interval = setInterval(async () => {
      await this.validarVisibilidade();
    }, 1000);
  },
  beforeDestroy() {
    // Limpa o intervalo quando o componente for destruído
    clearInterval(this.interval);
  }
};
</script>

<style scoped>
.sem-visibilidade {
  width: 100%;
  height: 70vh;
  display: grid;
  place-items: center;
  color: #173B56;
}
</style>
