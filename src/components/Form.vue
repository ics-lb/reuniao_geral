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
            gapiReady: false,
            CLIENT_ID: 'YOUR_CLIENT_ID.apps.googleusercontent.com', // Substitua pelo seu Client ID
            API_KEY: 'AIzaSyA5OHQT5QZcEQTu4lrT71ga5-HS3cLJT0M', // Substitua pela sua API Key
            SCOPES: "https://www.googleapis.com/auth/spreadsheets",
            SPREADSHEET_ID: '14rHKdk3mLpkUh7ISJce_bDNKwNOgjBPUxWq9nF5xNG8', // Substitua pelo ID da sua planilha
            RANGE: 'Sheet1!A1', // Ajuste o intervalo conforme necessário
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
        initGAPI() {
            gapi.load("client:auth2", () => {
                gapi.client.init({
                    apiKey: this.API_KEY,
                    clientId: this.CLIENT_ID,
                    scope: this.SCOPES,
                    discoveryDocs: ["https://sheets.googleapis.com/$discovery/rest?version=v4"]
                }).then(() => {
                    this.gapiReady = true;
                });
            });
        },
        async salvarPresenca() {
            try {
                // ... seu código anterior ...

                if (this.gapiReady) {
                    // Prepare os valores a serem enviados para a planilha
                    const values = [
                        [this.nome_completo, this.cpf.replace(/\D/g, ''), this.mes_referencia, this.ano_referencia, this.data_hora]
                    ];

                    const body = {
                        values: values
                    };

                    // Chamada para a API do Google Sheets
                    const response = await gapi.client.sheets.spreadsheets.values.append({
                        spreadsheetId: this.SPREADSHEET_ID,
                        range: this.RANGE,
                        valueInputOption: "RAW",
                        resource: body
                    });

                    if (response.status === 200) {
                        this.isSuccessModalVisible = true; // Define o modal de sucesso como visível
                    } else {
                        this.errorMessage = "Erro ao cadastrar presença na planilha. Por favor, tente novamente.";
                        this.isModalVisible = true;
                    }
                } else {
                    this.errorMessage = "API do Google não está pronta.";
                    this.isModalVisible = true;
                }
            } catch (error) {
                this.errorMessage = error.message || 'Ocorreu um erro desconhecido.';
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
        this.initGAPI();

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
