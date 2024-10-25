<template>
    <div v-if="showModal" class="modal fade show" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false"
        tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true" style="display: block">
        <div class="modal-dialog justify-content-center">
            <div class="modal-content">
                <div class="modal-header">
                    <div class="text-center w-100">
                        <h1 class="modal-title fs-5" id="staticBackdropLabel">PRESENÇA CONFIRMADA</h1>
                    </div>
                </div>
                <div class="modal-body text-center">
                    A sua presença na reunião geral deste mês foi confirmada. Muito obrigado!
                </div>
                <div class="modal-footer d-flex justify-content-center">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal"
                        @click="refreshPage()">Voltar para a lista</button>
                    <a :href="link" target="_blank">
                        <button type="button" class="btn btn-primary">Voltar para a reunião</button>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        showModal: {
            type: Boolean,
            required: true
        },
        link: {
            type: String,
            required: true
        }
    },
    watch: {
        showModal(newVal) {
            if (newVal) {
                setTimeout(() => {
                    this.refreshPage();
                }, 10000);
            }
        }
    },
    methods: {
        handleClose() {
            this.$emit('update:showModal', false);
        },
        refreshPage() {
            window.location.reload();
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
    justify-content: center;
}

/* Ajusta o conteúdo do modal */
.modal-content {
    border-radius: 8px;
}

/* Ajusta o cabeçalho do modal */
.modal-header {
    justify-content: center;
    /* Ajuste a cor conforme necessário */
    color: #000000;
}

/* Ajusta o corpo do modal */
.modal-body {
    color: #000000;
    /* Ajuste a cor conforme necessário */
    text-align: center;
}

/* Ajusta o rodapé do modal */
.modal-footer {
    justify-content: center;
}
</style>
