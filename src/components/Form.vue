<template>
    <div class="container justify-content-center mt-5">
        <h1 class="text-center mt-5">REUNIÃO GERAL - LABO</h1>
        <div class="d-flex justify-content-center mt-4">
            <div class="formulario w-100" style="max-width: 600px;">
                <form class="mb-4">
                    <h4 class="text-center">Confirme a sua presença:</h4>
                    <div class="d-grid">
                        <div class="m-3">
                            <label for="inputCPF" class="form-label">Qual é o seu CPF?</label>
                            <input id="inputCPF" class="form-control"
                                placeholder="Por favor, insira o seu CPF sem pontos." maxlength="14" required
                                v-model="cpf" @blur="formatCpf" @input="validateCpf" autocomplete="off" />
                            <span v-if="cpf && !cpfIsValid" style="color: red">CPF inválido!<br /></span>
                        </div>
                        <div class="m-3">
                            <label for="inputNome" class="form-label">Qual é o seu nome completo?</label>
                            <input type="text" class="form-control"
                                placeholder="Por favor, insira o seu nome completo, sem abreviar." id="inputNome"
                                v-model="nome_completo" autocomplete="off">
                        </div>
                    </div>
                    <div class="text-center">
                        <label class="form-label">Agora, por favor, tire uma foto sua:</label>
                        <div class="video-container mt-3" style="display: none;">
                            <video ref="video" width="320" height="240" autoplay
                                style="display: block; margin: 0 auto;"></video>
                            <div id="countdown" class="countdown">5</div>
                        </div>
                    </div>
                    <div class="text-center">
                        <button type="button" class="btn btn-secondary" id="snapshotBtn" @click="startCountdown">
                            <img class="w-25" src="../components/icons/camera.png" alt="camera">
                        </button>
                    </div>
                    <div>
                        <canvas ref="canvas" width="320" height="240" style="display: none;"></canvas>
                        <div id="results" class="m-3"></div>
                    </div>
                    <div class="text-center m-4">
                        <button type="button" class="btn btn-primary w-50" :class="{ 'btn-disabled': !fotoTirada }"
                            :disabled="!fotoTirada" @click="salvarPresenca()">
                            Enviar
                        </button>
                    </div>
                </form>
            </div>

            <!-- Modal de sucesso -->
            <ModalSucessoReuniao :showModal.sync="isSuccessModalVisible"
                @update:showModal="updateSuccessModalVisibility" :link="link" />

            <!-- Modal de erro -->
            <ModalErro :showModal.sync="isModalVisible" :errorMessage="errorMessage" title="Atenção!"
                @update:showModal="updateErrorModalVisibility" />
        </div>
    </div>

</template>

<script>
import axios from "axios";
import { API_BASE_URL } from "@/config.js";
import ModalErro from '@/components/ModalErro.vue';
import ModalSucessoReuniao from '@/components/ModalSucessoReuniao.vue';

