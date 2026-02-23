<template>
  <div class="panel-gestion">
    <div class="cabecera">
      <h2>Registro de Alumnos</h2>
      <div class="formulario-alta">
        <input v-model="nuevoAlumno.dni" placeholder="DNI (Ej: 12345678Z)" class="input-form">
        <input v-model="nuevoAlumno.nombre" placeholder="Nombre" class="input-form">
        <input v-model="nuevoAlumno.apellidos" placeholder="Apellidos" class="input-form">
        <button @click="ejecutarAlta" class="btn-add">Añadir Alumno</button>
      </div>
    </div>

    <table class="tabla-estilo">
      <thead>
        <tr>
          <th>DNI</th>
          <th>Nombre</th>
          <th>Apellidos</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="alumno in listaAlumnos" :key="alumno.dni">
          <td>{{ alumno.dni }}</td>
          <td>{{ alumno.nombre }}</td>
          <td>{{ alumno.apellidos }}</td>
          <td>
            <button @click="eliminarAlumno(alumno.dni)" class="btn-borrar">Eliminar</button>
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
      listaAlumnos: [],
      // ACTUALIZADO: Cambiado dni_nie por dni
      nuevoAlumno: { dni: '', nombre: '', apellidos: '' },
      url: 'http://100.27.173.196:3000/alumnos',
      miFirma: 'acuesta'
    }
  },
  mounted() {
    this.cargarAlumnos();
  },
  methods: {
    async cargarAlumnos() {
      const res = await fetch(`${this.url}?zusuario=${this.miFirma}`);
      this.listaAlumnos = await res.json();
    },

    async ejecutarAlta() {
      if (!this.nuevoAlumno.dni) return alert("El DNI es obligatorio");

      // Enviamos el objeto con el nombre de columna 'dni'
      const alumnoParaEnviar = {
        dni: this.nuevoAlumno.dni.trim(),
        nombre: this.nuevoAlumno.nombre.trim(),
        apellidos: this.nuevoAlumno.apellidos.trim()
      };

      try {
        const res = await fetch(`${this.url}?zusuario=${this.miFirma}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(alumnoParaEnviar)
        });

        const data = await res.json();

        if (res.ok) {
          alert("¡Alumno guardado con éxito!");
          this.nuevoAlumno = { dni: '', nombre: '', apellidos: '' };
          this.cargarAlumnos();
        } else {
          // Esto nos mostrará si falta algún otro campo
          alert("Error: " + (data.error || "Datos incorrectos"));
        }
      } catch (e) {
        alert("Error de red");
      }
    },

    async eliminarAlumno(dniValue) {
      if (!confirm("¿Borrar?")) return;
      await fetch(`${this.url}/${dniValue}?zusuario=${this.miFirma}`, { method: 'DELETE' });
      this.cargarAlumnos();
    }
  }
}
</script>

<style scoped>
.panel-gestion { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
.formulario-alta { display: flex; gap: 10px; margin-bottom: 20px; }
.input-form { padding: 10px; border: 1px solid #ddd; border-radius: 6px; flex: 1; }
.btn-add { background: #34a853; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.tabla-estilo { width: 100%; border-collapse: collapse; }
.tabla-estilo th, .tabla-estilo td { padding: 12px; border-bottom: 1px solid #eee; text-align: left; }
.btn-borrar { color: #ff4d4f; background: none; border: 1px solid #ffccc7; padding: 4px 10px; border-radius: 4px; cursor: pointer; }
</style>