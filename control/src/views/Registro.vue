<template>
  <div class="registro">
    <h2>Registrar Persona</h2>
    <video ref="video" autoplay playsinline class="video"></video>
    <p class="estado">{{ estado }}</p>

    <div class="form">
      <input v-model="nombre" placeholder="Nombre" />
      <input v-model="id" placeholder="ID" />
      <button @click="guardarPersona" :disabled="!rostroDetectado">Guardar Registro</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const video = ref(null);
const nombre = ref("");
const id = ref("");
const estado = ref("Cargando modelos...");
const rostroDetectado = ref(false);
let descriptorActual = null;

// Cargar modelos desde /public/models
async function cargarModelos() {
  await faceapi.nets.tinyFaceDetector.loadFromUri("/models");
  await faceapi.nets.faceLandmark68Net.loadFromUri("/models");
  await faceapi.nets.faceRecognitionNet.loadFromUri("/models");
}

async function iniciarCamara() {
  const stream = await navigator.mediaDevices.getUserMedia({ video: true });
  video.value.srcObject = stream;
}

function detectarRostro() {
  setInterval(async () => {
    const deteccion = await faceapi
      .detectSingleFace(video.value, new faceapi.TinyFaceDetectorOptions())
      .withFaceLandmarks()
      .withFaceDescriptor();

    if (deteccion) {
      estado.value = "✔ Rostro detectado";
      rostroDetectado.value = true;
      descriptorActual = Array.from(deteccion.descriptor);
    } else {
      estado.value = "Buscando rostro...";
      rostroDetectado.value = false;
    }
  }, 700);
}

function guardarPersona() {
  if (!nombre.value || !id.value || !descriptorActual) {
    alert("Falta nombre, ID o rostro");
    return;
  }

  const personas = JSON.parse(localStorage.getItem("personas") || "[]");
  personas.push({ nombre: nombre.value, id: id.value, descriptor: descriptorActual });
  localStorage.setItem("personas", JSON.stringify(personas));

  alert("Persona registrada correctamente");
  nombre.value = "";
  id.value = "";
  rostroDetectado.value = false;
}

onMounted(async () => {
  await cargarModelos();
  await iniciarCamara();
  detectarRostro();
});
</script>

<style>
.registro {
  text-align: center;
  margin-top: 20px;
}
.video {
  width: 300px;
  border: 2px solid #4b9be0;
  border-radius: 10px;
  margin-bottom: 10px;
}
.form {
  margin-top: 15px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: center;
}
input {
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #ccc;
}
button {
  padding: 8px 12px;
  background: #4b9be0;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
button:hover {
  background: #357abd;
}
.estado {
  font-weight: bold;
  margin-bottom: 10px;
}
</style>
