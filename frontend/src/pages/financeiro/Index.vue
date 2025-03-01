<template>
  <div>
    <q-table flat
      bordered
      square
      hide-bottom
      class="contact-table container-rounded-10 my-sticky-dynamic q-ma-lg"
      :data="financeiro"
      :columns="columns"
      :loading="loading"
      row-key="id"
      :pagination.sync="pagination"
      :rows-per-page-options="[0]">
      <template v-slot:top-left>
        <div>
          <h2  :class="$q.dark.isActive ? ('text-white') : ''">
            <q-icon name="mdi-cash-multiple q-pr-sm" />
            Financeiro
          </h2>
        </div>

      </template>
      <template v-slot:body-cell-value="props">
        <q-td key="value" :props="props">
          R$ {{ props.row.value.toFixed(2).replace('.', ',') }}
        </q-td>
      </template>
      <template v-slot:body-cell-dueDate="props">
        <q-td key="dueDate" :props="props">
          {{ formatDate(props.row.dueDate) }}
        </q-td>
      </template>
      <template v-slot:body-cell-status="props">
        <q-td
          key="status"
          :props="props"
          :style="getStatusStyle(props.row.status, props.row.dueDate)"
        >
          {{ getStatusText(props.row.status, props.row.dueDate) }}
        </q-td>
      </template>
      <template v-slot:body-cell-acoes="props">
        <q-td class="text-center">
          <q-btn
            v-if="props.row.status !== 'paid'"
            color="primary"
            label="PAGAR"
            @click="GerarQRCODE(props.row)"
          />
        </q-td>
      </template>
    </q-table>

    <q-dialog v-model="showQRCodeModal" persistent>
    <q-card class="container-rounded-10">
      <q-card-section>
        <div class="text-h6 font-family-main text-center">
          QR Code de Pagamento
          <q-btn round
          flat
            class="q-ml-md float-right"
            color="negative"
            icon="eva-close"
            v-close-popup />
        </div>
      </q-card-section>
      <q-card-section class="text-center q-ma-lg container-rounded-10 container-border"
        :style="$q.dark.isActive ? 'background: white !important' : ''">
          <div>Faça leitura do QR Code no aplicativo de seu banco</div>
          <qrcode-vue :value="qrCodeValue" :size="200"></qrcode-vue>

      <q-card-section>
          <div>
            <q-input
              v-model="pixString"
              readonly
              filled
              class="q-ma-md"
              style="overflow: hidden; text-overflow: ellipsis; white-space: nowrap;"
            />
          </div>
          <q-btn
            class="generate-button btn-rounded-50"
            label="Copiar código QR"
            @click="copiarPixString"
            icon="mdi-content-copy" />
        <div class="text-center q-mb-lg" style="font-size: 14px">Para finalizar, basta realizar o pagamento escaneando ou colando o código Pix acima :) </div>
        <div class="row col-12 justify-center">
        </div>
      </q-card-section>

      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { ListarInvoices, CriarQRCODE } from 'src/service/financeiro'
import { format, parseISO, isBefore, addDays } from 'date-fns'
import QRCodeVue from 'qrcode.vue'

export default {
  name: 'Financeiro',
  components: {
    'qrcode-vue': QRCodeVue
  },
  data () {
    return {
      financeiro: [],
      pagination: {
        rowsPerPage: 40,
        rowsNumber: 0,
        lastIndex: 0
      },
      loading: false,
      showQRCodeModal: false,
      qrCodeValue: '',
      pixString: '',
      columns: [
        { name: 'id', label: '#', field: 'id', align: 'left' },
        { name: 'detail', label: 'Plano', field: 'detail', align: 'left' },
        { name: 'value', label: 'Valor', field: 'value', align: 'center' },
        { name: 'dueDate', label: 'Data Venc.', field: 'dueDate', align: 'center' },
        { name: 'status', label: 'Status', field: 'status', align: 'center' },
        { name: 'acoes', label: 'Ações', field: 'acoes', align: 'center' }
      ]
    }
  },
  methods: {
    formatDate(date) {
      try {
        return format(parseISO(date), 'dd/MM/yyyy')
      } catch (error) {
        console.error('Invalid date format:', error)
        return date // Fallback to original date if parsing/formatting fails
      }
    },
    async listarFinanceiro () {
      const { data } = await ListarInvoices()
      this.financeiro = data
    },
    getStatusStyle(status, dueDate) {
      const currentDate = addDays(new Date(), -1)
      const parsedDueDate = parseISO(dueDate)
      if (status === 'paid') {
        return { backgroundColor: 'green', color: 'black', fontWeight: 'bold' }
      } else if (status === 'open') {
        if (isBefore(currentDate, parsedDueDate)) {
          return { backgroundColor: 'yellow', color: 'black', fontWeight: 'bold' }
        } else {
          return { backgroundColor: 'red', color: 'black', fontWeight: 'bold' }
        }
      }
      return {}
    },
    getStatusText(status, dueDate) {
      const currentDate = addDays(new Date(), -1)
      const parsedDueDate = parseISO(dueDate)
      if (status === 'paid') {
        return 'PAGO'
      } else if (status === 'open') {
        if (isBefore(currentDate, parsedDueDate)) {
          return 'EM ABERTO'
        } else {
          return 'VENCIDO'
        }
      }
      return status
    },
    async GerarQRCODE(row) {
      try {
        const response = await CriarQRCODE({
          firstName: '',
          lastName: '',
          address2: '',
          city: '',
          state: '',
          zipcode: '',
          country: '',
          useAddressForPaymentDetails: false,
          nameOnCard: '',
          cardNumber: '',
          cvv: '',
          price: row.value,
          invoiceId: row.id
        })
        this.qrCodeValue = response.data.qrcode.qrcode
        this.pixString = response.data.qrcode.qrcode
        this.showQRCodeModal = true
      } catch (error) {
        console.error('Erro ao gerar QRCode:', error)
        this.$q.notify({
          type: 'negative',
          message: 'Erro ao gerar QRCode. Tente novamente mais tarde.'
        })
      }
    },
    copiarPixString() {
      navigator.clipboard.writeText(this.pixString).then(() => {
        this.$q.notify({
          type: 'positive',
          message: 'Código Pix copiado com sucesso!'
        })
      }).catch((err) => {
        console.error('Erro ao copiar código Pix:', err)
        this.$q.notify({
          type: 'negative',
          message: 'Erro ao copiar código Pix.'
        })
      })
    }
  },
  mounted () {
    this.listarFinanceiro()
  }
}
</script>

<style lang="scss" scoped>
</style>
