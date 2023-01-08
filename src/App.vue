<script setup>
import { computed, onMounted, ref, watch } from 'vue';

const ratesAr = ref([]);
const rateClp = ref(parseFloat(localStorage.getItem('rate-clp')) || 840 );
const selectedArRate = ref(parseInt(localStorage.getItem('selected-rate')) || 1);
const clpAmount = ref(0);

watch(rateClp, (newVal, oldVal) => {
  localStorage.setItem('rate-clp', newVal);
})

watch(selectedArRate, (newVal, oldVal) => {
  localStorage.setItem('selected-rate', newVal);
});

const rateArs = computed( () => {
  if ( ratesAr.value.length == 0 ) return null;

  return parseFloat(ratesAr.value[selectedArRate.value].venta.replace(',', '.'));
})

const exchange = computed( () => {
  if ( rateClp.value === 0 || rateArs.value === 0) {
    return 0; 
  } 

  const rate = rateArs.value / rateClp.value; 

  return (clpAmount.value * rate).toFixed(2);
})

const exchangeUsd = computed( () => {
  if ( rateClp.value === 0 ) {
    return 0; 
  } 

  const rate = rateClp.value; 

  return (clpAmount.value / rate).toFixed(2);
})

onMounted( () => {
  if ( sessionStorage.getItem('cotizaciones')) {
    ratesAr.value = JSON.parse(sessionStorage.getItem('cotizaciones')); 
    return; 
  } 

  fetch('https://mercados.ambito.com/home/general').then(res => res.json())
    .then( res => {
      ratesAr.value = res.filter( rate => rate.nombre.includes('Dólar'));
      sessionStorage.setItem('cotizaciones', JSON.stringify(ratesAr.value));
    }); 
});
</script>

<template>

<h1>Rápida calculadora de cambio</h1>
<div class="input-group">
  <label for="rateAr">Seleccionar Tipo de Dolar</label>
  <select name="rates" id="rateAr" v-model="selectedArRate">
    <option v-for="(rate, index) in ratesAr" :value="index" :key="index">
      {{ rate.nombre }} - ${{  rate.venta  }} 
    </option>
  </select>
</div>

<div class="input-group">
  <label for="">Cotizacion USD a CLP</label>
  <input type="number" v-model="rateClp" min="1" step="any">
</div>

<div class="input-group">
  <label for="">Pesos Chilenos</label>
  <input type="number" min="1" step="any" v-model="clpAmount" class="big">
</div>

<hr>

<h3>Total en ARS: {{ exchange }}</h3>
<h3>Total en USD: {{  exchangeUsd }} </h3>
</template>


<style>
h1 {
  font-size: 22px;
}

.input-group {
  display: block; 
  margin-bottom: 20px;
  text-align: left;
}

.input-group label {
  display: block; 
  font-weight: bold;
}

.input-group input, 
.input-group select {
  display: block; 
  width: 100%;
  padding: 3px 10px;
}

.input-group input.big {
  font-size: 20px;
} 
</style>