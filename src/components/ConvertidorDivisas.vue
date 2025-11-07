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
        <p class="text-gray-600">Aplicación realizada por Juan Vazquez</p>
        <div class="mt-2 inline-flex items-center gap-2 px-3 py-1 bg-green-100 text-green-700 rounded-full text-sm font-medium">
          <span class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
          Tasas en tiempo real
        </div>
      </div>

      <!-- Mensaje de error -->
      <div v-if="error" class="mb-6 bg-red-50 border-l-4 border-red-500 p-4 rounded-lg">
        <p class="text-red-700">{{ error }}</p>
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
          <div v-if="cargando" class="flex items-center gap-3">
            <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-white"></div>
            <span class="text-xl">Cargando tasas...</span>
          </div>
          <div v-else>
            <div class="text-3xl font-bold mb-1">
              {{ obtenerSimbolo(monedaDestino) }} {{ resultado.toFixed(2) }}
            </div>
            <div class="text-sm opacity-80">
              {{ cantidad }} {{ monedaOrigen }} = {{ resultado.toFixed(2) }} {{ monedaDestino }}
            </div>
          </div>
        </div>
      </div>

      <!-- Información de Tasas -->
      <div class="bg-white rounded-2xl shadow-lg p-6">
        <div class="flex items-center justify-between mb-4">
          <h2 class="text-lg font-bold text-gray-800">Información de Tasas</h2>
          <button
            @click="obtenerTasasCambio"
            :disabled="cargando"
            class="flex items-center gap-2 px-4 py-2 bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition-colors text-sm font-medium disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <svg 
              class="w-4 h-4"
              :class="{ 'animate-spin': cargando }"
              fill="none" 
              stroke="currentColor" 
              viewBox="0 0 24 24"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path>
            </svg>
            {{ cargando ? 'Actualizando...' : 'Actualizar' }}
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
              {{ ultimaActualizacion || 'Cargando...' }}
            </span>
          </div>
        </div>

        <!-- Conversiones Rápidas -->
        <div v-if="!cargando" class="mt-6 pt-6 border-t border-gray-200">
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
        <p>Tasas de cambio actualizadas en tiempo real desde ExchangeRate-API</p>
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
      tasasCambio: {},
      ultimaActualizacion: '',
      cargando: true,
      error: '',
      
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
      const tasa = this.tasasCambio[this.monedaDestino] || 0;
      return cantidadNum * tasa;
    },
    tasaCambio() {
      if (this.monedaOrigen === this.monedaDestino) {
        return 1;
      }
      return this.tasasCambio[this.monedaDestino] || 0;
    }
  },
  watch: {
    monedaOrigen() {
      this.obtenerTasasCambio();
    }
  },
  mounted() {
    this.obtenerTasasCambio();
  },
  methods: {
    async obtenerTasasCambio() {
      this.cargando = true;
      this.error = '';
      try {
        const response = await fetch(`https://api.exchangerate-api.com/v4/latest/${this.monedaOrigen}`);
        const data = await response.json();
        
        if (data && data.rates) {
          this.tasasCambio = data.rates;
          this.ultimaActualizacion = new Date().toLocaleString('es-ES', {
            hour: '2-digit',
            minute: '2-digit',
            second: '2-digit',
            day: '2-digit',
            month: '2-digit',
            year: 'numeric'
          });
        }
      } catch (err) {
        this.error = 'Error al obtener las tasas de cambio. Por favor, intenta nuevamente.';
        console.error('Error:', err);
      } finally {
        this.cargando = false;
      }
    },
    intercambiarMonedas() {
      const temp = this.monedaOrigen;
      this.monedaOrigen = this.monedaDestino;
      this.monedaDestino = temp;
    },
    obtenerSimbolo(codigo) {
      const moneda = this.monedas.find(m => m.codigo === codigo);
      return moneda ? moneda.simbolo : '';
    }
  }
}
</script>

<style scoped>
.animate-spin {
  animation: spin 1s linear infinite;
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}
</style>