<template>
  <q-dialog
    persistent
    :value="modalFila"
    @hide="fecharModal"
    @show="abrirModal"
  >
    <q-card
      class="q-pa-lg modal-container container-rounded-10"
    >
    <q-card-actions
        align="right"
        class="q-mt-md"
      >
        <q-btn
          icon="eva-close"
          color="negative"
          v-close-popup
          flat
        />
      </q-card-actions>
      <q-card-section>
        <div class="text-h6 text-center font-family-main">{{ filaEdicao.id ? 'Editar': 'Criar' }} Fila</div>
      </q-card-section>
      <q-card-section class="container-border container-rounded-10">
        <div class="text-h6 font-family-main q-mb-sm">
          Informações
        </div>
        <q-input
          class="row col"
          rounded
          outlined
          v-model="fila.queue"
          label="Nome da Fila"
        />
        <q-input
          filled
          hide-bottom-space
          :style="`background: ${fila.color} `"
          v-model="fila.color"
          :rules="['anyColor']"
          class="q-my-md"
          :dark="false"
        >
          <template v-slot:preappend>
          </template>
          <template v-slot:append>
            <q-icon
              name="colorize"
              class="cursor-pointer"
            >
              <q-popup-proxy
                transition-show="scale"
                transition-hide="scale"
              >
                <q-color
                  format-model="hex"
                  square
                  default-view="palette"
                  no-header
                  bordered
                  v-model="fila.color"
                />
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
        <q-checkbox
          v-model="fila.isActive"
          label="Ativo"
        />
      </q-card-section>
      <q-card-actions
        align="right"
        class="q-mt-md"
      >
        <q-btn
          label="Cancelar"
          color="negative"
          v-close-popup
          class="q-mr-md btn-rounded-50"
        />
        <q-btn
          label="Salvar"
          class="generate-button btn-rounded-50"
          @click="handleFila"
          icon="eva-save-outline"
        />
      </q-card-actions>
    </q-card>
  </q-dialog>

</template>

<script>
import { CriarFila, AlterarFila } from 'src/service/filas'
export default {
  name: 'ModalFila',
  props: {
    modalFila: {
      type: Boolean,
      default: false
    },
    filaEdicao: {
      type: Object,
      default: () => {
        return { id: null }
      }
    }
  },
  data () {
    return {
      fila: {
        id: null,
        queue: null,
        from_ia: false,
        color: '#ffffff',
        isActive: true
      }
    }
  },
  methods: {
    resetarFila () {
      this.fila = {
        id: null,
        queue: null,
        from_ia: false,
        color: '#ffffff',
        isActive: true
      }
    },
    fecharModal () {
      this.resetarFila()
      this.$emit('update:filaEdicao', { id: null })
      this.$emit('update:modalFila', false)
    },
    abrirModal () {
      if (this.filaEdicao.id) {
        this.fila = { ...this.filaEdicao }
      } else {
        this.resetarFila()
      }
    },
    async handleFila () {
      try {
        this.loading = true
        if (this.fila.id) {
          const { data } = await AlterarFila(this.fila)
          this.$emit('modal-fila:editada', data)
          this.$q.notify({
            type: 'info',
            progress: true,
            position: 'top',
            textColor: 'black',
            message: 'Etapa editada!',
            actions: [{
              icon: 'close',
              round: true,
              color: 'white'
            }]
          })
        } else {
          const { data } = await CriarFila(this.fila)
          this.$emit('modal-fila:criada', data)
          this.$q.notify({
            type: 'positive',
            progress: true,
            position: 'top',
            message: 'Fila criada!',
            actions: [{
              icon: 'close',
              round: true,
              color: 'white'
            }]
          })
        }
        this.loading = false
        this.fecharModal()
      } catch (error) {
        console.error(error)
        this.$notificarErro('Ocorreu um erro!', error)
      }
    }
  }

}
</script>

<style lang="scss" scoped>
</style>
