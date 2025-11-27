<template>
  <div class="min-h-screen bg-gradient-to-br from-emerald-100 via-teal-50 to-cyan-100 flex items-center justify-center p-4">
    <div class="w-full max-w-lg">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl font-bold bg-gradient-to-r from-emerald-600 to-teal-600 bg-clip-text text-transparent mb-2 p-4">
          Picas y Fijas
        </h1>
      </div>

      <!-- Card Principal -->
      <div class="bg-white/80 backdrop-blur-lg rounded-3xl shadow-2xl p-8">
        <!-- Pantalla de inicio -->
        <div v-if="!juegoIniciado" class="space-y-6">
          <div class="text-center space-y-2">
            <div class="inline-block p-3 bg-emerald-100 rounded-full mb-4">
              <svg class="w-8 h-8 text-emerald-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
              </svg>
            </div>
            <h2 class="text-2xl font-semibold text-gray-800">¿Listos para jugar?</h2>
            <p class="text-gray-600 text-sm">Adivina el número secreto</p>
          </div>

          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-2">
                Número de cifras
              </label>
              <div class="flex gap-2">
                <button
                  v-for="num in [2, 3, 4, 5, 6]"
                  :key="num"
                  @click="cifras = num"
                  :class="[
                    'flex-1 py-3 rounded-xl font-semibold transition-all',
                    cifras === num
                      ? 'bg-gradient-to-r from-emerald-600 to-teal-600 text-white shadow-lg scale-105'
                      : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                  ]"
                >
                  {{ num }}
                </button>
              </div>
            </div>

            <button
              @click="iniciarJuego"
              class="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-4 rounded-xl font-semibold shadow-lg hover:shadow-xl transition-all hover:scale-105"
            >
              Comenzar Juego
            </button>

            <button
              @click="mostrarReglas = !mostrarReglas"
              class="w-full text-sm text-emerald-600 hover:text-emerald-700 font-medium"
            >
              {{ mostrarReglas ? '▼ Ocultar reglas' : '▶ ¿Cómo se juega?' }}
            </button>

            <div v-if="mostrarReglas" class="bg-emerald-50 rounded-xl p-4 space-y-2 text-sm text-gray-700">
              <p><strong class="text-emerald-600">🎯 Objetivo:</strong> Adivina el número secreto de dígitos únicos</p>
              <p><strong class="text-teal-600">📍 Fijas:</strong> Dígitos correctos en la posición correcta</p>
              <p><strong class="text-cyan-600">🎲 Picas:</strong> Dígitos correctos pero en posición incorrecta</p>
            </div>

            <p v-if="mensaje" class="text-red-500 text-sm text-center">{{ mensaje }}</p>
          </div>
        </div>

        <!-- Pantalla de juego -->
        <div v-else class="space-y-6">
          <div v-if="!juegoGanado">
            <!-- Contador de intentos -->
            <div class="flex justify-between items-center mb-6">
              <div class="text-sm text-gray-600">
                Intento <span class="font-bold text-emerald-600">#{{ intentos + 1 }}</span>
              </div>
              <button
                @click="reiniciarJuego"
                class="text-sm text-gray-500 hover:text-gray-700"
              >
                Nueva partida
              </button>
            </div>

            <!-- Input de entrada -->
            <div class="space-y-3">
              <input
                v-model="entrada"
                @keypress.enter="procesarIntento"
                type="text"
                :maxlength="cifras"
                :placeholder="`${cifras} dígitos...`"
                class="w-full text-center text-3xl font-bold tracking-widest py-4 px-4 rounded-xl border-2 border-gray-200 focus:border-emerald-400 focus:outline-none transition-colors"
              />
              
              <button
                @click="procesarIntento"
                class="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-4 rounded-xl font-semibold shadow-lg hover:shadow-xl transition-all hover:scale-105"
              >
                Enviar
              </button>
            </div>

            <div v-if="mensaje" class="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded-xl text-sm text-center">
              {{ mensaje }}
            </div>

            <!-- Resultado del último intento -->
            <div v-if="resultado" class="bg-gradient-to-r from-emerald-50 to-teal-50 rounded-xl p-6">
              <div class="grid grid-cols-2 gap-4">
                <div class="text-center">
                  <div class="text-3xl font-bold text-emerald-600">{{ resultado.fijas }}</div>
                  <div class="text-sm text-gray-600 mt-1">Fijas</div>
                </div>
                <div class="text-center">
                  <div class="text-3xl font-bold text-teal-600">{{ resultado.picas }}</div>
                  <div class="text-sm text-gray-600 mt-1">Picas</div>
                </div>
              </div>
            </div>
          </div>

          <!-- Pantalla de victoria -->
          <div v-else class="text-center space-y-6">
            <div class="text-6xl">🎉</div>
            <div>
              <h3 class="text-3xl font-bold bg-gradient-to-r from-emerald-600 to-teal-600 bg-clip-text text-transparent mb-2">
                ¡Lo lograste!
              </h3>
              <p class="text-gray-600">en {{ intentos }} {{ intentos === 1 ? 'intento' : 'intentos' }}</p>
            </div>
            <div class="bg-gradient-to-r from-emerald-50 to-teal-50 rounded-xl p-6">
              <p class="text-sm text-gray-600 mb-2">El número secreto era</p>
              <p class="text-4xl font-bold tracking-widest text-emerald-600">
                {{ numeroSecreto.join('') }}
              </p>
            </div>
            <button
              @click="reiniciarJuego"
              class="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-4 rounded-xl font-semibold shadow-lg hover:shadow-xl transition-all hover:scale-105"
            >
              Jugar de nuevo
            </button>
          </div>

          <!-- Historial -->
          <div v-if="historial.length > 0" class="space-y-3">
            <h3 class="text-sm font-semibold text-gray-700">Historial</h3>
            <div class="space-y-2 max-h-64 overflow-y-auto">
              <div
                v-for="(item, idx) in historial"
                :key="idx"
                class="bg-gray-50 rounded-lg p-3 flex items-center justify-between"
              >
                <span class="text-xs text-gray-500">#{{ idx + 1 }}</span>
                <span class="font-mono font-bold text-lg tracking-wider">{{ item.guess }}</span>
                <div class="flex gap-3 text-sm">
                  <span class="text-emerald-600 font-semibold">{{ item.fijas }}F</span>
                  <span class="text-teal-600 font-semibold">{{ item.picas }}P</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const cifras = ref(4);
