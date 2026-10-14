<script setup>
import { computed, onMounted, ref } from 'vue'
import ServiceCard from '../components/ServiceCard.vue'
import { obtenerServicios } from '../services/api.js'

const servicios = ref([])
const busqueda = ref('')
const cargando = ref(true)
const error = ref('')
const servicioSeleccionado = ref(null)      // ← NUEVO

const serviciosFiltrados = computed(() => {
  const texto = busqueda.value.toLowerCase()
  return servicios.value.filter(servicio =>
    servicio.nombre.toLowerCase().includes(texto)
  )
})

onMounted(async () => {
  try {
    servicios.value = await obtenerServicios()
  } catch (e) {
    error.value = e.message
  } finally {
    cargando.value = false
  }
})

function seleccionarServicio(servicio) {    // ← NUEVO
  servicioSeleccionado.value = servicio
}
</script>

<template>
  <section>
    <header class="section-header">
      <div>
        <h2>Servicios disponibles</h2>
        <p>Consulta los servicios disponibles.</p>
      </div>
      <label>
        Buscar servicio
        <input v-model="busqueda" type="search" placeholder="Ej. desarrollo web">
      </label>
    </header>

    <p v-if="cargando" aria-live="polite">
      Cargando información...
    </p>
    <p v-else-if="error" role="alert">
      {{ error }}
    </p>
    <p v-else-if="serviciosFiltrados.length === 0">
      No existen servicios que coincidan con la búsqueda.
    </p>

    <div v-else class="services-grid">
      <ServiceCard
        v-for="servicio in serviciosFiltrados"
        :key="servicio.id"
        :servicio="servicio"
        @seleccionar="seleccionarServicio"
      />
    </div>

    <aside v-if="servicioSeleccionado" class="detalle">
      <h3>Detalle del servicio</h3>
      <p><strong>Nombre:</strong> {{ servicioSeleccionado.nombre }}</p>
      <p><strong>Descripción:</strong> {{ servicioSeleccionado.descripcion }}</p>
      <p><strong>Precio:</strong> ${{ servicioSeleccionado.precio }}</p>
      <button @click="servicioSeleccionado = null">Cerrar</button>
    </aside>
  </section>
</template>

<style scoped>
.detalle {
  margin-top: 2rem;
  padding: 1.5rem;
  border: 1px solid #ddd;
  border-radius: 0.75rem;
  background: #f9f9f9;
}
.detalle h3 {
  margin-top: 0;
}
</style>