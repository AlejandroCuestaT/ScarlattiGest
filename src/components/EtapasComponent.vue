<template>
  <div class="etapas-container">
    <div class="cabecera">
      <h2>🎓 Etapas Educativas</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btn-toggle">
        {{ mostrarFormulario ? '✖️ Cancelar' : '➕ Nueva Etapa' }}
      </button>
    </div>

    <transition name="fade">
      <div v-if="mostrarFormulario" class="formulario-card">
        <h3>Añadir Nueva Etapa</h3>
        <form @submit.prevent="guardar">
          <div class="form-group">
            <label>ID de Etapa (Código único):</label>
            <input v-model="form.id" placeholder="Ej: BACH, ESO, CYBER" required @input="form.id = form.id.toUpperCase()">
          </div>
          <div class="form-group">
            <label>Nombre:</label>
            <input v-model="form.nombre" placeholder="Ej: Bachillerato" required>
          </div>
          <div class="form-group">
            <label>Descripción:</label>
            <textarea v-model="form.descripcion" placeholder="Breve descripción de la etapa" required></textarea>
          </div>
          <button type="submit" class="btn-save">💾 Guardar Etapa</button>
        </form>
      </div>
    </transition>

    <div class="grid-etapas">
      <div v-for="etapa in etapas" :key="etapa.id" class="tarjeta-etapa">
        <div class="badge-id">ID: {{ etapa.id || 'S/N' }}</div>
        <h4>{{ etapa.nombre }}</h4>
        <p>{{ etapa.descripcion }}</p>
        <div class="acciones">
          <button @click="eliminar(etapa.id)" class="btn-eliminar">
            🗑️ Eliminar
          </button>
        </div>
      </div>
    </div>

    <div v-if="etapas.length === 0" class="no-data">
      No se encontraron etapas educativas cargadas.
    </div>
  </div>
</template>

<script>
export default {
  name: 'EtapasComponent',
  data() {
    return {
      // IP corregida para evitar errores de conexión
      url: 'http://44.207.19.239:3000/etapas',
      zusuario: 'acuesta',
      etapas: [],               
      mostrarFormulario: false,
      form: { id: '', nombre: '', descripcion: '' }
    }
  },
  mounted() {
    this.cargar();
  },
  methods: {
    async cargar() {
      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        if (!res.ok) throw new Error("Error al obtener datos");
        this.etapas = await res.json();
      } catch (e) {
        console.error('Error al cargar etapas:', e);
      }
    },
    async guardar() {
      // Bloqueo de seguridad: Evita enviar IDs vacíos que rompen la vista
      if (!this.form.id.trim()) {
        return alert("Error: El ID es obligatorio para identificar la etapa.");
      }

      const payload = { 
        id: this.form.id.trim(), 
        nombre: this.form.nombre,
        descripcion: this.form.descripcion,
        zusuario: this.zusuario,
        zfecha: new Date().toISOString()
      };

      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          this.mostrarFormulario = false;
          this.form = { id: '', nombre: '', descripcion: '' };
          await this.cargar();
        } else {
          const err = await res.json();
          alert("Error de la API: " + (err.error || "Datos no válidos"));
        }
      } catch (e) {
        alert("Fallo de conexión al guardar.");
      }
    },
    async eliminar(id) {
      // Si el ID está vacío (como el de tu prueba), avisamos al usuario
      if (!id) {
        return alert("Este registro tiene un ID corrupto. Intenta borrarlo desde la consola del navegador.");
      }

      if (!confirm(`¿Estás seguro de eliminar la etapa "${id}"?`)) return;

      // Ruta dinámica correcta: BASE_URL/ID?zusuario=USUARIO
      const urlDelete = `${this.url}/${id}?zusuario=${this.zusuario}`;

      try {
        const res = await fetch(urlDelete, { method: 'DELETE' });
        if (res.ok) {
          await this.cargar();
        } else {
          // Captura el 404 si el ID no existe en el servidor
          alert("La API no pudo encontrar el registro para borrarlo.");
        }
      } catch (e) {
        console.error('Error al eliminar:', e);
      }
    }
  }
}
</script>

<style scoped>
.etapas-container { padding: 20px; font-family: sans-serif; background: #f4f7f9; min-height: 80vh; }
.cabecera { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.btn-toggle { background: #3b82f6; color: white; border: none; padding: 10px 15px; border-radius: 8px; cursor: pointer; }

.formulario-card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); margin-bottom: 30px; }
.form-group { margin-bottom: 15px; }
.form-group label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 14px; }
.form-group input, .form-group textarea { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; }
.btn-save { background: #10b981; color: white; border: none; padding: 12px; width: 100%; border-radius: 6px; font-weight: bold; cursor: pointer; }

.grid-etapas { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
.tarjeta-etapa { background: white; border-radius: 10px; padding: 20px; border-left: 5px solid #3b82f6; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
.badge-id { font-size: 10px; color: #94a3b8; font-weight: bold; margin-bottom: 10px; }
.tarjeta-etapa h4 { margin: 0 0 10px 0; color: #1e293b; }
.tarjeta-etapa p { font-size: 14px; color: #64748b; line-height: 1.4; }
.acciones { margin-top: 15px; border-top: 1px solid #f1f5f9; padding-top: 10px; }
.btn-eliminar { background: none; border: none; color: #ef4444; font-weight: bold; cursor: pointer; font-size: 13px; }

.no-data { text-align: center; margin-top: 50px; color: #94a3b8; }
</style>