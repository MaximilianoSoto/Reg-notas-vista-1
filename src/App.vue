<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

// Variables reactivas
const periodos = ref([]);
const asigProf = ref([]);
const asigProfLimpio = ref([]);
const spinner = ref(false);
const isProf = ref(false);
const seleccionPeriodo = ref('');

// Meses en texto
const meses = ["enero", "febrero", "marzo", "abril", "mayo", "junio", 
               "julio", "agosto", "septiembre", "octubre", "noviembre", "diciembre"];

// Función para formatear fecha
const formatearFecha = (fecha) => {
  const [dia, mes, año] = fecha.split("/");
  return `${dia} ${meses[parseInt(mes, 10) - 1]} de ${año}`;
};

// Obtener los periodos del profesor
const fetchPeriodos = async () => {
  try {
    const { data } = await axios.get('https://portalonlinedev.unap.cl/rn-fg/?bandera=getPeriodoProf&rutProfesor=17111875');
    periodos.value = data.message[0].periodos;
  } catch (e) {
    console.error('Error al obtener los periodos:', e.message);
  }
};

// Obtener asignaciones según el periodo seleccionado

const fetchAsigProf = async (año, proceso) => {
  spinner.value = true;
  try {
    const { data } = await axios.get(`https://portalonlinedev.unap.cl/rn-fg/?bandera=getAsigProf&anho=${año}&semestre=${proceso}&rutProfesor=17111875`);
    asigProf.value = data.message;
    limpiarAsigProfe(asigProf.value);
    isProf.value = true;
  } catch (e) {
    console.error('Hubo un error en la llamada:', e.message);
  } finally {
    spinner.value = false;
  }
};

// Llamada API basada en el periodo seleccionado
const llamadaApi = (periodo) => {
  const [año, proceso] = periodo.split(" / ").map(str => str.trim());
  fetchAsigProf(año, proceso);
};

// Formatear la lista de asignaturas
const limpiarAsigProfe = (lista) => {
  asigProfLimpio.value = lista.map(item => ({
    ...item,
    NOMBASIG: item.NOMBASIG.toLowerCase(),
    FINI: formatearFecha(item.FINI),
    FFIN: formatearFecha(item.FFIN)
  }));
};


onMounted(fetchPeriodos);

const getStatusClass = computed(() => status =>
  status === 'Cerrada' ? 'estado-cerrado' : 'estado-abierto'
);

const handleClick = (item) => {
  console.log("Accion", item)
}

</script>

<template>
  <div class="container-fluid mt-4">
    <div class="row mb-3 align-items-center">
      <div class="col-12 col-md-5">
        <h2>Ingreso Ponderación de Notas</h2>
      </div>
      <div class="col-12 col-md-7 text-md-end">

        <!-- Selector de periodo -->
        <select class="form-select d-inline-block w-auto" 
                v-model="seleccionPeriodo"
                @change="llamadaApi(seleccionPeriodo)">
          <option value="">Seleccione Periodo</option>
          <option v-for="periodo in periodos" :key="periodo" :value="periodo">{{ periodo }}</option>
        </select>
      </div>
    </div>

    <!-- Tabla con datos -->
    <div class="table-responsive" v-if="!spinner">
      <table v-if="isProf" class="table table-hover">
        <thead>
          <tr class="table-header">
            <th>Actividad Curricular</th>
            <th>Desde</th>
            <th>Hasta</th>
            <th>Estado</th>
            <th>Ponderación</th>
          </tr>
        </thead>
        <tbody class="table-body">
          <tr v-for="(item, index) in asigProfLimpio" :key="index" @click="handleClick(item)" >
            <td>{{ item.NOMBASIG }}</td>
            <td>{{ item.FINI }}</td>
            <td>{{ item.FFIN }}</td>
            <td>
              <span :class="['badge', getStatusClass(item.status)]">
                Abierta
              </span>
            </td>
            <td>{{ item.PORC }}%</td>
          </tr>
        </tbody>
      </table>
      
      <!-- Mensaje de selección de periodo -->
      <div v-else class="card text-center shadow-xs h-100 my-3" style="background-color: #f8f9fa; border-radius: 10px;">
        <div class="card-body">
          <h2 class="card-title mb-4 text-primary">Seleccione el periodo</h2>
          <p class="card-text text-muted">Por favor, elija un periodo para continuar con el proceso de asignación.</p>
        </div>
      </div>
    </div>

    <!-- Spinner de carga -->
    <div v-if="spinner" class="spinner">
      <div class="cube1"></div>
      <div class="cube2"></div>
    </div>
  </div>

</template>

<style scoped>
.table-header {
  background-color: #808A9D;
  color: white;
}
.estado-cerrado {
  background-color: rgba(183, 47, 99, 0.4);
  color: #B72F63;
}
.estado-abierto {
  background-color: rgba(15, 174, 192, 0.4);
  color: #0FAEC0;
}
.badge {
  padding: 5px 15px;
}
.spinner {
  margin: 100px auto;
  width: 40px;
  height: 40px;
}
.cube1, .cube2 {
  background-color: #0FAEC0;
  width: 15px;
  height: 15px;
  position: absolute;
  top: 0;
  left: 0;
  animation: sk-cubemove 1.8s infinite ease-in-out;
}
.cube2 {
  animation-delay: -0.9s;
}
@keyframes sk-cubemove {
  25% { transform: translateX(42px) rotate(-90deg) scale(0.5); }
  50% { transform: translateX(42px) translateY(42px) rotate(-180deg); }
  75% { transform: translateX(0) translateY(42px) rotate(-270deg) scale(0.5); }
  100% { transform: rotate(-360deg); }
}
</style>
