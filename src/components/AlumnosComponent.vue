<template>
  <div class="panel-alumnos">
    <div class="header-seccion">
      <h2>Registro de Estudiantes (Developers)</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btn-nuevo">
        {{ mostrarFormulario ? '✖ Cancelar' : '➕ Matricular Alumno' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="card-formulario">
      <h3>Nueva Matrícula Dev</h3>
      <div class="grid-form">
        <input v-model="nuevo.nia" placeholder="NIA (Ej: DEV008)" maxlength="10">
        <input v-model="nuevo.nombre" placeholder="Nombre">
        <input v-model="nuevo.apellidos" placeholder="Apellidos">
        
        <select v-model="nuevo.curso_id">
          <option value="" disabled>Seleccionar Curso</option>
          <option v-for="c in cursosDisponibles" :key="c.id" :value="c.id">
            {{ c.id }} - {{ c.nombre_curso }}
          </option>
        </select>

        <input v-model="nuevo.correo_electronico" type="email" placeholder="Email">
        <input v-model="nuevo.tutor_legal_contacto" placeholder="Contacto (Tutor/Org)">
        
        <button @click="crearAlumno" class="btn-guardar">Confirmar Matrícula</button>
      </div>
    </div>

    <table class="tabla-alumnos">
      <thead>
        <tr>
          <th>NIA</th>
          <th>Nombre Completo</th>
          <th>Curso ID</th>
          <th>Email</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="a in alumnos" :key="a.nia">
          <td><span class="badge-nia">{{ a.nia }}</span></td>
          <td>{{ a.nombre }} {{ a.apellidos }}</td>
          <td><span class="tag-curso">{{ a.curso_id }}</span></td>
          <td>{{ a.correo_electronico }}</td>
          <td>
            <button @click="eliminarAlumno(a.nia)" class="btn-eliminar">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'AlumnosComponent',
  data() {
    return {
      alumnos: [],
      cursosDisponibles: [], // Para cargar los cursos como 11111, 33333, etc.
      mostrarFormulario: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      nuevo: {
        nia: '',
        nombre: '',
        apellidos: '',
        curso_id: '',
        correo_electronico: '',
        tutor_legal_contacto: '',
        estado_id: 'Act_a'
      }
    }
  },
  mounted() {
    this.cargarAlumnos();
    this.cargarCursos(); // Cargamos cursos para el selector
  },
  methods: {
    async cargarAlumnos() {
      try {
        const res = await fetch(`${this.apiUrl}/alumnos?zusuario=${this.zusuario}`);
        const data = await res.json();
        this.alumnos = Array.isArray(data) ? data : [];
      } catch (e) {
        console.error("Error al obtener alumnos", e);
      }
    },
    async cargarCursos() {
      try {
        const res = await fetch(`${this.apiUrl}/cursos?zusuario=${this.zusuario}`);
        const data = await res.json();
        this.cursosDisponibles = Array.isArray(data) ? data : [];
      } catch (e) {
        console.error("Error al obtener cursos", e);
      }
    },
    async crearAlumno() {
      if (!this.nuevo.nia || !this.nuevo.curso_id) return alert("NIA y Curso son obligatorios");

      const payload = {
        ...this.nuevo,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const res = await fetch(`${this.apiUrl}/alumnos?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          alert(`Alumno ${this.nuevo.nombre} matriculado con éxito`);
          this.resetForm();
          this.mostrarFormulario = false;
          this.cargarAlumnos();
        } else {
          const err = await res.json();
          alert("Error de API: " + err.error);
        }
      } catch (err) {
        console.error("Error en la petición", err);
      }
    },
    async eliminarAlumno(nia) {
      if (!confirm(`¿Dar de baja al alumno ${nia}?`)) return;
      try {
        await fetch(`${this.apiUrl}/alumnos/${nia}?zusuario=${this.zusuario}`, { method: 'DELETE' });
        this.cargarAlumnos();
      } catch (e) {
        console.error("Error al eliminar", e);
      }
    },
    resetForm() {
      this.nuevo = { nia: '', nombre: '', apellidos: '', curso_id: '', correo_electronico: '', tutor_legal_contacto: '', estado_id: 'Act_a' };
    }
  }
}
</script>

<style scoped>
.panel-alumnos { background: #fff; padding: 25px; border-radius: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }

.card-formulario { background: #f8fafc; padding: 20px; border-radius: 10px; border: 1px solid #e2e8f0; margin-bottom: 25px; }
.grid-form { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; }
.grid-form input, .grid-form select { padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 14px; }

.btn-nuevo { background: #1e293b; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: 600; }
.btn-guardar { background: #3b82f6; color: white; border: none; padding: 12px; border-radius: 8px; cursor: pointer; grid-column: 1 / -1; font-weight: bold; margin-top: 10px; }

.tabla-alumnos { width: 100%; border-collapse: separate; border-spacing: 0 8px; }
.tabla-alumnos th { text-align: left; padding: 12px; color: #64748b; border-bottom: 2px solid #f1f5f9; }
.tabla-alumnos td { padding: 15px; background: #fff; border-bottom: 1px solid #f1f5f9; }

.badge-nia { font-family: 'Courier New', Courier, monospace; background: #f1f5f9; padding: 5px 10px; border-radius: 6px; font-weight: bold; color: #1e293b; }
.tag-curso { background: #dbeafe; color: #1e40af; padding: 4px 10px; border-radius: 20px; font-size: 12px; font-weight: bold; }
.btn-eliminar { background: #ef4444; color: white; border: none; padding: 8px 15px; border-radius: 6px; cursor: pointer; }
</style>