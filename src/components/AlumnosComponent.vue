<template>
  <div class="panel-alumnos">
    <div class="header-seccion">
      <h2>Gestión de Alumnos</h2>
      <div class="formulario-directo">
        <input v-model="nuevo.dni" placeholder="DNI (ej: 12345678Z)" class="input-txt">
        <input v-model="nuevo.nombre" placeholder="Nombre" class="input-txt">
        <input v-model="nuevo.apellidos" placeholder="Apellidos" class="input-txt">
        <button @click="agregarAlumno" class="btn-guardar">Añadir Alumno</button>
      </div>
    </div>

    <table class="tabla-datos">
      <thead>
        <tr>
          <th>DNI</th>
          <th>Nombre</th>
          <th>Apellidos</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="alumno in lista" :key="alumno.dni">
          <td>{{ alumno.dni }}</td>
          <td>{{ alumno.nombre }}</td>
          <td>{{ alumno.apellidos }}</td>
          <td>
            <button @click="borrarAlumno(alumno.dni)" class="btn-eliminar">Eliminar</button>
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
      nuevo: {
        dni: '', // IMPORTANTE: nombre de campo corregido
        nombre: '',
        apellidos: ''
      },
      url: "http://44.207.19.239:3000/alumnos",
      zusuario: "acuesta"
    }
  },
  mounted() {
    this.obtenerAlumnos();
  },
  methods: {
    async obtenerAlumnos() {
      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        this.lista = await res.json();
      } catch (e) {
        console.error("Error al cargar alumnos");
      }
    },

    async agregarAlumno() {
      if (!this.nuevo.dni || !this.nuevo.nombre) {
        alert("El DNI y el Nombre son obligatorios");
        return;
      }

      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.nuevo)
        });

        const data = await res.json();

        if (res.ok) {
          alert("Alumno guardado");
          this.nuevo = { dni: '', nombre: '', apellidos: '' };
          this.obtenerAlumnos(); // Refrescar la tabla
        } else {
          alert("Error: " + (data.error || "No se pudo guardar"));
        }
      } catch (e) {
        alert("Error de conexión con la IP 44.207.19.239");
      }
    },

    async borrarAlumno(dniBorrar) {
      if (!confirm("¿Seguro que quieres eliminar este alumno?")) return;

      try {
        const res = await fetch(`${this.url}/${dniBorrar}?zusuario=${this.zusuario}`, {
          method: 'DELETE'
        });

        if (res.ok) {
          this.obtenerAlumnos();
        } else {
          alert("Error al eliminar");
        }
      } catch (e) {
        console.error("Error en el borrado");
      }
    }
  }
}
</script>

<style scoped>
.panel-alumnos { background: white; padding: 25px; border-radius: 10px; }
.header-seccion { margin-bottom: 25px; border-bottom: 1px solid #eee; padding-bottom: 20px; }
.formulario-directo { display: flex; gap: 10px; margin-top: 15px; }
.input-txt { padding: 10px; border: 1px solid #ccc; border-radius: 5px; flex: 1; }
.btn-guardar { background: #28a745; color: white; border: none; padding: 10px 20px; border-radius: 5px; cursor: pointer; font-weight: bold; }
.tabla-datos { width: 100%; border-collapse: collapse; }
.tabla-datos th, .tabla-datos td { text-align: left; padding: 12px; border-bottom: 1px solid #eee; }
.btn-eliminar { background: #ff4d4f; color: white; border: none; padding: 5px 10px; border-radius: 4px; cursor: pointer; }
</style>