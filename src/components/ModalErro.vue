<template>
    <div>
        <div v-if="showModal" class="modal fade show d-block" tabindex="-1" aria-labelledby="modalErroLabel"
            aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-3" id="modalErroLabel">{{ title }}</h1>
                    </div>
                    <div class="modal-body">
                        {{ errorMessage }}
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeModal()">Fechar</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Button trigger modal -->
        <!-- <div class="container-modal-erro">
        <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#modalErro">
            MOSTRAR MODAL DE ERRO
        </button> -->

        <!-- Modal -->
        <!--  <div class="modal fade" id="modalErro" tabindex="-1" aria-labelledby="modalErroLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-5" id="modalErroLabel">ATENÇÃO</h1>
                    </div>
                    <div class="modal-body">
                        Não foi possível realizar a operação solicitada. Por gentileza, tente novamente mais tarde!
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Fechar</button>
                    </div>
                </div>
            </div>
        </div>
    </div> -->

        <!-- <div class="container-modal-notificacao">
        <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#modalNotificacao">
            MOSTRAR MODAL DE NOTIFICAÇÃO
        </button>

        <div class="modal fade" id="modalNotificacao" tabindex="-1" aria-labelledby="modalNotificacaoLabel"
            aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header modal-header2">
                        <h1 class="modal-title fs-5" id="modalNotificacaoLabel">IMPORTANTE</h1>
                    </div>
                    <div class="modal-body">
                        Você possui mensagens na caixa de notificações. Verifique!
                    </div>
                    <div class="modal-footer modal-footer2">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Fechar</button>
                        <router-link to="notificacoes">
                            <button type="button" class="btn btn-primary btn-primary2"
                                data-bs-target="#exampleModalToggle2" data-bs-toggle="modal">Ver notificações</button>
                        </router-link>
                    </div>
                </div>
            </div>
        </div>
    </div> -->
    </div>
</template>

<script>
export default {
    props: {
        showModal: {
            type: Boolean,
            default: false
        },
        errorMessage: {
            type: String,
            default: ''
        },
        title: {
            type: String,
            default: ''
        }
    },
    methods: {
        closeModal() {
            this.$emit('update:showModal', false);
        },
    }
}
</script>

<style scoped>
/* Adiciona um fundo escuro e transparente ao modal */
.modal.fade.show {
    background-color: rgba(0, 0, 0, 0.5);
    /* Fundo escuro com 50% de opacidade */
}

/* Alinha o conteúdo do modal no centro */
.modal-dialog {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
}

/* Ajusta o conteúdo do modal */
.modal-content {
    border-radius: 8px;
}

/* Ajusta o cabeçalho do modal */
.modal-header {
    justify-content: center;
    /* Ajuste a cor conforme necessário */
    color: #173B56;
}

/* Ajusta o corpo do modal */
.modal-body {
    color: #173B56;
    /* Ajuste a cor conforme necessário */
    text-align: center;
}

/* Ajusta o rodapé do modal */
.modal-footer {
    justify-content: center;
}

/* Estilo do botão de fechar */
.btn-secondary {
    color: white;
    border: none;
    border-radius: 5px;
}

/* Estilo do botão de fechar ao passar o mouse */
.btn-secondary:hover {
    background-color: #0d2f49;
    color: white;
}
</style>