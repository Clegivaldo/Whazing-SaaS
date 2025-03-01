<template>
  <q-dialog
    persistent
    :value="modalMensagemRapida"
    @hide="fecharModal"
    @show="abrirModal"
  >
    <q-card class="q-pa-lg modal-container container-rounded-10">
      <q-card-actions align="right" class="q-mt-md">
        <q-btn flat color="negative" v-close-popup icon="eva-close" />
      </q-card-actions>
      <q-card-section>
        <div class="text-h6 text-center font-family-main">
          {{ mensagemRapida.id ? 'Editar' : 'Criar' }} Mensagem Rápida {{ mensagemRapida.id ? `(ID: ${mensagemRapida.id})` : '' }}
        </div>
      </q-card-section>
      <q-card-section class="q-pa-none">
        <div class="row q-my-md q-pa-lg container-border container-rounded-10">
          <div class="full-width text-h6 font-family-main q-mb-sm">Chave</div>
          <div class="full-width">
            <q-input
              rounded
              outlined
              v-model="mensagemRapida.key"
              label="Chave"
            />
            <p class="text-center" style="font-size: 10px;">
              A chave é o atalho para pesquisa da mensagem pelos usuários.
            </p>
          </div>
        </div>
        <div class="row items-center container-border container-rounded-10 q-pa-lg">
          <div class="text-h6 full-width font-family-main q-mb-sm">Mensagem</div>
          <div class="col-xs-3 col-sm-2 col-md-1">
            <q-btn round flat class="q-ml-sm">
              <q-icon size="2em" name="mdi-emoticon-happy-outline" />
              <q-tooltip>Emoji</q-tooltip>
              <q-menu anchor="top right" self="bottom middle" :offset="[5, 40]">
                <VEmojiPicker
                  style="width: 40vw"
                  :showSearch="false"
                  :emojisByRow="20"
                  labelSearch="Localizar..."
                  lang="pt-BR"
                  @select="onInsertSelectEmoji"
                />
              </q-menu>
            </q-btn>
          </div>
          <div class="col-xs-8 col-sm-10 col-md-11 q-pl-sm">
            <label class="text-caption">Mensagem:</label>
            <textarea
              ref="inputEnvioMensagem"
              style="min-height: 15vh; max-height: 15vh;"
              class="q-pa-sm bg-white full-width"
              placeholder="Digite a mensagem"
              autogrow
              dense
              outlined
              @input="(v) => mensagemRapida.message = v.target.value"
              :value="mensagemRapida.message"
            />
          </div>
          <div class="col-xs-8 col-sm-10 col-md-11 q-pl-sm">
            <q-file dense outlined v-model="arquivoCarregado" label="Escolha o arquivo" filled />
          </div>
        </div>
      </q-card-section>
      <q-card-actions align="right" class="q-mt-md">
        <q-btn label="Cancelar" color="negative" v-close-popup class="q-mr-md btn-rounded-50" />
        <q-btn
          label="Salvar"
          class="generate-button btn-rounded-50"
          @click="handleMensagemRapida"
          icon="eva-save-outline"
        />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
import { VEmojiPicker } from 'v-emoji-picker'

import { CriarMensagemRapida, AlterarMensagemRapida } from 'src/service/mensagensRapidas'
export default {
  name: 'ModalMensagemRapida',
  components: { VEmojiPicker },
  props: {
    modalMensagemRapida: {
      type: Boolean,
      default: false
    },
    mensagemRapidaEmEdicao: {
      type: Object,
      default: () => {
        return { id: null, key: '', message: '', medias: '' }
      }
    }
  },
  data () {
    return {
      mensagemRapida: {
        key: null,
        message: '',
        medias: null
      },
      arquivoCarregado: null,
      loading: false
    }
  },
  methods: {
    limparArquivo() {
      this.arquivoCarregado = null
    },
    onInsertSelectEmoji (emoji) {
      const self = this
      var tArea = this.$refs.inputEnvioMensagem
      var startPos = tArea.selectionStart,
        endPos = tArea.selectionEnd,
        cursorPos = startPos,
        tmpStr = tArea.value
      if (!emoji.data) {
        return
      }
      self.txtContent = this.mensagemRapida.message
      self.txtContent = tmpStr.substring(0, startPos) + emoji.data + tmpStr.substring(endPos, tmpStr.length)
      this.mensagemRapida.message = self.txtContent
      setTimeout(() => {
        tArea.selectionStart = tArea.selectionEnd = cursorPos + emoji.data.length
      }, 10)
    },
    fecharModal () {
      this.$emit('update:mensagemRapidaEmEdicao', { id: null })
      this.$emit('update:modalMensagemRapida', false)
    },
    abrirModal () {
      if (this.mensagemRapidaEmEdicao.id) {
        this.mensagemRapida = { ...this.mensagemRapidaEmEdicao }
        this.arquivoCarregado = this.mensagemRapidaEmEdicao.media
      } else {
        this.mensagemRapida = {
          key: null,
          message: '',
          medias: null
        }
        this.arquivoCarregado = null
      }
    },
    async handleMensagemRapida () {
      if (!this.mensagemRapida.key) {
        this.$q.notify({
          type: 'negative',
          message: 'A chave não pode estar em branco.'
        })
        return
      }
      if (this.mensagemRapida.message && this.arquivoCarregado) {
        this.$q.notify({
          type: 'negative',
          message: 'Você não pode preencher a mensagem e escolher um arquivo ao mesmo tempo. Por favor, escolha apenas um.'
        })
        return
      }
      if (!this.mensagemRapida.message && !this.arquivoCarregado) {
        this.$q.notify({
          type: 'negative',
          message: 'Você deve preencher a mensagem ou escolher um arquivo.'
        })
        return
      }

      this.loading = true
      try {
        let data
        const formData = new FormData()
        formData.append('key', this.mensagemRapida.key)
        formData.append('message', this.mensagemRapida.message)
        if (this.arquivoCarregado) {
          formData.append('medias', this.arquivoCarregado)
        } else {
          formData.append('medias', null)
        }

        if (this.mensagemRapida.id) {
          data = await AlterarMensagemRapida(this.mensagemRapida.id, formData)
          this.$emit('mensagemRapida:editada', { ...this.mensagemRapida, ...data.data })
        } else {
          data = await CriarMensagemRapida(formData)
          this.$emit('mensagemRapida:criada', { ...this.mensagemRapida, ...data.data })
        }

        this.$q.notify({
          type: 'positive',
          message: this.mensagemRapida.id ? 'Mensagem rápida atualizada com sucesso!' : 'Mensagem rápida criada com sucesso!'
        })
        this.fecharModal()
      } catch (error) {
        console.error(error)
        this.$q.notify({
          type: 'negative',
          message: 'Ocorreu um erro ao salvar a mensagem rápida.'
        })
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
