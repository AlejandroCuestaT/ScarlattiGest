<template>
  <div class="data-container">
    <div class="section-header">
      <h2>Gestión de Cursos</h2>
      <div class="quick-add">
        <input v-model="nuevo.id" placeholder="ID del Curso (ej: 1A)">
        <input v-model="nuevo.nombre" placeholder="Nombre del Curso (ej: 1º Primaria)">
        <button @click="crear" class="btn-add">Añadir Curso</button>
      </div>
    </div>

    <table class="data-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre del Curso</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="curso in lista" :key="curso.id">
          <td>{{ curso.id }}</td>
          <td>{{ curso.nombre }}</td>
          <td>
            <button @click="eliminar(curso.id)" class="btn-del">Borrar</button>
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
      // El objeto 'nuevo' debe tener los nombres exactos que pide la API
      nuevo: { id: '', nombre: '' }, 
      url: 'http://100.27.173.196:3000/cursos',
      zusuario: 'acuesta' // Tu firma obligatoria
    }
  },
  mounted() {
    // Al cargar el componente, pedimos los cursos al servidor
    this.cargar();
  },
  methods: {
    async cargar() {
      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        this.lista = await res.json();
      } catch (e) {
        console.error("Error cargando cursos");
      }
    },
    async crear() {
      // Validación básica antes de enviar
      if (!this.nuevo.id || !this.nuevo.nombre) return alert("Rellena todos los campos");

      await fetch(`${this.url}?zusuario=${this.zusuario}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(this.nuevo)
      });

      this.cargar(); // Refrescar la tabla
      this.nuevo = { id: '', nombre: '' }; // Limpiar formulario
    },
    async eliminar(id) {
      if (confirm('¿Seguro que quieres eliminar este curso?')) {
        await fetch(`${this.url}/${id}?zusuario=${this.zusuario}`, { 
          method: 'DELETE' 
        });
        this.cargar();
      }
    }
  }
}
</script>

<style scoped>
/* Estilos coherentes con el resto de la App para un acabado profesional */
.data-container { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }
.quick-add { display: flex; gap: 10px; }
.quick-add input { padding: 10px; border: 1px solid #ddd; border-radius: 6px; }
.btn-add { background: #34a853; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.data-table { width: 100%; border-collapse: collapse; }
.data-table th { text-align: left; padding: 15px; background: #f8f9fa; border-bottom: 2px solid #eee; }
.data-table td { padding: 15px; border-bottom: 1px solid #f0f0f0; }
.btn-del { background: #fff1f0; color: #ff4d4f; border: 1px solid #ffccc7; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
.btn-del:hover { background: #ff4d4f; color: white; }
</style>