<template>
  <div class="cursos-admin">
    <div class="header-section">
      <h2>📚 Gestión de Cursos y Asignaturas</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-nuevo">
        {{ mostrarForm ? '✖ Cancelar' : '➕ Nuevo Curso' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="card-formulario">
      <div class="grid-form">
        <div class="campo">
          <label>ID Curso (Máx 10)</label>
          <input v-model="nuevoCurso.id" maxlength="10">
        </div>
        <div class="campo">
          <label>Nombre del Curso</label>
          <input v-model="nuevoCurso.nombre_curso">
        </div>
        <div class="campo">
          <label>Etapa (ID)</label>
          <input v-model="nuevoCurso.etapa_id" maxlength="10">
        </div>
        <div class="campo">
          <label>Grupo</label>
          <input v-model="nuevoCurso.grupo">
        </div>
        <div class="campo">
          <label>Turno ID</label>
          <input v-model="nuevoCurso.turno_id">
        </div>
        <div class="campo">
          <label>Aula ID</label>
          <input v-model="nuevoCurso.aula_id">
        </div>
        <div class="campo">
          <label>DNI Tutor</label>
          <input v-model="nuevoCurso.tutor_id">
        </div>
        <div class="campo">
          <label>Año Académico</label>
          <input v-model="nuevoCurso.anio_academico" placeholder="Ej: 2026">
        </div>
        <button @click="guardarCurso" class="btn-guardar full-width">Guardar en Base de Datos</button>
      </div>
    </div>

    <table class="tabla-gestion">
      <thead>
        <tr>
          <th>ID</th>
          <th>Asignatura</th>
          <th>Etapa</th>
          <th>Aula</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="c in cursos" :key="c.id">
          <td><span class="id-badge">{{ c.id }}</span></td>
          <td>{{ c.nombre_curso }}</td>
          <td>{{ c.etapa_id }}</td>
          <td>{{ c.aula_id }}</td>
          <td>
            <button @click="eliminarCurso(c.id)" class="btn-eliminar">🗑</button>
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
      cursos: [],
      mostrarForm: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      nuevoCurso: { 
        id: '', 
        nombre_curso: '', 
        etapa_id: '', 
        grupo: 'A', 
        turno_id: '', 
        aula_id: '', 
        tutor_id: '', 
        anio_academico: '2026' // Sincronizado con la API
      }
    }
  },
  mounted() { this.cargarCursos(); },
  methods: {
    async cargarCursos() {
      const res = await fetch(`${this.apiUrl}/cursos?zusuario=${this.zusuario}`);
      this.cursos = await res.json();
    },
    async guardarCurso() {
      try {
        const res = await fetch(`${this.apiUrl}/cursos?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            ...this.nuevoCurso,
            zfecha: new Date().toISOString(),
            zusuario: this.zusuario
          })
        });

        if (res.ok) {
          this.mostrarForm = false;
          this.cargarCursos();
        } else {
          const err = await res.json();
          alert("Error: " + err.error); // Aquí verás si falta algún otro campo
        }
      } catch (e) { alert("Error de conexión"); }
    },
    async eliminarCurso(id) {
      if (!confirm("¿Eliminar curso?")) return;
      await fetch(`${this.apiUrl}/cursos/${id}?zusuario=${this.zusuario}`, { method: 'DELETE' });
      this.cargarCursos();
    }
  }
}
</script>

<style scoped>
.cursos-admin { padding: 20px; }
.header-section { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.btn-nuevo { background: #6366f1; color: white; border: none; padding: 10px 15px; border-radius: 8px; cursor: pointer; }
.card-formulario { background: white; padding: 20px; border-radius: 10px; border: 1px solid #e5e7eb; margin-bottom: 20px; }
.grid-form { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; align-items: end; }
.campo label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 5px; color: #4b5563; }
.campo input { width: 100%; padding: 8px; border: 1px solid #d1d5db; border-radius: 6px; }
.btn-guardar { background: #10b981; color: white; border: none; padding: 10px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.tabla-gestion { width: 100%; border-collapse: collapse; background: white; border-radius: 8px; overflow: hidden; }
.tabla-gestion th { background: #f9fafb; padding: 12px; text-align: left; border-bottom: 2px solid #edf2f7; }
.tabla-gestion td { padding: 12px; border-bottom: 1px solid #edf2f7; }
.id-badge { background: #f3f4f6; padding: 2px 6px; border-radius: 4px; font-family: monospace; font-weight: bold; }
.btn-eliminar { background: #fee2e2; color: #ef4444; border: none; padding: 5px 10px; border-radius: 4px; cursor: pointer; }
</style>