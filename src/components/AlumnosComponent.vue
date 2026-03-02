<template>
  <div class="panelAlumnos">
    
    <div class="headerSeccion">
      <h2>Registro de Estudiantes</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btnNuevo">
        {{ mostrarFormulario ? 'Cancelar' : 'Matricular Alumno' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="cardFormulario">
      <h3>Nueva Matrícula</h3>
      <div class="gridForm">
        <input v-model="nuevo.nia" placeholder="NIA (Ej: DEV008)" maxlength="10">
        <input v-model="nuevo.nombre" placeholder="Nombre">
        <input v-model="nuevo.apellidos" placeholder="Apellidos">
        
        <select v-model="nuevo.curso_id">
          <option value="" disabled>Seleccionar Curso</option>
          <option v-for="curso in cursosDisponibles" :key="curso.id" :value="curso.id">
            {{ curso.id }} - {{ curso.nombre_curso }}
          </option>
        </select>

        <input v-model="nuevo.correo_electronico" type="email" placeholder="Email">
        <input v-model="nuevo.tutor_legal_contacto" placeholder="Contacto (Tutor/Org)">
        
        <button @click="crearAlumno" class="btnGuardar">Confirmar Matrícula</button>
      </div>
    </div>

    <table class="tablaAlumnos">
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
        <tr v-for="alumno in alumnos" :key="alumno.nia">
          <td><span class="badgeNia">{{ alumno.nia }}</span></td>
          <td>{{ alumno.nombre }} {{ alumno.apellidos }}</td>
          <td><span class="tagCurso">{{ alumno.curso_id }}</span></td>
          <td>{{ alumno.correo_electronico }}</td>
          <td>
            <button @click="eliminarAlumno(alumno.nia)" class="btnEliminar">Eliminar</button>
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
      // Listas de datos cargadas desde la API
      alumnos: [],
      cursosDisponibles: [], 
      // Estado de la interfaz
      mostrarFormulario: false,
      // Conexión obligatoria con el servidor
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Objeto base para el formulario de alta
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
    // Inicialización: cargamos los datos necesarios nada más montar la vista
    this.cargarAlumnos();
    this.cargarCursos();
  },
  methods: {
    // Obtiene la lista de alumnos desde el endpoint
    async cargarAlumnos() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/alumnos?zusuario=${this.zusuario}`);
        const datos = await respuesta.json();
        this.alumnos = Array.isArray(datos) ? datos : [];
      } catch (e) {
        console.error("Error al obtener alumnos", e);
      }
    },
    // Obtiene la lista de cursos disponibles para el selector
    async cargarCursos() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/cursos?zusuario=${this.zusuario}`);
        const datos = await respuesta.json();
        this.cursosDisponibles = Array.isArray(datos) ? datos : [];
      } catch (e) {
        console.error("Error al obtener cursos", e);
      }
    },
    // Función para crear un alumno con los datos del formulario
    async crearAlumno() {
      // Validación básica en frontend
      if (!this.nuevo.nia || !this.nuevo.curso_id) return alert("NIA y Curso son obligatorios");

      //Con esto nos ahorramos el escribir todas las propiedades como id: this.nuevo.id
      const payload = {
        ...this.nuevo, // Copiamos todos los datos del formulario
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}/alumnos?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          alert(`Alumno ${this.nuevo.nombre} matriculado con éxito`);
          this.resetFormulario();
          this.mostrarFormulario = false;
          this.cargarAlumnos(); // Refrescamos la tabla
        } else {
          const errorServidor = await respuesta.json();
          alert("Error de API: " + errorServidor.error);
        }
      } catch (err) {
        console.error("Error en la petición", err);
      }
    },
    // Función para dar de baja un alumno usando su NIA
    async eliminarAlumno(nia) {
      if (!confirm(`¿Dar de baja al alumno con NIA: ${nia}?`)) return;
      try {
        await fetch(`${this.apiUrl}/alumnos/${nia}?zusuario=${this.zusuario}`, { method: 'DELETE' });
        this.cargarAlumnos(); // Refrescamos la tabla tras el borrado
      } catch (e) {
        console.error("Error al eliminar", e);
      }
    },
    // Limpia el formulario para dejarlo listo para un nuevo uso
    resetFormulario() {
      this.nuevo = { 
        nia: '', 
        nombre: '', 
        apellidos: '', 
        curso_id: '', 
        correo_electronico: '', 
        tutor_legal_contacto: '', 
        estado_id: 'Act_a' 
      };
    }
  }
}
</script>

<style scoped>
.panelAlumnos { background: #fff; padding: 25px; border-radius: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }

.cardFormulario { background: #f8fafc; padding: 20px; border-radius: 10px; border: 1px solid #e2e8f0; margin-bottom: 25px; }
.gridForm { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; }
.gridForm input, .gridForm select { padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 14px; }

.btnNuevo { background: #1e293b; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: 600; }
.btnGuardar { background: #3b82f6; color: white; border: none; padding: 12px; border-radius: 8px; cursor: pointer; grid-column: 1 / -1; font-weight: bold; margin-top: 10px; }

.tablaAlumnos { width: 100%; border-collapse: separate; border-spacing: 0 8px; }
.tablaAlumnos th { text-align: left; padding: 12px; color: #64748b; border-bottom: 2px solid #f1f5f9; }
.tablaAlumnos td { padding: 15px; background: #fff; border-bottom: 1px solid #f1f5f9; }

.badgeNia { font-family: 'Courier New', Courier, monospace; background: #f1f5f9; padding: 5px 10px; border-radius: 6px; font-weight: bold; color: #1e293b; }
.tagCurso { background: #dbeafe; color: #1e40af; padding: 4px 10px; border-radius: 20px; font-size: 12px; font-weight: bold; }
.btnEliminar { background: #ef4444; color: white; border: none; padding: 8px 15px; border-radius: 6px; cursor: pointer; }
</style>