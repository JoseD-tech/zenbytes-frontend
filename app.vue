<script setup>
import { ref, onBeforeUnmount } from "vue";

// Datos iniciales
const datos = ref({
  title: "Análisis Hematológico",
  indicators: [
    { name: "Glóbulos Rojos", value: "0", color: "red" },
    { name: "Glóbulos Blancos", value: "0", color: "blue" },
    { name: "Plaquetas", value: "0", color: "purple" },
  ],
});

// Parámetros de configuración
const parameters = ref([
  {
    label: "Grosor de cuadros",
    value: 1,
    min: 1,
    max: 5,
    step: 0.1,
    unit: "px",
    key: "line_thickness",
  },
  {
    label: "Transparencia",
    value: 0.3,
    min: 0,
    max: 1,
    step: 0.1,
    unit: "%",
    key: "alpha",
  },
  {
    label: "Tamaño de fuente de etiquetas",
    value: 0.5,
    min: 0.1,
    max: 2,
    step: 0.1,
    unit: "px",
    key: "font_scale",
  },
  {
    label: "Grosor de fuente",
    value: 1,
    min: 1,
    max: 5,
    step: 0.1,
    unit: "px",
    key: "font_thickness",
  },
]);

// Referencias para el input de archivo, archivo seleccionado, vista previa y respuesta de la API
const fileInput = ref(null);
const selectedFile = ref(null);
const previewUrl = ref(null);
const response = ref(null);
const estado = ref(false);

// Maneja la selección del archivo
const handleFileChange = async (event) => {
  const file = event.target.files[0];
  if (file) {
    selectedFile.value = file;
    previewUrl.value = URL.createObjectURL(file); // Crear URL de vista previa
  }
};

// Procesa la imagen enviándola a la API
const processImage = async () => {
  if (!selectedFile.value) return;
  // Activar el estado de carga
  estado.value = true;
  // Crear FormData y agregar el archivo
  const formData = new FormData();
  formData.append("file", selectedFile.value);

  // Crear objeto de configuración
  const config = {
    line_thickness: parameters.value.find((p) => p.key === "line_thickness")
      .value,
    alpha: parameters.value.find((p) => p.key === "alpha").value,
    font_scale: parameters.value.find((p) => p.key === "font_scale").value,
    font_thickness: parameters.value.find((p) => p.key === "font_thickness")
      .value,
  };

  // Agregar configuración como un campo JSON
  formData.append("config", JSON.stringify(config));

  try {
    // Enviar la solicitud a la API
    const result = await fetch("https://dev.galysa.com/modelocelulas/detect/", {
      method: "POST",
      body: formData,
    });

    // Manejar errores de la respuesta
    if (!result.ok) {
      throw new Error(`Error: ${result.statusText}`);
    }

    // Procesar la respuesta
    response.value = await result.json();
    console.log("Respuesta de la API:", response.value);

    // Actualizar los indicadores con los datos de la respuesta
    if (response.value.detections) {
      datos.value.indicators[0].value = response.value.detections.filter(
        (d) => d.class === "RBC"
      ).length;
      datos.value.indicators[1].value = response.value.detections.filter(
        (d) => d.class === "WBC"
      ).length;
      datos.value.indicators[2].value = response.value.detections.filter(
        (d) => d.class === "Platelet"
      ).length;
    }
  } catch (error) {
    console.error("Error al subir la imagen:", error);
    alert("Hubo un error al procesar la imagen.");
  } finally {
    // Desactivar el estado de carga (se ejecuta siempre, haya éxito o error)
    estado.value = false;
  }
};

// Limpiar la URL del objeto cuando el componente se desmonte
onBeforeUnmount(() => {
  if (previewUrl.value) {
    URL.revokeObjectURL(previewUrl.value);
  }
});
</script>

