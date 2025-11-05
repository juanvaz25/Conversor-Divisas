<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 via-indigo-50 to-purple-50 p-4">
    <div class="max-w-2xl mx-auto pt-8">
      <!-- Encabezado -->
      <div class="text-center mb-8">
        <div class="flex items-center justify-center gap-3 mb-2">
          <svg class="w-10 h-10 text-indigo-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"></path>
          </svg>
          <h1 class="text-4xl font-bold text-gray-800">Convertidor de Divisas</h1>
        </div>
        <p class="text-gray-600">Convierte entre las principales monedas del mundo</p>
      </div>

      <!-- Tarjeta Principal -->
      <div class="bg-white rounded-2xl shadow-xl p-8 mb-6">
        <!-- Input de Cantidad -->
        <div class="mb-6">
          <label class="block text-sm font-semibold text-gray-700 mb-2">
            Cantidad
          </label>
          <input
            type="number"
            v-model="cantidad"
            class="w-full px-4 py-3 text-2xl font-semibold border-2 border-gray-200 rounded-xl focus:border-indigo-500 focus:outline-none transition-colors"
            placeholder="0.00"
          />
        </div>

        <!-- Selectores de Moneda -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
          <div>
            <label class="block text-sm font-semibold text-gray-700 mb-2">
              De
            </label>
            <select
              v-model="monedaOrigen"
              class="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:border-indigo-500 focus:outline-none transition-colors text-lg font-medium"
            >
              <option v-for="moneda in monedas" :key="moneda.codigo" :value="moneda.codigo">
                {{ moneda.codigo }} - {{ moneda.nombre }}
              </option>
            </select>
          </div>

          <div>
            <label class="block text-sm font-semibold text-gray-700 mb-2">
              A
            </label>
            <select
              v-model="monedaDestino"
              class="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:border-indigo-500 focus:outline-none transition-colors text-lg font-medium"
            >
              <option v-for="moneda in monedas" :key="moneda.codigo" :value="moneda.codigo">
                {{ moneda.codigo }} - {{ moneda.nombre }}
              </option>
            </select>
          </div>
        </div>

        <!-- Botón Intercambiar -->
        <div class="flex justify-center mb-6">
          <button
            @click="intercambiarMonedas"
            class="flex items-center gap-2 px-6 py-3 bg-indigo-100 text-indigo-700 rounded-xl hover:bg-indigo-200 transition-colors font-semibold"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16V4m0 0L3 8m4-4l4 4m6 0v12m0 0l4-4m-4 4l-4-4"></path>
            </svg>
            Intercambiar
          </button>
        </div>

        <!-- Resultado -->
        <div class="bg-gradient-to-r from-indigo-500 to-purple-600 rounded-xl p-6 text-white">
          <div class="text-sm font-medium opacity-90 mb-2">Resultado</div>
          <div class="text-3xl font-bold mb-1">
            {{ obtenerSimbolo(monedaDestino) }} {{ resultado.toFixed(2) }}
          </div>
          <div class="text-sm opacity-80">
            {{ cantidad }} {{ monedaOrigen }} = {{ resultado.toFixed(2) }} {{ monedaDestino }}
          </div>
        </div>
      </div>

      <!-- Información de Tasas -->
      <div class="bg-white rounded-2xl shadow-lg p-6">
        <div class="flex items-center justify-between mb-4">
          <h2 class="text-lg font-bold text-gray-800">Información de Tasas</h2>
          <button
            @click="actualizarHora"
            class="flex items-center gap-2 px-4 py-2 bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition-colors text-sm font-medium"
          >
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path>
            </svg>
            Actualizar
          </button>
        </div>
        
        <div class="space-y-2 text-sm text-gray-600">
          <div class="flex justify-between">
            <span>Tasa de cambio:</span>
            <span class="font-semibold text-gray-800">
              1 {{ monedaOrigen }} = {{ tasaCambio.toFixed(4) }} {{ monedaDestino }}
            </span>
          </div>
          <div class="flex justify-between">
            <span>Última actualización:</span>
            <span class="font-semibold text-gray-800">
              {{ ultimaActualizacion }}
            </span>
          </div>
        </div>

        <!-- Conversiones Rápidas -->
        <div class="mt-6 pt-6 border-t border-gray-200">
          <h3 class="text-sm font-bold text-gray-700 mb-3">Conversiones Rápidas</h3>
          <div class="grid grid-cols-2 gap-3">
            <div v-for="valor in [1, 10, 100, 1000]" :key="valor" class="bg-gray-50 rounded-lg p-3">
              <div class="text-xs text-gray-600">{{ valor }} {{ monedaOrigen }}</div>
              <div class="text-sm font-bold text-indigo-600">
                {{ (valor * tasaCambio).toFixed(2) }} {{ monedaDestino }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="text-center mt-6 text-sm text-gray-600">
        <p>Las tasas de cambio son referenciales y pueden variar</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ConvertidorDivisas',
  data() {
    return {
      cantidad: 100,
      monedaOrigen: 'USD',
      monedaDestino: 'EUR',
      ultimaActualizacion: new Date().toLocaleTimeString(),
      
      // Tasas de cambio
      tasasCambio: {
        USD: { EUR: 0.92, GBP: 0.79, JPY: 149.50, ARS: 350.00, BRL: 4.95, MXN: 17.20, CAD: 1.36, AUD: 1.52, CHF: 0.88, CNY: 7.24 },
        EUR: { USD: 1.09, GBP: 0.86, JPY: 162.50, ARS: 380.50, BRL: 5.38, MXN: 18.70, CAD: 1.48, AUD: 1.65, CHF: 0.96, CNY: 7.87 },
        GBP: { USD: 1.27, EUR: 1.16, JPY: 189.00, ARS: 444.50, BRL: 6.28, MXN: 21.84, CAD: 1.73, AUD: 1.93, CHF: 1.12, CNY: 9.19 },
        JPY: { USD: 0.0067, EUR: 0.0062, GBP: 0.0053, ARS: 2.34, BRL: 0.033, MXN: 0.115, CAD: 0.0091, AUD: 0.010, CHF: 0.0059, CNY: 0.048 },
        ARS: { USD: 0.0029, EUR: 0.0026, GBP: 0.0022, JPY: 0.43, BRL: 0.014, MXN: 0.049, CAD: 0.0039, AUD: 0.0043, CHF: 0.0025, CNY: 0.021 },
        BRL: { USD: 0.202, EUR: 0.186, GBP: 0.159, JPY: 30.15, ARS: 70.71, MXN: 3.47, CAD: 0.275, AUD: 0.307, CHF: 0.178, CNY: 1.46 },
        MXN: { USD: 0.058, EUR: 0.053, GBP: 0.046, JPY: 8.70, ARS: 20.41, BRL: 0.288, CAD: 0.079, AUD: 0.088, CHF: 0.051, CNY: 0.421 },
        CAD: { USD: 0.735, EUR: 0.676, GBP: 0.578, JPY: 109.93, ARS: 256.41, BRL: 3.64, MXN: 12.66, AUD: 1.12, CHF: 0.647, CNY: 5.32 },
        AUD: { USD: 0.658, EUR: 0.606, GBP: 0.518, JPY: 98.36, ARS: 229.61, BRL: 3.26, MXN: 11.34, CAD: 0.893, CHF: 0.579, CNY: 4.76 },
        CHF: { USD: 1.136, EUR: 1.045, GBP: 0.893, JPY: 169.89, ARS: 396.59, BRL: 5.62, MXN: 19.61, CAD: 1.545, AUD: 1.726, CNY: 8.23 },
        CNY: { USD: 0.138, EUR: 0.127, GBP: 0.109, JPY: 20.65, ARS: 48.19, BRL: 0.685, MXN: 2.38, CAD: 0.188, AUD: 0.210, CHF: 0.122 }
      },
      
      // Lista de monedas
      monedas: [
        { codigo: 'USD', nombre: 'Dólar Estadounidense', simbolo: '$' },
        { codigo: 'EUR', nombre: 'Euro', simbolo: '€' },
        { codigo: 'GBP', nombre: 'Libra Esterlina', simbolo: '£' },
        { codigo: 'JPY', nombre: 'Yen Japonés', simbolo: '¥' },
        { codigo: 'ARS', nombre: 'Peso Argentino', simbolo: '$' },
        { codigo: 'BRL', nombre: 'Real Brasileño', simbolo: 'R$' },
        { codigo: 'MXN', nombre: 'Peso Mexicano', simbolo: '$' },
        { codigo: 'CAD', nombre: 'Dólar Canadiense', simbolo: 'C$' },
        { codigo: 'AUD', nombre: 'Dólar Australiano', simbolo: 'A$' },
        { codigo: 'CHF', nombre: 'Franco Suizo', simbolo: 'CHF' },
        { codigo: 'CNY', nombre: 'Yuan Chino', simbolo: '¥' }
      ]
    }
  },
  computed: {
    resultado() {
      const cantidadNum = parseFloat(this.cantidad) || 0;
      if (this.monedaOrigen === this.monedaDestino) {
        return cantidadNum;
      }
      return cantidadNum * this.tasaCambio;
    },
    tasaCambio() {
      if (this.monedaOrigen === this.monedaDestino) {
        return 1;
      }
      return this.tasasCambio[this.monedaOrigen][this.monedaDestino];
    }
  },
  methods: {
    intercambiarMonedas() {
      const temp = this.monedaOrigen;
      this.monedaOrigen = this.monedaDestino;
      this.monedaDestino = temp;
    },
    actualizarHora() {
      this.ultimaActualizacion = new Date().toLocaleTimeString();
    },
    obtenerSimbolo(codigo) {
      const moneda = this.monedas.find(m => m.codigo === codigo);
      return moneda ? moneda.simbolo : '';
    }
  }
}
</script>

<style scoped>
/* Estilos adicionales si son necesarios */
</style>