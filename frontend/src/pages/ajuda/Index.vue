<template>
  <div>
    <div class="bg-white mass-container container-rounded-10">
      <div class="row col full-width q-pa-lg">
      <q-card
        flat
        bordered
        class="full-width"
      >
        <q-card-section class="text-h6 text-bold">
          <h2 :class="$q.dark.isActive ? ('text-white') : ''">
            <q-icon name="mdi-help q-pr-sm" />
            Ajuda
          </h2>
        </q-card-section>
      </q-card>
    </div>

    <div class="container-border q-ma-lg q-pb-md container-rounded-10">
      <q-card-section>
        <h2 :class="$q.dark.isActive ? ('text-white') : ''">
        <q-icon name="eva-list-outline q-pr-sm" />
        Listagem
      </h2>
      </q-card-section>

    <div class="q-pa-md row q-gutter-md">
      <div v-for="ajuda in ajudas" :key="ajuda.id">
        <q-card @click="abrirModal(ajuda)">
          <q-card-section>
            <img :src="`https://img.youtube.com/vi/${ajuda.video}/hqdefault.jpg`" width="100%" height="100%" :alt="ajuda.title">
            <div class="text-h6">{{ ajuda.title }}</div>
            <div class="text-subtitle2">{{ ajuda.description }}</div>
          </q-card-section>
        </q-card>
      </div>
    </div>
    <q-dialog v-model="modalVisivel" persistent>
      <q-card style="width: 580px; height: 335px; position: relative;">
        <q-btn icon="close" @click="modalVisivel = false" style="position: absolute; top: 0; right: 0; z-index: 10;"></q-btn>
        <q-card-section>
          <iframe width="100%" height="290px" :src="modalVideoUrl" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { ListarHelpList } from 'src/service/help'

export default {
  name: 'Ajuda',
  data () {
    return {
      ajudas: [],
      modalVisivel: false,
      modalVideoUrl: ''
    }
  },
  methods: {
    async listarAjuda () {
      try {
        const { data } = await ListarHelpList()
        this.ajudas = data
        console.log('Dados das ajudas:', data)
      } catch (error) {
        console.error('Erro ao carregar ajudas:', error)
      }
    },
    abrirModal (ajuda) {
      this.modalVideoUrl = `https://www.youtube.com/embed/${ajuda.video}`
      this.modalVisivel = true
    }
  },
  mounted () {
    this.listarAjuda()
  }
}
</script>
