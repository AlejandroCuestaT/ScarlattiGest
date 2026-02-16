<template>
  <div>
    <div class="titulo-seccion">
      <h2>Listado de Alumnos</h2>
      <button @click="abrirForm = !abrirForm">
        {{ abrirForm ? 'Cerrar formulario' : 'Nuevo Alumno' }}
      </button>
    </div>

    <div v-if="mensaje" :class="['alerta', claseMensaje]">{{ mensaje }}</div>

    <div v-if="abrirForm" class="contenedor-form">
      <h3>Datos del nuevo alumno</h3>
      <form @submit.prevent="enviar">
        <input v-model="form.nia" placeholder="NIA" required>
        <input v-model="form.nombre" placeholder="Nombre" required>
        <input v-model="form.apellidos" placeholder="Apellidos" required>
        <input v-model="form.curso_id" placeholder="ID de Curso" required>
        <input v-model="form.correo_electronico" placeholder="Email" type="email" required>
        <input v-model="form.tutor_legal_contacto" placeholder="ID Tutor" required>
        <select v-model="form.estado_id">
          <option value="1">Activo</option>
          <option value="2">Inactivo</option>
        </select>
        <button type="submit" class="btn-guardar">Guardar Registro</button>
      </form>
    </div>

    <button @click="obtenerDatos" class="btn-recargar">Actualizar lista</button>

    <div class="lista">
      <div v-for="item in lista" :key="item.nia" class="ficha" :class="{ 'alumno-inactivo': item.estado_id == '2' }">
        <strong>{{ item.nombre }} {{ item.apellidos }}</strong>
        <p>NIA: {{ item.nia }}</p>
        <p>Estado: 
          <span :class="item.estado_id == '1' ? 'txt-activo' : 'txt-inactivo'">
            {{ item.estado_id == '1' ? 'Activo' : 'Inactivo' }}
          </span>
        </p>

        <div class="acciones">
          <button @click="alternarEstado(item)" class="btn-estado">
            {{ item.estado_id == '1' ? 'Desactivar' : 'Activar' }}
          </button>
          
          <button @click="borrarAlumno(item.nia)" class="btn-borrar">Eliminar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      api: 'http://100.52.46.68:3000/alumnos',
      lista: [],
      abrirForm: false,
      mensaje: '',
      claseMensaje: '',
      form: { nia: '', nombre: '', apellidos: '', curso_id: '', correo_electronico: '', tutor_legal_contacto: '', estado_id: '1' }
    }
  },
  mounted() { this.obtenerDatos(); },
  methods: {
    async obtenerDatos() {
      try {
        let res = await fetch(this.api);
        this.lista = await res.json();
      } catch (e) { this.notificar('Error al cargar', 'error'); }
    },
    async enviar() {
      try {
        let res = await fetch(this.api, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.form)
        });
        if (res.ok) {
          this.notificar('Guardado', 'exito');
          this.abrirForm = false;
          this.form = { nia: '', nombre: '', apellidos: '', curso_id: '', correo_electronico: '', tutor_legal_contacto: '', estado_id: '1' };
          this.obtenerDatos();
        }
      } catch (e) { this.notificar('Error al guardar', 'error'); }
    },
    async alternarEstado(alumno) {
      const nuevoEstado = alumno.estado_id == '1' ? '2' : '1';
      
      const alumnoActualizado = { ...alumno, estado_id: nuevoEstado };

      try {
        let res = await fetch(`${this.api}/${alumno.nia}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(alumnoActualizado)
        });

        if (res.ok) {
          this.notificar('Estado actualizado', 'exito');
          this.obtenerDatos();
        } else {
          this.notificar('Error al cambiar estado', 'error');
        }
      } catch (e) {
        this.notificar('Error de conexión', 'error');
      }
    },
    async borrarAlumno(nia) {
      if (confirm('¿Seguro que quieres eliminar este alumno?')) {
        try {
          let res = await fetch(`${this.api}/${nia}`, { method: 'DELETE' });
          if (res.ok) {
            this.notificar('Alumno eliminado', 'exito');
            this.obtenerDatos();
          }
        } catch (e) { this.notificar('No se pudo eliminar', 'error'); }
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
input, select { display: block; width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; }

.btn-guardar { width: 100%; padding: 10px; background-color: #28a745; color: white; border: none; cursor: pointer; }
.btn-recargar { margin: 10px 0; padding: 5px 10px; cursor: pointer; }

.acciones { display: flex; gap: 10px; margin-top: 10px; }
.btn-borrar { background: #dc3545; color: white; border: none; padding: 5px 10px; cursor: pointer; }
.btn-estado { background: #6c757d; color: white; border: none; padding: 5px 10px; cursor: pointer; }

.lista { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.ficha { background: #fff; padding: 10px; border: 1px solid #ccc; border-left: 5px solid #28a745; }
.alumno-inactivo { border-left: 5px solid #dc3545; opacity: 0.8; }

.txt-activo { color: #28a745; font-weight: bold; }
.txt-inactivo { color: #dc3545; font-weight: bold; }

.alerta { padding: 10px; margin-bottom: 15px; border: 1px solid #ccc; }
.exito { background-color: #d4edda; color: #155724; }
.error { background-color: #f8d7da; color: #721c24; }
</style>