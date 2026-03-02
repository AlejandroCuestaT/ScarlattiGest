<template>
  <div class="etapasContainer">
    <div class="cabeceraSeccion">
      <h2>Etapas Educativas</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btnToggle">
        {{ mostrarFormulario ? 'Cancelar' : 'Nueva Etapa' }}
      </button>
    </div>

    <transition name="fade">
      <div v-if="mostrarFormulario" class="formularioCard">
        <h3>Añadir Nueva Etapa</h3>
        <form @submit.prevent="guardarEtapa">
          <div class="formGroup">
            <label>ID de Etapa (Código único):</label>
            <input v-model="formEtapa.id" placeholder="Ej: BACH, ESO, CYBER" required @input="formEtapa.id = formEtapa.id.toUpperCase()">
          </div>
          <div class="formGroup">
            <label>Nombre:</label>
            <input v-model="formEtapa.nombre" placeholder="Ej: Bachillerato" required>
          </div>
          <div class="formGroup">
            <label>Descripción:</label>
            <textarea v-model="formEtapa.descripcion" placeholder="Breve descripción de la etapa" required></textarea>
          </div>
          <button type="submit" class="btnSave">Guardar Etapa</button>
        </form>
      </div>
    </transition>

    <div class="gridEtapas">
      <div v-for="etapa in listaEtapas" :key="etapa.id" class="tarjetaEtapa">
        <div class="badgeId">ID: {{ etapa.id || 'S/N' }}</div>
        <h4>{{ etapa.nombre }}</h4>
        <p>{{ etapa.descripcion }}</p>
        <div class="accionesCard">
          <button @click="eliminarEtapa(etapa.id)" class="btnEliminar">
            Eliminar Etapa
          </button>
        </div>
      </div>
    </div>

    <div v-if="listaEtapas.length === 0" class="noData">
      No se encontraron etapas educativas cargadas en el sistema.
    </div>
  </div>
</template>

<script>
export default {
  name: 'EtapasComponent',
  data() {
    return {
      // Configuración de endpoint y zusuario
      apiUrl: 'http://44.207.19.239:3000/etapas',
      zusuario: 'acuesta',
      // Estado de datos
      listaEtapas: [],               
      mostrarFormulario: false,
      // Modelo para el formulario de alta
      formEtapa: { id: '', nombre: '', descripcion: '' }
    }
  },
  mounted() {
    // Carga automática al iniciar el componente
    this.cargarEtapas();
  },
  methods: {
    // Recupera la lista de etapas desde el servidor
    async cargarEtapas() {
      try {
        const respuesta = await fetch(`${this.apiUrl}?zusuario=${this.zusuario}`);
        if (!respuesta.ok) throw new Error("Error al obtener datos");
        this.listaEtapas = await respuesta.json();
      } catch (error) {
        console.error('Error al cargar etapas:', error);
      }
    },

    // Envía la nueva etapa validando que el ID no esté vacío
    async guardarEtapa() {
      if (!this.formEtapa.id.trim()) {
        return alert("El código ID es obligatorio para identificar la etapa.");
      }

      // Estructura de datos con spread y metadatos de auditoría
      const payload = { 
        ...this.formEtapa,
        id: this.formEtapa.id.trim(), 
        zusuario: this.zusuario,
        zfecha: new Date().toISOString()
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          this.mostrarFormulario = false;
          this.resetFormulario();
          await this.cargarEtapas(); // Refrescamos la vista
        } else {
          const errorJson = await respuesta.json();
          alert("Error de la API: " + (errorJson.error || "Datos no válidos"));
        }
      } catch (error) {
        alert("Fallo de conexión al intentar guardar.");
      }
    },

    // Elimina un registro tras confirmar y verificar el ID
    async eliminarEtapa(id) {
      if (!id) {
        return alert("Error: ID no válido para eliminación.");
      }

      if (!confirm(`¿Estás seguro de eliminar la etapa "${id}"?`)) return;

      try {
        // Formato de URL: BASE_URL/ID?zusuario=USUARIO
        const urlDelete = `${this.apiUrl}/${id}?zusuario=${this.zusuario}`;
        const respuesta = await fetch(urlDelete, { method: 'DELETE' });
        
        if (respuesta.ok) {
          await this.cargarEtapas();
        } else {
          alert("El servidor no pudo encontrar el registro solicitado.");
        }
      } catch (error) {
        console.error('Error al eliminar:', error);
      }
    },

    // Limpia el objeto del formulario
    resetFormulario() {
      this.formEtapa = { id: '', nombre: '', descripcion: '' };
    }
  }
}
</script>

<style scoped>
.etapasContainer { padding: 20px; background: #f4f7f9; min-height: 80vh; }
.cabeceraSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.btnToggle { background: #3b82f6; color: white; border: none; padding: 10px 15px; border-radius: 8px; cursor: pointer; font-weight: 500; }

.formularioCard { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); margin-bottom: 30px; }
.formGroup { margin-bottom: 15px; }
.formGroup label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 14px; color: #475569; }
.formGroup input, .formGroup textarea { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; }
.btnSave { background: #10b981; color: white; border: none; padding: 12px; width: 100%; border-radius: 6px; font-weight: bold; cursor: pointer; transition: background 0.2s; }
.btnSave:hover { background: #059669; }

.gridEtapas { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
.tarjetaEtapa { background: white; border-radius: 10px; padding: 20px; border-left: 5px solid #3b82f6; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
.badgeId { font-size: 10px; color: #94a3b8; font-weight: bold; margin-bottom: 10px; text-transform: uppercase; }
.tarjetaEtapa h4 { margin: 0 0 10px 0; color: #1e293b; font-size: 18px; }
.tarjetaEtapa p { font-size: 14px; color: #64748b; line-height: 1.5; }

.accionesCard { margin-top: 15px; border-top: 1px solid #f1f5f9; padding-top: 12px; }
.btnEliminar { background: none; border: none; color: #ef4444; font-weight: 600; cursor: pointer; font-size: 13px; }
.btnEliminar:hover { text-decoration: underline; }

.noData { text-align: center; margin-top: 50px; color: #94a3b8; font-style: italic; }

/* Animación simple para el formulario */
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter, .fade-leave-to { opacity: 0; }
</style>