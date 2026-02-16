<template>
  <div class="gestion-container">
    <div class="header-seccion">
      <h2>Niveles Educativos (Etapas)</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-principal">
        {{ mostrarForm ? 'Cerrar' : 'Nueva Etapa' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="formulario-card">
      <h3>Registrar Etapa</h3>
      <form @submit.prevent="guardarEtapa">
        <div class="grupo-input">
          <label>Nombre:</label>
          <input v-model="nuevaEtapa.nombre" placeholder="Ej: Bachillerato" required>
        </div>
        <div class="grupo-input">
          <label>Descripción:</label>
          <input v-model="nuevaEtapa.descripcion" placeholder="Ej: Bachillerato de Ciencias" required>
        </div>
        <button type="submit" class="btn-guardar">Guardar Etapa</button>
      </form>
    </div>

    <div class="grid-listado">
      <div v-for="etapa in etapas" :key="etapa.id" class="tarjeta">
        <span class="badge-id">ID: {{ etapa.id || 'S/N' }}</span>
        <h4>{{ etapa.nombre }}</h4>
        <p>{{ etapa.descripcion }}</p>
        <button @click="eliminarEtapa(etapa.id)" class="btn-eliminar">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      etapas: [],
      mostrarForm: false,
      url: 'http://100.52.46.68:3000/etapas',
      nuevaEtapa: {
        nombre: '',
        descripcion: ''
      }
    }
  },
  mounted() {
    this.cargarEtapas();
  },
  methods: {
    async cargarEtapas() {
      try {
        const res = await fetch(this.url);
        this.etapas = await res.json();
      } catch (e) {
        console.error("Error al cargar etapas");
      }
    },
    async guardarEtapa() {
      const idsNumericos = this.etapas
        .map(e => parseInt(e.id))
        .filter(id => !isNaN(id));
      
      const nextId = idsNumericos.length > 0 ? Math.max(...idsNumericos) + 1 : 1;

      const objetoEnvio = {
        id: nextId.toString(),
        nombre: this.nuevaEtapa.nombre,
        descripcion: this.nuevaEtapa.descripcion
      };

      try {
        const res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(objetoEnvio)
        });

        if (res.ok) {
          this.mostrarForm = false;
          this.nuevaEtapa = { nombre: '', descripcion: '' };
          this.cargarEtapas();
        }
      } catch (e) {
        console.error("Error al guardar");
      }
    },
    async eliminarEtapa(id) {
      if (confirm("¿Eliminar etapa?")) {
        try {
          await fetch(`${this.url}/${id}`, { method: 'DELETE' });
          this.cargarEtapas();
        } catch (e) {
          console.error("Error al eliminar");
        }
      }
    }
  }
}
</script>

<style scoped>
.gestion-container { padding: 20px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.formulario-card { background: #fff; padding: 20px; border: 1px solid #ccc; margin-bottom: 25px; max-width: 400px; }
.grupo-input { margin-bottom: 12px; }
.grupo-input label { display: block; font-weight: bold; font-size: 0.85rem; margin-bottom: 4px; }
.grupo-input input { width: 100%; padding: 8px; border: 1px solid #ccc; box-sizing: border-box; }
.btn-principal { background: #000; color: #fff; border: none; padding: 10px 15px; cursor: pointer; }
.btn-guardar { width: 100%; background: #000; color: white; border: none; padding: 10px; cursor: pointer; }
.grid-listado { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 15px; }
.tarjeta { border: 1px solid #ddd; padding: 15px; background: #fff; position: relative; text-align: center; }
.badge-id { background: #eee; color: #666; padding: 2px 6px; font-size: 0.7rem; border-radius: 4px; position: absolute; top: 10px; left: 10px; }
.btn-eliminar { background: none; border: 1px solid #ff4444; color: #ff4444; padding: 4px; width: 100%; cursor: pointer; margin-top: 15px; }
.btn-eliminar:hover { background: #ff4444; color: white; }
</style>