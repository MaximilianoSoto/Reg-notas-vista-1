<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import axios from 'axios';

const periodos = ref([])


const fetchPeriodos = async () => {
      try {
        const response = await axios.get('https://portalonlinedev.unap.cl/rn-fg/?bandera=getPeriodoProf&rutProfesor=17111875')
        periodos.value = response.data.message[0].periodos
      
      } catch (e) {
        error.value = 'Error al obtener los periodos: ' + e.message
      }
}



const llamadaApi = (periodos) =>{
  alert(periodos)
}


const years = computed(() => {
      return [...new Set(periodos.value.map(periodo => periodo.split(' / ')[0]))];
    });

const semesters = computed(() => {
      return [...new Set(periodos.value.map(periodo => periodo.split(' / ')[1]))];
    });

onMounted(()=>{
    fetchPeriodos()
})


const selectedYear = ref('');
const selectedSemester = ref('1er. semestre');
 
const activities = ref([
  { name: 'Lorem Ipsum 1', from: '01 septiembre 2024', to: '18 septiembre 2024', status: 'Cerrada', ponderacion: 100 },
  { name: 'Lorem Ipsum 2', from: '02 agosto 2024', to: '01 agosto 2024', status: 'Abierta', ponderacion: 80 },
  { name: 'Lorem Ipsum 3', from: '07 julio 2024', to: '10 julio 2024', status: 'Cerrada', ponderacion: 65 },
]);
 
const getStatusClass = computed(() => (status) =>
  status === 'Cerrada' ? 'estado-cerrado' : 'estado-abierto'
);
 
const handleClick = () => alert("ACCION");

watch(selectedYear, (newValue) => {
  console.log('Período seleccionado:', newValue);
});

</script>
 
<template>
  <div class="container-fluid mt-4">
    <div class="row mb-3 align-items-center">
      <div class="col-12 col-md-6">
        <h2>Ingreso Ponderación de Notas</h2>
      </div>
      <div class="col-12 col-md-6 text-md-end">
        <select class="form-select d-inline-block w-auto" 
                v-model="selectedYear"
                @change="llamadaApi(selectedYear)">
          <option value="">Seleccione</option>
          <option 
            v-for="periodo in periodos"
            :key="periodo"
            :value="periodo"
          >
            {{ periodo }}
          </option>
        </select>
      </div>
    </div>
    <div class="table-responsive">
      <table class="table table-hover">
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
          <tr v-for="(item, index) in activities" :key="index" @click="handleClick">
            <td>{{ item.name }}</td>
            <td>{{ item.from }}</td>
            <td>{{ item.to }}</td>
            <td>
              <span :class="['badge', getStatusClass(item.status)]">
                {{ item.status }}
              </span>
            </td>
            <td>{{ item.ponderacion }}%</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
 
<style scoped>
.table-header {
  text-align: left;
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
.table-body {
  font-weight: 600;
}
.badge {
  padding: 5px 15px;
}
</style>