<template>
    <div class="bg-white shadow-md rounded-lg p-8 w-full max-w-md mx-auto">
      <h2 class="text-2xl font-bold mb-6 text-center">Juego Picas y Fijas</h2>
      <!-- Descripción y explicación del juego -->
      <div class="mb-6">
        <p class="text-gray-700 text-justify">
          El juego de "Picas y Fijas" consiste en adivinar un número secreto compuesto por dígitos únicos. 
          Primero, selecciona la cantidad de cifras (entre 2 y 9). Luego, intenta adivinar el número secreto ingresando tu intento.
        </p>
        <p class="text-gray-700 mt-2 text-justify">
          Cada intento te proporcionará dos pistas: 
          <strong>Fijas</strong> indican los dígitos que están en la posición correcta y 
          <strong>Picas</strong> los dígitos correctos pero en posiciones distintas. 
          Usa estas pistas para ir deduciendo el número correcto.
        </p>
      </div>
      <!-- Pantalla de configuración inicial -->
      <div v-if="!juegoIniciado">
        <label class="block text-lg mb-2" for="cifras">Cantidad de cifras (2-9):</label>
        <input
          v-model.number="cifras"
          id="cifras"
          type="number"
          min="2"
          max="9"
          class="border border-gray-300 rounded p-2 w-full mb-4 focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <button
          @click="iniciarJuego"
          class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded transition duration-300"
        >
          Iniciar Juego
        </button>
        <p v-if="mensaje" class="text-red-500 mt-2 text-center">{{ mensaje }}</p>
      </div>
      <!-- Pantalla del juego -->
      <div v-else>
        <div class="mb-4">
          <p class="mb-2 text-center">Intenta adivinar el número de {{ cifras }} cifras:</p>
          <input
            v-model="entrada"
            type="text"
            maxlength="9"
            placeholder="Ingresa tu número"
            class="border border-gray-300 rounded p-2 w-full focus:outline-none focus:ring-2 focus:ring-green-500"
          />
        </div>
        <button
          @click="procesarIntento"
          class="w-full bg-green-600 hover:bg-green-700 text-white py-2 px-4 rounded transition duration-300 mb-4"
        >
          Enviar
        </button>
        <div v-if="resultado">
          <p class="text-center">
            Tu intento: <span class="font-bold">{{ ultimoIntento }}</span>
          </p>
          <p class="text-center">
            Fijas: <span class="font-bold">{{ resultado.fijas }}</span>,
            Picas: <span class="font-bold">{{ resultado.picas }}</span>
          </p>
          <p class="text-center">Intentos: <span class="font-bold">{{ intentos }}</span></p>
        </div>
        <div v-if="juegoGanado" class="mt-4">
          <p class="text-center text-green-600 font-bold text-xl">¡Ganaste en {{ intentos }} intentos!</p>
          <p class="text-center mt-2">
            El número secreto era: <span class="font-bold">{{ numeroSecreto.join('') }}</span>
          </p>
        </div>
        <p v-if="mensaje" class="text-red-500 mt-2 text-center">{{ mensaje }}</p>
        <!-- Historial de intentos -->
        <div v-if="historial.length" class="mt-6">
          <h3 class="text-xl font-bold mb-2 text-center">Historial de Intentos</h3>
          <div class="overflow-x-auto">
            <table class="min-w-full text-left text-gray-600">
              <thead class="border-b">
                <tr>
                  <th class="px-2 py-2 font-medium text-gray-800">#</th>
                  <th class="px-2 py-2 font-medium text-gray-800">Número</th>
                  <th class="px-2 py-2 font-medium text-gray-800">Fijas</th>
                  <th class="px-2 py-2 font-medium text-gray-800">Picas</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200">
                <tr v-for="(attempt, index) in historial" :key="index">
                  <td class="px-2 py-2">{{ index + 1 }}</td>
                  <td class="px-2 py-2">{{ attempt.guess }}</td>
                  <td class="px-2 py-2">{{ attempt.fijas }}</td>
                  <td class="px-2 py-2">{{ attempt.picas }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const cifras = ref(4); // Valor predeterminado
  const juegoIniciado = ref(false);
  const juegoGanado = ref(false);
  const numeroSecreto = ref([]);
  const intentos = ref(0);
  const entrada = ref('');
  const mensaje = ref('');
  const resultado = ref(null);
  const ultimoIntento = ref('');
  const historial = ref([]); // Array para almacenar el historial de intentos
  
  // Función para generar el número secreto con dígitos únicos
  function generarNumeroSecreto(cantidad) {
    const digitos = Array.from({ length: 10 }, (_, i) => i);
    for (let i = digitos.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [digitos[i], digitos[j]] = [digitos[j], digitos[i]];
    }
    return digitos.slice(0, cantidad);
  }
  
  // Función para calcular picas y fijas
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
      mensaje.value = 'La cantidad de cifras debe estar entre 2 y 9.';
      return;
    }
    numeroSecreto.value = generarNumeroSecreto(cifras.value);
    juegoIniciado.value = true;
    juegoGanado.value = false;
    intentos.value = 0;
    entrada.value = '';
    mensaje.value = '';
    resultado.value = null;
    ultimoIntento.value = '';
    historial.value = []; // Reiniciamos el historial
  }
  
  function procesarIntento() {
    if (entrada.value.length !== cifras.value || isNaN(entrada.value)) {
      mensaje.value = `Debe ser un número de ${cifras.value} cifras.`;
      return;
    }
    // Guardamos el intento actual
    ultimoIntento.value = entrada.value;
    intentos.value++;
    
    const numeroJugador = Array.from(entrada.value).map(Number);
    if (numeroJugador.join('') === numeroSecreto.value.join('')) {
      // Juego ganado: agregamos el intento con todas las fijas
      historial.value.push({
        guess: ultimoIntento.value,
        fijas: cifras.value,
        picas: 0
      });
      juegoGanado.value = true;
      resultado.value = null;
      mensaje.value = '';
    } else {
      const res = calcularPicasFijas(numeroJugador, numeroSecreto.value);
      historial.value.push({
        guess: ultimoIntento.value,
        fijas: res.fijas,
        picas: res.picas
      });
      resultado.value = res;
      mensaje.value = '';
    }
    entrada.value = '';
  }
  </script>
  
  <style scoped>
  /* Puedes agregar estilos adicionales personalizados aquí si lo deseas */
  </style>
  