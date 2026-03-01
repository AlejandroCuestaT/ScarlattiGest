<template>
  <div class="data-container">
    <div class="section-header">
      <h2>Gestión de Departamentos</h2>
      <div class="quick-add">
        <input v-model="nuevo.id" placeholder="ID (ej: INF)">
        <input v-model="nuevo.nombre" placeholder="Nombre (ej: Informática)">
        <button @click="crear" class="btn-add">Añadir Departamento</button>
      </div>
    </div>

    <table class="data-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre del Departamento</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="depto in lista" :key="depto.id">
          <td>{{ depto.id }}</td>
          <td>{{ depto.nombre }}</td>
          <td>
            <button @click="eliminar(depto.id)" class="btn-del">Eliminar</button>
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
      // CORRECCIÓN: IP unificada con el resto del proyecto
      url: 'http://44.207.19.239:3000/departamentos',
      zusuario: 'acuesta'
    }
  },
  mounted() {
    this.cargar();
  },
  methods: {
    async cargar() {
      try {
        // Añadimos zusuario para cumplir requisitos de auditoría de la API
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        if (!res.ok) throw new Error("Fallo al cargar");
        this.lista = await res.json();
      } catch (e) {
        console.error("Error al conectar con la API de departamentos:", e);
      }
    },
    async crear() {
      if (!this.nuevo.id || !this.nuevo.nombre) {
        return alert("Por favor, rellena el ID y el nombre");
      }

      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            ...this.nuevo,
            zusuario: this.zusuario, // Aseguramos auditoría en el body
            zfecha: new Date().toISOString()
          })
        });

        if (res.ok) {
          this.cargar();
          this.nuevo = { id: '', nombre: '' };
        }
      } catch (e) {
        alert("Error al crear el departamento");
      }
    },
    async eliminar(id) {
      if (confirm(`¿Seguro que quieres borrar el departamento ${id}?`)) {
        // Ajustamos la ruta para que sea /departamentos/ID?zusuario=...
        const urlEliminar = `${this.url}/${id}?zusuario=${this.zusuario}`;
        
        try {
          const res = await fetch(urlEliminar, { 
            method: 'DELETE' 
          });
          
          if (res.ok) {
            this.cargar();
          } else {
            alert("No se pudo eliminar el departamento. Verifique si tiene dependencias.");
          }
        } catch (e) {
          console.error("Error en la petición DELETE", e);
        }
      }
    }
  }
}
</script>

<style scoped>
/* Reutilizamos los estilos para que la interfaz sea coherente */
.data-container { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }
.quick-add { display: flex; gap: 10px; }
.quick-add input { padding: 10px; border: 1px solid #ddd; border-radius: 6px; outline: none; }
.quick-add input:focus { border-color: #007bff; }
.btn-add { background: #34a853; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.data-table { width: 100%; border-collapse: collapse; }
.data-table th { text-align: left; padding: 15px; background: #f8f9fa; border-bottom: 2px solid #eee; }
.data-table td { padding: 15px; border-bottom: 1px solid #f0f0f0; }
.btn-del { background: #fff1f0; color: #ff4d4f; border: 1px solid #ffccc7; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
.btn-del:hover { background: #ff4d4f; color: white; }
</style>