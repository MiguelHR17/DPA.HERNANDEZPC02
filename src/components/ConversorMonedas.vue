<template>
  <q-page class="q-pa-md column items-center">
    <q-card class="q-pa-md" style="max-width: 400px; width: 100%">
      <q-card-section>
        <div class="text-h6">Conversor de Monedas</div>
      </q-card-section>

      <q-card-section class="q-gutter-md">
        <q-input v-model="amount" type="number" label="Monto a convertir" />

        <q-select
          v-model="from"
          :options="currencyOptions"
          label="Moneda origen"
          emit-value
          map-options
        />

        <q-select
          v-model="to"
          :options="currencyOptions"
          label="Moneda destino"
          emit-value
          map-options
        />

        <div class="row justify-between">
          <q-btn label="Convertir" color="primary" @click="convertir" />
          <q-btn flat label="↔" @click="intercambiar" />
        </div>

        <div v-if="resultado" class="q-mt-md text-subtitle1 text-primary">
          {{ resultado }}
        </div>

        <q-banner v-if="error" dense class="bg-red text-white q-mt-sm">
          {{ error }}
        </q-banner>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const amount = ref(null)
const from = ref(null)
const to = ref(null)
const resultado = ref('')
const error = ref('')

const currencyOptions = ref([])

onMounted(async () => {
  try {
    const res = await axios.get('https://api.frankfurter.app/currencies')
    const monedas = Object.entries(res.data).map(([code, name]) => ({
      label: `${code} - ${name}`,
      value: code,
    }))
    currencyOptions.value = monedas
  } catch {
    error.value = 'Error al cargar monedas'
  }
})

function convertir() {
  error.value = ''
  resultado.value = ''

  if (!amount.value || amount.value <= 0) {
    error.value = 'Ingrese un monto válido'
    return
  }

  if (!from.value || !to.value) {
    error.value = 'Seleccione ambas monedas'
    return
  }

  if (from.value === to.value) {
    resultado.value = `${amount.value} ${from.value} equivalen a ${amount.value} ${to.value}`
    return
  }

  axios
    .get(`https://api.frankfurter.app/latest`, {
      params: {
        amount: amount.value,
        from: from.value,
        to: to.value,
      },
    })
    .then((res) => {
      const rate = res.data.rates[to.value]
      resultado.value = `${amount.value} ${from.value} equivalen a ${rate} ${to.value}`
    })
    .catch(() => {
      error.value = 'Error en la conversión'
    })
}

function intercambiar() {
  const temp = from.value
  from.value = to.value
  to.value = temp
}
</script>