<template>
  <div class="min-h-screen bg-gray-50 p-8">
    <h1 class="text-3xl font-bold mb-8">{{ datos.title }}</h1>

    <!-- Indicadores principales -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
      <!-- Glóbulos Rojos -->
      <div class="bg-white rounded-lg p-6 shadow-sm">
        <div class="flex items-center gap-2 mb-2">
          <div
            class="w-8 h-8 rounded-full bg-red-100 flex items-center justify-center"
          >
            <div class="w-4 h-4 rounded-full bg-red-500"></div>
          </div>
          <h2 class="text-lg font-semibold">{{ datos.indicators[0].name }}</h2>
        </div>
        <div class="mt-2">
          <p class="text-3xl font-bold">{{ datos.indicators[0].value }}</p>
        </div>
      </div>

      <!-- Glóbulos Blancos -->
      <div class="bg-white rounded-lg p-6 shadow-sm">
        <div class="flex items-center gap-2 mb-2">
          <div
            class="w-8 h-8 rounded-full bg-blue-100 flex items-center justify-center"
          >
            <div class="w-4 h-4 rounded-full bg-blue-500"></div>
          </div>
          <h2 class="text-lg font-semibold">{{ datos.indicators[1].name }}</h2>
        </div>
        <div class="mt-2">
          <p class="text-3xl font-bold">{{ datos.indicators[1].value }}</p>
        </div>
      </div>

      <!-- Plaquetas -->
      <div class="bg-white rounded-lg p-6 shadow-sm">
        <div class="flex items-center gap-2 mb-2">
          <div
            class="w-8 h-8 rounded-full bg-purple-100 flex items-center justify-center"
          >
            <div class="w-4 h-4 rounded-full bg-purple-500"></div>
          </div>
          <h2 class="text-lg font-semibold">{{ datos.indicators[2].name }}</h2>
        </div>
        <div class="mt-2">
          <p class="text-3xl font-bold">{{ datos.indicators[2].value }}</p>
        </div>
      </div>
    </div>

    <!-- Análisis de Imagen -->
    <div class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-xl font-bold mb-6">Análisis de Imagen</h2>
      <div
        v-if="estado"
        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
      >
        <div class="bg-white p-6 rounded-lg shadow-lg">
          <p class="text-lg font-semibold">Procesando imagen...</p>
          <div class="mt-4 flex justify-center">
            <div
              class="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full animate-spin"
            ></div>
          </div>
        </div>
      </div>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <!-- Panel izquierdo -->
        <div>
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2"
              >Subir Imagen</label
            >
            <div class="relative">
              <input
                type="file"
                ref="fileInput"
                @change="handleFileChange"
                accept="image/*"
                class="hidden"
              />
              <div class="flex gap-2">
                <button
                  @click="$refs.fileInput.click()"
                  class="flex-1 border rounded-lg px-4 py-2 bg-gray-50 text-left hover:bg-gray-100 transition-colors"
                >
                  <span class="text-gray-500">{{
                    selectedFile ? selectedFile.name : "Seleccionar archivo..."
                  }}</span>
                </button>
                <button
                  class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
                  @click="processImage"
                  :disabled="!selectedFile"
                >
                  Procesar
                </button>
              </div>
            </div>
          </div>

          <div class="space-y-6">
            <h3 class="font-semibold">Parámetros de Configuración</h3>

            <!-- Controles deslizantes -->
            <div
              v-for="(param, index) in parameters"
              :key="index"
              class="space-y-2"
            >
              <div class="flex justify-between">
                <label class="text-sm font-medium text-gray-700">{{
                  param.label
                }}</label>
                <span class="text-sm text-gray-500"
                  >{{ param.value }}{{ param.unit }}</span
                >
              </div>
              <input
                type="range"
                v-model="param.value"
                :min="param.min"
                :max="param.max"
                :step="param.step"
                class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
              />
            </div>
          </div>
        </div>

        <!-- Panel derecho -->
        <div>
          <div v-if="response && response.image_base64" class="mt-8">
            <h3 class="text-xl font-bold mb-4">Imagen Procesada</h3>
            <div
              class="border rounded-lg overflow-hidden bg-gray-50 aspect-square flex items-center justify-center"
            >
              <img
                :src="`data:image/jpeg;base64,${response.image_base64}`"
                alt="Imagen procesada"
                class="max-w-full max-h-full object-contain"
              />
            </div>
          </div>

          <div v-else>
            <h3 class="font-semibold mb-4">Imagen Original</h3>
            <div
              class="border rounded-lg overflow-hidden bg-gray-50 aspect-square flex items-center justify-center"
            >
              <img
                v-if="previewUrl"
                :src="previewUrl"
                alt="Muestra de sangre"
                class="max-w-full max-h-full object-contain"
              />
              <div v-else class="text-gray-400 text-center p-8">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-12 w-12 mx-auto mb-2"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke="currentColor"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
                  />
                </svg>
                <p>No hay imagen seleccionada</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
