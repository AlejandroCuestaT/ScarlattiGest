<template>
  <div class="admin-espacios-container">
    <div v-if="!espacioSeleccionado">
      <div class="header-admin">
        <h2>🛠 Inventario de Espacios Técnicos</h2>
        <button @click="mostrarForm = !mostrarForm" class="btn-nuevo">
          {{ mostrarForm ? '✖ Cancelar' : '➕ Registrar Aula' }}
        </button>
      </div>

      <div v-if="mostrarForm" class="card-formulario-admin">
        <div class="grid-admin">
          <div class="campo">
            <label>ID Aula (Máx 10 car.)</label>
            <input v-model="nuevaAula.id" maxlength="10">
          </div>
          <div class="campo">
            <label>Nombre</label>
            <input v-model="nuevaAula.nombre">
          </div>
          <button @click="crearAula" class="btn-confirmar-alta">Confirmar Alta</button>
        </div>
      </div>

      <div class="grid-aulas">
        <div v-for="e in espacios" :key="e.id" class="card-aula">
          <div class="card-badge">ID: {{ e.id }}</div>
          <h3>{{ e.nombre }}</h3>
          <p>📍 {{ e.ubicacion_planta }} | 👥 {{ e.capacidad_max }}</p>
          <div class="acciones-card">
            <button @click="entrarEnEspacio(e)" class="btn-ver-reservas">📅 Ver Reservas</button>
            <button @click="eliminarAula(e.id)" class="btn-borrar">Eliminar</button>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="detalle-reservas">
      <button @click="espacioSeleccionado = null" class="btn-back">← Volver</button>
      <h2>Reservas: {{ espacioSeleccionado.nombre }}</h2>
      </div>
  </div>
</template>

<script>
export default {
  name: 'AdminEspacios',
  props: ['usuarioActivo'],
  data() {
    return {
      espacios: [],
      horarios: [],
      espacioSeleccionado: null,
      mostrarForm: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      nuevaAula: { id: '', nombre: '', ubicacion_planta: '', capacidad_max: 20, equipamiento: '' }
    }
  },
  mounted() {
    this.cargarDatosBase();
  },
  methods: {
    async cargarDatosBase() {
      const res = await fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`);
      this.espacios = await res.json();
    },

    // ESTA ES LA FUNCIÓN QUE FALTABA O DABA ERROR
    async eliminarAula(id) {
      if (!confirm(`¿Estás seguro de eliminar el aula con ID: ${id}?`)) return;

      try {
        const res = await fetch(`${this.apiUrl}/espacios/${id}?zusuario=${this.zusuario}`, {
          method: 'DELETE'
        });

        if (res.ok) {
          alert("Aula eliminada correctamente");
          await this.cargarDatosBase(); // Recarga la lista para actualizar la interfaz
        } else {
          const err = await res.json();
          alert("Error al eliminar: " + err.error);
        }
      } catch (e) {
        console.error("Error de red:", e);
        alert("No se pudo conectar con el servidor para eliminar.");
      }
    },

    async crearAula() {
      if (this.nuevaAula.id.length > 10) return alert("ID demasiado largo");
      const res = await fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ ...this.nuevaAula, zfecha: new Date().toISOString(), zusuario: this.zusuario })
      });
      if (res.ok) {
        this.mostrarForm = false;
        this.cargarDatosBase();
      }
    },

    entrarEnEspacio(e) {
      this.espacioSeleccionado = e;
    }
  }
}
</script>

<style scoped>
/* Mantén tus estilos anteriores aquí */
.btn-borrar { background: #fee2e2; color: #dc2626; border: none; padding: 10px; border-radius: 6px; cursor: pointer; }
.btn-borrar:hover { background: #fecaca; }
</style>