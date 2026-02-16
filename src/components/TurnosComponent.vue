<template>
  <div>
    <div class="titulo-seccion">
      <h2>Gestión de Turnos</h2>
      <button @click="abrirForm = !abrirForm">
        {{ abrirForm ? 'Cerrar' : 'Nuevo Turno' }}
      </button>
    </div>

    <div v-if="mensaje" :class="['alerta', claseMensaje]">{{ mensaje }}</div>

    <div v-if="abrirForm" class="contenedor-form">
      <h3>Registrar Nuevo Turno</h3>
      <form @submit.prevent="guardar">
        <input v-model="form.nombre" placeholder="Nombre (Ej: Mañana, Tarde, Nocturno)" required>
        <button type="submit" class="btn-guardar">Guardar Turno</button>
      </form>
    </div>

    <div class="lista">
      <div v-for="t in turnos" :key="t.id" class="ficha">
        <div class="info">
          <span class="badge">ID: {{ t.id }}</span>
          <strong>{{ t.nombre }}</strong>
        </div>
        <button @click="borrar(t.id)" class="btn-borrar">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      api: 'http://100.52.46.68:3000/turnos',
      turnos: [], abrirForm: false, mensaje: '', claseMensaje: '',
      form: { id: '', nombre: '' }
    }
  },
  mounted() { this.obtener(); },
  methods: {
    async obtener() {
      try {
        let res = await fetch(this.api);
        this.turnos = await res.json();
      } catch (e) { this.notificar('Error al cargar turnos', 'error'); }
    },
    async guardar() {
      const maxId = this.turnos.length > 0 ? Math.max(...this.turnos.map(t => Number(t.id))) : 0;
      this.form.id = (maxId + 1).toString();

      try {
        let res = await fetch(this.api, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.form)
        });
        if (res.ok) {
          this.notificar('Turno creado con ID ' + this.form.id, 'exito');
          this.abrirForm = false;
          this.form = { id: '', nombre: '' };
          this.obtener();
        }
      } catch (e) { this.notificar('Error de red', 'error'); }
    },
    async borrar(id) {
      if (confirm('¿Eliminar este turno?')) {
        await fetch(`${this.api}/${id}`, { method: 'DELETE' });
        this.obtener();
      }
    },
    notificar(m, c) { this.mensaje = m; this.claseMensaje = c; setTimeout(() => this.mensaje = '', 3000); }
  }
}
</script>

<style scoped>
.titulo-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.contenedor-form { background: #eee; padding: 15px; border: 1px solid #ccc; margin-bottom: 20px; }
input { display: block; width: 100%; padding: 8px; margin-bottom: 10px; border: 1px solid #ccc; box-sizing: border-box; }
.btn-guardar { width: 100%; padding: 10px; background-color: #000; color: white; border: none; cursor: pointer; }
.lista { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 10px; }
.ficha { background: #fff; padding: 15px; border: 1px solid #ddd; display: flex; flex-direction: column; align-items: center; gap: 10px; }
.badge { font-size: 0.7rem; color: #999; }
.btn-borrar { background: none; border: 1px solid #ddd; color: #999; padding: 4px 8px; cursor: pointer; font-size: 0.8rem; width: 100%; }
.btn-borrar:hover { background: #f0f0f0; color: #000; border-color: #000; }
.alerta { padding: 10px; margin-bottom: 10px; border: 1px solid #ccc; text-align: center; font-size: 0.9rem; }
.exito { background: #f9f9f9; color: #333; }
.error { background: #333; color: #fff; }
</style>