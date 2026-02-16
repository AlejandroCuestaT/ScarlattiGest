<template>
  <div>
    <div class="titulo-seccion">
      <h2>Listado de Profesores</h2>
      <button @click="abrirForm = !abrirForm">
        {{ abrirForm ? 'Cerrar formulario' : 'Nuevo Profesor' }}
      </button>
    </div>

    <div v-if="mensaje" :class="['alerta', claseMensaje]">{{ mensaje }}</div>

    <div v-if="abrirForm" class="contenedor-form">
      <h3>Datos del nuevo profesor</h3>
      <form @submit.prevent="enviar">
        <input v-model="form.dni_nie" placeholder="DNI o NIE" required>
        <input v-model="form.nombre" placeholder="Nombre" required>
        <input v-model="form.apellidos" placeholder="Apellidos" required>
        <input v-model="form.correo_institucional" placeholder="Email institucional" type="email" required>
        <input v-model="form.departamento_id" placeholder="ID Departamento" required>
        <input v-model="form.rol_id" placeholder="ID Rol" required>
        <input v-model="form.password_hash" placeholder="Contraseña" type="password" required>
        <button type="submit" class="btn-guardar">Guardar Profesor</button>
      </form>
    </div>

    <button @click="obtenerDatos" class="btn-recargar">Actualizar lista</button>

    <div class="lista">
      <div v-for="p in lista" :key="p.dni_nie" class="ficha">
        <strong>{{ p.nombre }} {{ p.apellidos }}</strong>
        <p>DNI: {{ p.dni_nie }}</p>
        <p>Email: {{ p.correo_institucional }}</p>
        <p>Dpto ID: {{ p.departamento_id }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      api: 'http://100.52.46.68:3000/profesores',
      lista: [],
      abrirForm: false,
      mensaje: '',
      claseMensaje: '',
      form: { dni_nie: '', nombre: '', apellidos: '', correo_institucional: '', departamento_id: '', rol_id: '', password_hash: '' }
    }
  },
  mounted() {
    this.obtenerDatos();
  },
  methods: {
    async obtenerDatos() {
      try {
        let res = await fetch(this.api);
        this.lista = await res.json();
      } catch (e) {
        this.notificar('Error al cargar profesores', 'error');
      }
    },
    async enviar() {
      try {
        let res = await fetch(this.api, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.form)
        });
        if (res.ok) {
          this.notificar('Profesor registrado correctamente', 'exito');
          this.abrirForm = false;
          this.form = { dni_nie: '', nombre: '', apellidos: '', correo_institucional: '', departamento_id: '', rol_id: '', password_hash: '' };
          this.obtenerDatos();
        }
      } catch (e) {
        this.notificar('Fallo al guardar profesor', 'error');
      }
    },
    notificar(texto, tipo) {
      this.mensaje = texto;
      this.claseMensaje = tipo;
      setTimeout(() => this.mensaje = '', 3000);
    }
  }
}
</script>

<style scoped>
.titulo-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.contenedor-form { background: #fff; padding: 15px; border: 1px solid #ccc; margin-bottom: 20px; }
input { display: block; width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; }
.btn-guardar { width: 100%; padding: 10px; background-color: #007bff; color: white; border: none; cursor: pointer; }
.btn-recargar { margin: 10px 0; padding: 5px 10px; cursor: pointer; }
.alerta { padding: 10px; margin-bottom: 15px; border: 1px solid #ccc; }
.exito { background-color: #d4edda; color: #155724; }
.error { background-color: #f8d7da; color: #721c24; }
.lista { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.ficha { background: #fff; padding: 10px; border: 1px solid #ccc; border-top: 3px solid #007bff; }
.ficha p { margin: 5px 0; font-size: 13px; }
</style>