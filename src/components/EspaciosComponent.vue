<template>
  <div class="data-container">
    <div class="section-header">
      <h2>Gestión de Espacios (Aulas)</h2>
      <div class="quick-add">
        <input v-model="nuevo.id" placeholder="Código (ej: A01)">
        <input v-model="nuevo.nombre" placeholder="Nombre (ej: Aula de Música)">
        <button @click="crear" class="btn-add">Añadir Espacio</button>
      </div>
    </div>

    <table class="data-table">
      <thead>
        <tr>
          <th>Código</th>
          <th>Descripción del Espacio</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="espacio in lista" :key="espacio.id">
          <td>{{ espacio.id }}</td>
          <td>{{ espacio.nombre }}</td>
          <td>
            <button @click="eliminar(espacio.id)" class="btn-del">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      lista: [],
      nuevo: { id: '', nombre: '' },
      url: 'http://100.27.173.196:3000/espacios',
      zusuario: 'acuesta'
    }
  },
  mounted() {
    this.cargar();
  },
  methods: {
    async cargar() {
      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        this.lista = await res.json();
      } catch (e) {
        console.error("Error al cargar la lista de espacios");
      }
    },
    async crear() {
      if (!this.nuevo.id || !this.nuevo.nombre) return alert("Completa los datos del aula");

      await fetch(`${this.url}?zusuario=${this.zusuario}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(this.nuevo)
      });

      this.cargar();
      this.nuevo = { id: '', nombre: '' };
    },
    async eliminar(id) {
      if (confirm(`¿Deseas eliminar el espacio ${id}?`)) {
        await fetch(`${this.url}/${id}?zusuario=${this.zusuario}`, { method: 'DELETE' });
        this.cargar();
      }
    }
  }
}
</script>

<style scoped>
/* Estilos coherentes con los otros componentes */
.data-container { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }
.quick-add { display: flex; gap: 10px; }
.quick-add input { padding: 10px; border: 1px solid #ddd; border-radius: 6px; }
.btn-add { background: #34a853; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.data-table { width: 100%; border-collapse: collapse; }
.data-table th { text-align: left; padding: 15px; background: #f8f9fa; border-bottom: 2px solid #eee; }
.data-table td { padding: 15px; border-bottom: 1px solid #f0f0f0; }
.btn-del { background: #fff1f0; color: #ff4d4f; border: 1px solid #ffccc7; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
</style>