export default {
    components: {
        ModalErro,
        ModalSucessoReuniao
    },
    data() {
        return {
            fotoTirada: false,
            timeoutAnterior: null,
            cpf: "",
            nome_completo: "",
            mes_referencia: undefined,
            ano_referencia: undefined,
            cpfIsValid: false,
            imagemCapturada: null,
            isModalVisible: false,
            errorMessage: '',
            isSuccessModalVisible: false,
            link: undefined,
            data_hora: undefined,
        }
    },
    methods: {
        async getMesAno() {
            try {
                const date = new Date();
                const day = String(date.getDate()).padStart(2, '0');
                const month = String(date.getMonth() + 1).padStart(2, '0');
                const year = date.getFullYear();
                const hours = String(date.getHours()).padStart(2, '0');
                const minutes = String(date.getMinutes()).padStart(2, '0');

                this.mes_referencia = month;
                this.ano_referencia = year;
                this.data_hora = new Date();
            } catch (error) {
                this.errorMessage = error.response?.data.errors[0] || 'Ocorreu um erro desconhecido.';
                this.isModalVisible = true;
            }
        },
        async getLink() {
            try {
                const response = await axios.get(`${API_BASE_URL}/reuniao/`);
                this.link = response.data[0].link;
            } catch (error) {
                this.errorMessage = error.response.data.errors[0] || 'Ocorreu um erro desconhecido.';
                this.isModalVisible = true;
            }
        },
        async salvarPresenca() {
            try {
                this.getMesAno();
                // Verifica se os campos estão preenchidos
                if (!this.nome_completo || !this.cpf || !this.imagemCapturada) {
                    this.errorMessage = "Por favor, preencha todos os campos.";
                    this.isModalVisible = true;
                    return null;
                }

                if (!this.cpfIsValid) {
                    this.errorMessage = "O CPF digitado é inválido."
                    this.isModalVisible = true;
                    return null;
                }

                // Remove a formatação do CPF
                const cpfSemFormatacao = this.cpf.replace(/\D/g, '');

                // Cria um objeto FormData para enviar os dados incluindo o arquivo de imagem e localização
                const formData = new FormData();
                formData.append('cpf_colaborador', cpfSemFormatacao);
                formData.append('nome_completo', this.nome_completo);
                formData.append('foto', this.imagemCapturada);
                formData.append('mes_referencia', this.mes_referencia);
                formData.append('ano_referencia', this.ano_referencia);
                formData.append('data_hora', this.data_hora);

                const response = await axios.post(`${API_BASE_URL}/formulario/`, formData, {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                });

                // Verifica o status da resposta
                if (response.status === 200) {
                    this.isSuccessModalVisible = true; // Define o modal de sucesso como visível
                } else {
                    this.errorMessage = "Erro ao cadastrar presença. Por favor, tente novamente.";
                    this.isModalVisible = true;
                }
            } catch (error) {
                this.errorMessage = error.response.data.errors[0] || 'Ocorreu um erro desconhecido.';
                this.isModalVisible = true;
            }
        },
        validateCpf() {
            // Remove caracteres não numéricos
            let cpf = this.cpf.replace(/\D/g, "");

            // Verifica se o CPF é inválido por comprimento ou se é uma sequência repetida
            const invalidCpfs = new Set([
                "00000000000", "11111111111", "22222222222", "33333333333",
                "44444444444", "55555555555", "66666666666", "77777777777",
                "88888888888", "99999999999"
            ]);

            if (cpf.length !== 11 || invalidCpfs.has(cpf)) {
                this.cpfIsValid = false;
                return;
            }

            // Função para calcular o dígito verificador
            const calculateCheckDigit = (cpf, length) => {
                let sum = 0;
                for (let i = 0; i < length; i++) {
                    sum += parseInt(cpf.charAt(i)) * (length + 1 - i);
                }
                const checkDigit = 11 - (sum % 11);
                return checkDigit >= 10 ? 0 : checkDigit;
            };

            // Validação do primeiro e segundo dígitos verificadores
            const firstCheckDigit = calculateCheckDigit(cpf, 9);
            const secondCheckDigit = calculateCheckDigit(cpf, 10);

            if (firstCheckDigit !== parseInt(cpf.charAt(9)) || secondCheckDigit !== parseInt(cpf.charAt(10))) {
                this.cpfIsValid = false;
            } else {
                this.cpfIsValid = true;
            }
        },
        formatCpf() {
            // Remove caracteres não numéricos e formata o CPF
            let formattedCpf = this.cpf.replace(/\D/g, "");

            if (formattedCpf.length > 3) {
                formattedCpf = formattedCpf.replace(/(\d{3})(\d)/, "$1.$2");
            }
            if (formattedCpf.length > 7) {
                formattedCpf = formattedCpf.replace(/(\d{3}\.\d{3})(\d)/, "$1.$2");
            }
            if (formattedCpf.length > 11) {
                formattedCpf = formattedCpf.replace(/(\d{3}\.\d{3}\.\d{3})(\d)/, "$1-$2");
            }

            this.cpf = formattedCpf;
        },
        refreshPage() {
            window.location.reload();
        },
        startCountdown() {
            const videoContainer = this.$el.querySelector('.video-container');
            const snapshotBtn = this.$el.querySelector('#snapshotBtn');
            const countdown = this.$el.querySelector('#countdown');
            const results = this.$el.querySelector('#results');

            // Remove a imagem anterior
            results.innerHTML = ''; // Limpa o resultado anterior
            videoContainer.style.display = 'block'; // Exibe o vídeo
            snapshotBtn.style.display = 'none'; // Oculta o botão enquanto a contagem está em andamento

            let seconds = 3;
            countdown.style.display = 'block'; // Exibe o contador
            countdown.innerText = seconds;

            const interval = setInterval(() => {
                seconds--;
                countdown.innerText = seconds;

                if (seconds < 1) {
                    clearInterval(interval);
                    this.takeSnapshot();
                    countdown.style.display = 'none'; // Esconde o contador após a contagem
                    snapshotBtn.style.display = 'inline-block'; // Mostra o botão novamente
                }
            }, 1000);
        },
        takeSnapshot() {
            const context = this.$refs.canvas.getContext('2d');
            context.drawImage(this.$refs.video, 0, 0, this.$refs.canvas.width, this.$refs.canvas.height);
            const data_uri = this.$refs.canvas.toDataURL('image/jpeg');

            // Salvar a imagem no estado 'data' como um objeto File
            fetch(data_uri)
                .then(res => res.blob())
                .then(blob => {
                    this.imagemCapturada = new File([blob], 'foto.jpg', { type: 'image/jpeg' }); // Nome da foto e tipo
                });

            // Exibe a imagem capturada no lugar do vídeo
            const results = this.$el.querySelector('#results');
            results.innerHTML = '<img src="' + data_uri + '" width="320" height="240" style="display:block;margin: 0 auto;"/>';
            this.$el.querySelector('.video-container').style.display = 'none'; // Esconde o vídeo

            // Define fotoTirada como true
            this.fotoTirada = true;

            if (this.timeoutAnterior) {
                clearTimeout(this.timeoutAnterior);
            }

            this.timeoutAnterior = setTimeout(() => {
                this.fotoTirada = false;
                results.innerHTML = ''; // Limpa o resultado anterior
            }, 10000);
        },
        handleImageUpload(event) {
            this.imagem = event.target.files[0]; // Armazenar a imagem selecionada
            if (this.imagem) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.imagemCapturada = e.target.result; // Salva a imagem capturada como a imagem selecionada
                };
                reader.readAsDataURL(this.imagem); // Lê a imagem como uma URL
            }
        },
        updateSuccessModalVisibility(val) {
            this.isSuccessModalVisible = val;
        },
        updateErrorModalVisibility(val) {
            this.isModalVisible = val;
        },
    },
    mounted() {
        const video = this.$refs.video;

        // Solicita permissão para acessar a câmera
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(stream => {
                video.srcObject = stream;
            })
            .catch(err => {
                console.error("Erro ao acessar a câmera: ", err);
            });
    },
    created() {
        this.getLink()
    }
}
</script>

<style scoped>
.container {
    color: #173B56;
    max-width: 1000px;
}

.formulario {
    padding: 2%;
    background-color: #fff;
    border: #173B56 2px solid;
    border-radius: 15px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

input {
    border: #173B56 1px solid;
}

#snapshotBtn {
    background-color: white;
    border: #173B56 1px solid;
    padding: 1px;
}

#snapshotBtn:hover {
    box-shadow: cornflowerblue 0 0 5px;
}

.btn-primary {
    --bs-btn-bg: #8AB156;
    --bs-btn-border-color: #6a8e3b;
    --bs-btn-hover-bg: #6a8e3b;
    --bs-btn-hover-border-color: #8AB156;
    --bs-btn-active-bg: #53702e;
    --bs-btn-active-border-color: #53702e;
}

.video-container {
    position: relative;
}

.countdown {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 120px;
    font-weight: bold;
    color: rgba(255, 255, 255, 0.545);
    display: block;
}

.btn-disabled {
    background-color: rgba(128, 128, 128, 0.452) !important;
    /* Cor cinza quando desabilitado */
    color: #000000c4 !important;
    /* Cor do texto branca para contraste */
    font-weight: bold;
    border: gray;
}
</style>