const juegoIniciado = ref(false);
const juegoGanado = ref(false);
const numeroSecreto = ref([]);
const intentos = ref(0);
const entrada = ref('');
const mensaje = ref('');
const resultado = ref(null);
const historial = ref([]);
const mostrarReglas = ref(false);

function generarNumeroSecreto(cantidad) {
  const digitos = Array.from({ length: 10 }, (_, i) => i);
  for (let i = digitos.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [digitos[i], digitos[j]] = [digitos[j], digitos[i]];
  }
  return digitos.slice(0, cantidad);
}

function calcularPicasFijas(numeroUsuario, numeroSecreto) {
  let fijas = 0;
  let picas = 0;
  for (let i = 0; i < numeroSecreto.length; i++) {
    if (numeroUsuario.includes(numeroSecreto[i])) {
      if (numeroUsuario[i] === numeroSecreto[i]) {
        fijas++;
      } else {
        picas++;
      }
    }
  }
  return { picas, fijas };
}

function iniciarJuego() {
  if (cifras.value < 2 || cifras.value > 9) {
    mensaje.value = 'La cantidad debe estar entre 2 y 9 cifras';
    return;
  }
  numeroSecreto.value = generarNumeroSecreto(cifras.value);
  juegoIniciado.value = true;
  juegoGanado.value = false;
  intentos.value = 0;
  entrada.value = '';
  mensaje.value = '';
  resultado.value = null;
  historial.value = [];
}

function procesarIntento() {
  // Solo permitir números
  entrada.value = entrada.value.replace(/\D/g, '');
  
  if (entrada.value.length !== cifras.value) {
    mensaje.value = `Ingresa exactamente ${cifras.value} dígitos`;
    return;
  }

  const numeroJugador = Array.from(entrada.value).map(Number);
  intentos.value++;

  if (numeroJugador.join('') === numeroSecreto.value.join('')) {
    historial.value.push({ guess: entrada.value, fijas: cifras.value, picas: 0 });
    juegoGanado.value = true;
    resultado.value = null;
    mensaje.value = '';
  } else {
    const res = calcularPicasFijas(numeroJugador, numeroSecreto.value);
    historial.value.push({ guess: entrada.value, fijas: res.fijas, picas: res.picas });
    resultado.value = res;
    mensaje.value = '';
  }
  entrada.value = '';
}

function reiniciarJuego() {
  juegoIniciado.value = false;
  juegoGanado.value = false;
  numeroSecreto.value = [];
  intentos.value = 0;
  entrada.value = '';
  mensaje.value = '';
  resultado.value = null;
  historial.value = [];
}
</script>

