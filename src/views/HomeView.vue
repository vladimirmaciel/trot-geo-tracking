<template>
  <v-container class="fill-height">
    <v-responsive class="d-flex align-center text-center fill-height">
      <v-form>
        <v-text-field v-model="percurso" label="Novo percurso" required></v-text-field>

        <v-btn v-if="id == 0" variant="flat" color="secondary" block @click="iniciar">
          INICIAR
        </v-btn>
        <v-btn v-else variant="flat" color="secondary" block @click="parar"> PARAR </v-btn>
      </v-form>

      <div v-if="lastPosition">
        <h2>Última posição</h2>
        <p>
          <span>{{ lastPosition }}</span>
        </p>
      </div>

      <v-btn color="primary" variant="flat" :to="{ name: 'PercursoDetalheView' }" class="my-4">
        Ver percurso
      </v-btn>

      <open-layer-map-point-viewer v-if="latLon" :positions-list="latLon" :center="center">
      </open-layer-map-point-viewer>
    </v-responsive>
  </v-container>
</template>

<script>
import { mapState } from "pinia"
import { useLocationStore } from "@/store/location"
import OpenLayerMapPointViewer from "@/components/OpenLayerMapPointViewer.vue"

export default {
  components: {
    OpenLayerMapPointViewer,
  },
  setup() {
    const locationStore = useLocationStore()

    return {
      locationStore,
    }
  },
  data() {
    return {
      id: 0,
      lastPosition: {},
      positions: [],
      percurso: "",
    }
  },
  computed: {
    ...mapState(useLocationStore, ["latLon"]),
    center() {
      return this?.latLon[0]
    },
  },
  methods: {
    geoSuccess(position) {
      if (
        position.coords.latitude == this.lastPosition.latitude &&
        position.coords.longitude == this.lastPosition.longitude
      ) {
        return
      }
      const newPosition = {
        latitude: position.coords.latitude,
        longitude: position.coords.longitude,
        latLongAccuracy: position.coords.accuracy,
        heading: position.coords.heading,
        speed: position.coords.speed,
        altitude: position.coords.altitude,
        altitudeAccuracy: position.coords.altitudeAccuracy,
        date: new Date().getTime(),
      }
      this.locationStore.savePositions(newPosition)
      // this.positions.push(newPosition)
      this.lastPosition = newPosition
    },
    geoError(error) {
      console.log("Vish, deu ruim!", error)
    },
    iniciar() {
      //navigator.geolocation.getCurrentPosition(this.geoSuccess, this.geoError)
      this.id = navigator.geolocation.watchPosition(this.geoSuccess, this.geoError)
    },
    parar() {
      navigator.geolocation.clearWatch(this.id)
      this.id = 0
    },
  },
}
</script>
