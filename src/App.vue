<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

const paises = ref([]);
const busqueda = ref('');
const cargando = ref(true);
const error = ref(null);
const paisSeleccionado = ref(null);

const obtenerPaises = async () => {
  try {
    cargando.value = true;
    const response = await fetch('https://restcountries.com/v3.1/all?fields=name,flags,capital,region,subregion,population,languages,currencies');
    if (!response.ok) throw new Error('Error al obtener datos');
    paises.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    cargando.value = false;
  }
};

const abrirDetalle = (pais) => {
  paisSeleccionado.value = pais;
};

const cerrarDetalle = () => {
  paisSeleccionado.value = null;
};

const manejarTeclaEsc = (e) => {
  if (e.key === 'Escape' && paisSeleccionado.value) {
    cerrarDetalle();
  }
};

onMounted(() => {
  obtenerPaises();
  window.addEventListener('keydown', manejarTeclaEsc);
});

onUnmounted(() => {
  window.removeEventListener('keydown', manejarTeclaEsc);
});

const paisesFiltrados = computed(() => {
  return paises.value.filter((pais) => 
    pais.name.common.toLowerCase().includes(busqueda.value.toLowerCase()) ||
    pais.region.toLowerCase().includes(busqueda.value.toLowerCase())
  );
});
</script>

<template>
  <div class="app-container">
    <h1>Explorador de Países</h1>

    <div class="search-box">
      <input v-model="busqueda" type="text" placeholder="Buscar por nombre o región..." />
    </div>

    <div v-if="cargando" class="loading">⏳ Cargando países...</div>

    <div v-else>
      <div class="countries-grid">
        <div 
          v-for="pais in paisesFiltrados" 
          :key="pais.name.common" 
          class="country-card"
          @click="abrirDetalle(pais)" 
        >
          <img :src="pais.flags.png" :alt="pais.name.common" />
          <div class="country-info">
            <h3>{{ pais.name.common }}</h3>
            <p><strong>Capital:</strong> {{ pais.capital?.[0] || 'N/A' }}</p>
            <p><strong>Población:</strong> {{ pais.population.toLocaleString() }}</p>
            <p><strong>Región:</strong> {{ pais.region }}</p>
          </div>
        </div>
      </div>

      <div v-if="paisSeleccionado" class="modal-overlay" @click.self="cerrarDetalle">
        <div class="modal-content">
          <button class="close-btn" @click="cerrarDetalle">X</button>
          
          <img :src="paisSeleccionado.flags.svg" :alt="paisSeleccionado.name.common" class="modal-flag" />
          
          <div class="modal-body">
            <h2>{{ paisSeleccionado.name.common }}</h2>
            <hr />
            <div class="details-grid">
              <p><strong>Capital:</strong> {{ paisSeleccionado.capital?.[0] || 'N/A' }}</p>
              <p><strong>Población:</strong> {{ paisSeleccionado.population.toLocaleString() }}</p>
              <p><strong>Región:</strong> {{ paisSeleccionado.region }}</p>
              <p><strong>Subregión:</strong> {{ paisSeleccionado.subregion || 'N/A' }}</p>
              <p><strong>Lenguajes:</strong> 
                {{ Object.values(paisSeleccionado.languages || {}).join(', ') }}
              </p>
              <p><strong>Monedas:</strong> 
                {{ Object.values(paisSeleccionado.currencies || {}).map(c => `${c.name} (${c.symbol})`).join(', ') }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container { max-width: 1100px; margin: 0 auto; padding: 20px; font-family: 'Segoe UI', sans-serif; text-align: center; }
.search-box { margin-bottom: 2rem; }
.search-box input { width: 100%; max-width: 400px; padding: 12px; border-radius: 8px; border: 1px solid #ddd; }

.countries-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 20px; }
.country-card { border-radius: 10px; overflow: hidden; background: #fff; box-shadow: 0 4px 6px rgba(0,0,0,0.1); cursor: pointer; transition: transform 0.2s; }
.country-card:hover { transform: translateY(-5px); }
.country-card img { width: 100%; height: 140px; object-fit: cover; }
.country-info { padding: 15px; text-align: left; }

/* Estilos del Modal Actualizados */
.modal-overlay {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0, 0, 0, 0.7); display: flex; align-items: center; justify-content: center; z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 15px;
  width: 90%;
  max-width: 550px; /* Ancho movible hasta este máximo */
  height: 650px;    /* ALTURA FIJA PARA LA TARJETA COMPLETA */
  position: relative;
  display: flex;
  flex-direction: column;
  overflow: hidden; /* Mantiene el contenido dentro de los bordes redondeados */
  animation: fadeIn 0.3s ease;
}

.modal-flag {
  width: 100%;
  height: 280px;    /* ALTURA FIJA PARA LA BANDERA */
  object-fit: cover; /* Asegura que la bandera llene el espacio sin deformarse */
  flex-shrink: 0;   /* Evita que la bandera se encoja si hay mucho texto */
}

.modal-body {
  padding: 25px;
  text-align: left;
  overflow-y: auto; /* PERMITE SCROLL SI LA INFO ES LARGA */
  flex-grow: 1;
}

.close-btn {
  position: absolute; top: 15px; right: 15px;
  border: none; background: rgba(255, 255, 255, 0.8);
  border-radius: 50%; width: 35px; height: 35px;
  cursor: pointer; font-weight: bold; z-index: 10;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.details-grid p { margin: 12px 0; line-height: 1.6; border-bottom: 1px solid #f0f0f0; padding-bottom: 5px; }

@keyframes fadeIn { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
</style>