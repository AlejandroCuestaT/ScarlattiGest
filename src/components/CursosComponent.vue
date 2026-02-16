<template>
  <div class="gestion-container">
    <div class="header-seccion">
      <h2>Gestión de Cursos Académicos</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-principal">
        {{ mostrarForm ? 'Cerrar' : 'Nuevo Curso' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="formulario-card">
      <h3>Datos del curso</h3>
      <form @submit.prevent="guardarCurso">
        <div class="fila-form">
          <div class="grupo-input">
            <label>Nombre del Curso:</label>
            <input v-model="nuevoCurso.nombre_curso" placeholder="Ej: 2ºDAW" required>
          </div>
          <div class="grupo-input">
            <label>Grupo:</label>
            <input v-model="nuevoCurso.grupo" placeholder="Ej: A" required>
          </div>
        </div>

        <div class="fila-form">
          <div class="grupo-input">
            <label>ID Etapa:</label>
            <input v-model="nuevoCurso.etapa_id" placeholder="ID de Etapa" required>
          </div>
          <div class="grupo-input">
            <label>ID Turno:</label>
            <input v-model="nuevoCurso.turno_id" placeholder="1 o 2" required>
          </div>
        </div>

        <div class="fila-form">
          <div class="grupo-input">
            <label>Año Académico:</label>
            <input v-model="nuevoCurso.anio_academico" placeholder="2025-2026" required>
          </div>
          <div class="grupo-input">
            <label>ID Aula:</label>
            <input v-model="nuevoCurso.aula_id" placeholder="ID Espacio" required>
          </div>
        </div>

        <div class="grupo-input">
          <label>ID Tutor (DNI/NIE):</label>
          <input v-model="nuevoCurso.tutor_id" placeholder="ID del Profesor">
        </div>

        <button type="submit" class="btn-guardar">Registrar Curso</button>
      </form>
    </div>

    <div class="tabla-contenedor">
      <table class="tabla-minimal">
        <thead>
          <tr>
            <th>ID</th>
            <th>Curso</th>
            <th>Etapa</th>
            <th>Grupo</th>
            <th>Año</th>
            <th>Aula</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="curso in cursos" :key="curso.id">
            <td><span class="badge">{{ curso.id }}</span></td>
            <td><strong>{{ curso.nombre_curso }}</strong></td>
            <td>{{ curso.etapa_id }}</td>
            <td>{{ curso.grupo }}</td>
            <td>{{ curso.anio_academico }}</td>
            <td>{{ curso.aula_id }}</td>
            <td>
              <button @click="eliminarCurso(curso.id)" class="btn-borrar-tabla">Eliminar</button>
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
      cursos: [],
      mostrarForm: false,
      url: 'http://100.52.46.68:3000/cursos',
      nuevoCurso: {
        nombre_curso: '',
        etapa_id: '',
        grupo: '',
        turno_id: '',
        anio_academico: '2025-2026',
        tutor_id: null,
        aula_id: ''
      }
    }
  },
  mounted() {
    this.cargarCursos();
  },
  methods: {
    async cargarCursos() {
      try {
        const res = await fetch(this.url);
        this.cursos = await res.json();
      } catch (e) { console.error("Error al cargar cursos"); }
    },
    async guardarCurso() {
      const ids = this.cursos.map(c => parseInt(c.id)).filter(id => !isNaN(id));
      const nextId = ids.length > 0 ? Math.max(...ids) + 1 : 1;

      const objetoEnvio = {
        id: nextId.toString(),
        nombre_curso: this.nuevoCurso.nombre_curso,
        etapa_id: this.nuevoCurso.etapa_id,
        grupo: this.nuevoCurso.grupo,
        turno_id: this.nuevoCurso.turno_id,
        anio_academico: this.nuevoCurso.anio_academico,
        tutor_id: this.nuevoCurso.tutor_id,
        aula_id: this.nuevoCurso.aula_id
      };

      try {
        const res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(objetoEnvio)
        });

        if (res.ok) {
          this.mostrarForm = false;
          this.resetForm();
          this.cargarCursos();
        }
      } catch (e) { console.error("Error en POST"); }
    },
    async eliminarCurso(id) {
      if (confirm("¿Eliminar curso?")) {
        await fetch(`${this.url}/${id}`, { method: 'DELETE' });
        this.cargarCursos();
      }
    },
    resetForm() {
      this.nuevoCurso = { 
        nombre_curso: '', etapa_id: '', grupo: '', 
        turno_id: '', anio_academico: '2025-2026', 
        tutor_id: null, aula_id: '' 
      };
    }
  }
}
</script>

<style scoped>
.gestion-container { padding: 20px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.formulario-card { background: #fff; padding: 25px; border: 1px solid #ddd; margin-bottom: 25px; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
.fila-form { display: flex; gap: 15px; margin-bottom: 15px; }
.grupo-input { flex: 1; display: flex; flex-direction: column; }
.grupo-input label { font-size: 0.8rem; font-weight: bold; margin-bottom: 5px; }
.grupo-input input { padding: 10px; border: 1px solid #ccc; border-radius: 4px; }
.btn-principal, .btn-guardar { background: #000; color: #fff; border: none; padding: 10px 20px; cursor: pointer; border-radius: 4px; }
.btn-guardar { width: 100%; margin-top: 10px; font-weight: bold; }

.tabla-contenedor { background: #fff; border: 1px solid #eee; border-radius: 8px; overflow: hidden; }
.tabla-minimal { width: 100%; border-collapse: collapse; text-align: left; }
.tabla-minimal th { background: #f8f8f8; padding: 15px; font-size: 0.8rem; text-transform: uppercase; color: #666; }
.tabla-minimal td { padding: 15px; border-bottom: 1px solid #eee; font-size: 0.9rem; }
.badge { background: #eee; padding: 2px 8px; border-radius: 10px; font-size: 0.75rem; color: #333; }
.btn-borrar-tabla { background: none; border: 1px solid #ff4444; color: #ff4444; padding: 4px 8px; cursor: pointer; border-radius: 4px; }
.btn-borrar-tabla:hover { background: #ff4444; color: #fff; }
</style>