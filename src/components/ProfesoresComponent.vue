<template>
  <div class="panel-gestion">
    <h2>Gestión de Profesores</h2>

    <div class="caja-formulario">
      <h4>Registrar Nuevo Profesor</h4>
      <div class="fila-input">
        <input v-model="nuevo.dni_nie" placeholder="DNI/NIE">
        <input v-model="nuevo.nombre" placeholder="Nombre">
        <input v-model="nuevo.apellidos" placeholder="Apellidos">
        <button @click="crear" class="btn-azul">Guardar</button>
      </div>
    </div>

    <div class="listado">
      <table class="tabla-datos">
        <thead>
          <tr>
            <th>DNI</th>
            <th>Nombre</th>
            <th>Apellidos</th>
            <th class="texto-centro">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="p in profesores" :key="p.dni_nie">
            <td>{{ p.dni_nie }}</td>
            <td>{{ p.nombre }}</td>
            <td>{{ p.apellidos }}</td>
            <td class="texto-centro">
              <button @click="eliminar(p.dni_nie)" class="btn-rojo">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      profesores: [],
      nuevo: { dni_nie: '', nombre: '', apellidos: '' },
      url: 'http://100.27.173.196:3000/profesores',
      zusuario: 'acuesta'
    }
  },
  mounted() { this.obtenerTodos(); },
  methods: {
    async obtenerTodos() {
      const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
      this.profesores = await res.json();
    },
    async crear() {
      if (!this.nuevo.dni_nie) return alert("El DNI es obligatorio");
      const res = await fetch(`${this.url}?zusuario=${this.zusuario}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(this.nuevo)
      });
      if (res.ok) {
        this.obtenerTodos();
        this.nuevo = { dni_nie: '', nombre: '', apellidos: '' };
      }
    },
    async eliminar(dni) {
      if (!confirm('¿Seguro que quieres borrar este registro?')) return;
      const res = await fetch(`${this.url}/${dni}?zusuario=${this.zusuario}`, {
        method: 'DELETE'
      });
      if (res.ok) this.obtenerTodos();
    }
  }
}
</script>

<style scoped>
.panel-gestion { background: white; padding: 25px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }
.caja-formulario { background: #f8f9fa; padding: 20px; border-radius: 6px; margin-bottom: 30px; }
.fila-input { display: flex; gap: 10px; margin-top: 10px; }
.fila-input input { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
.tabla-datos { width: 100%; border-collapse: collapse; }
.tabla-datos th, .tabla-datos td { padding: 12px; text-align: left; border-bottom: 1px solid #eee; }
.btn-azul { background: #1a73e8; color: white; border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; }
.btn-rojo { background: #fce8e6; color: #d93025; border: 1px solid #f5c2c7; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
.texto-centro { text-align: center; }
</style>