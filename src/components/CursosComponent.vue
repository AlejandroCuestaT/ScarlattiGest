<template>
  <div class="cursosAdmin">
    <div class="headerSection">
      <h2>Gestión de Cursos y Asignaturas</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btnNuevo">
        {{ mostrarFormulario ? 'Cancelar' : 'Nuevo Curso' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="cardFormulario">
      <div class="gridForm">
        <div class="campo">
          <label>ID Curso (Máx 10)</label>
          <input v-model="nuevoCurso.id" maxlength="10" placeholder="Ej: 55555">
        </div>
        <div class="campo">
          <label>Nombre del Curso</label>
          <input v-model="nuevoCurso.nombre_curso" placeholder="Ej: Desarrollo">
        </div>
        <div class="campo">
          <label>Etapa (ID)</label>
          <input v-model="nuevoCurso.etapa_id" maxlength="10" placeholder="Ej: CYBER">
        </div>
        <div class="campo">
          <label>Grupo</label>
          <input v-model="nuevoCurso.grupo">
        </div>
        <div class="campo">
          <label>Turno ID</label>
          <input v-model="nuevoCurso.turno_id" placeholder="Ej: 1m">
        </div>
        <div class="campo">
          <label>Aula ID</label>
          <input v-model="nuevoCurso.aula_id" placeholder="Ej: 11111">
        </div>
        <div class="campo">
          <label>DNI Tutor</label>
          <input v-model="nuevoCurso.tutor_id" placeholder="50564597V">
        </div>
        <div class="campo">
          <label>Año Académico</label>
          <input v-model="nuevoCurso.anio_academico" placeholder="Ej: 2026">
        </div>
        <button @click="guardarCurso" class="btnGuardar fullWidth">Guardar en Base de Datos</button>
      </div>
    </div>

    <table class="tablaGestion">
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
        <tr v-for="curso in cursos" :key="curso.id">
          <td><span class="idBadge">{{ curso.id }}</span></td>
          <td>{{ curso.nombre_curso }}</td>
          <td>{{ curso.etapa_id }}</td>
          <td>{{ curso.aula_id }}</td>
          <td>
            <button @click="eliminarCurso(curso.id)" class="btnEliminar">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'CursosComponent',
  data() {
    return {
      // Almacén de cursos que vienen de la API
      cursos: [],
      // Control de visibilidad del formulario
      mostrarFormulario: false,
      // Configuración de red invariable
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Modelo de datos para un nuevo curso (valores por defecto según requisitos)
      nuevoCurso: { 
        id: '', 
        nombre_curso: '', 
        etapa_id: '', 
        grupo: 'A', 
        turno_id: '', 
        aula_id: '', 
        tutor_id: '', 
        anio_academico: '2026' 
      }
    }
  },
  mounted() { 
    // Al cargar el componente, solicitamos los datos al servidor
    this.cargarCursos(); 
  },
  methods: {
    // Obtiene la lista completa de cursos filtrada por nuestro zusuario
    async cargarCursos() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/cursos?zusuario=${this.zusuario}`);
        this.cursos = await respuesta.json();
      } catch (error) {
        console.error("Error de carga:", error);
      }
    },
    // Envía el nuevo curso a la API usando el método POST
    async guardarCurso() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/cursos?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          // Usamos el Spread Operator para incluir los datos de auditoría
          body: JSON.stringify({
            ...this.nuevoCurso,
            zfecha: new Date().toISOString(),
            zusuario: this.zusuario
          })
        });

        if (respuesta.ok) {
          alert("Curso registrado con éxito");
          this.mostrarFormulario = false; // Cerramos el formulario
          this.cargarCursos(); // Refrescamos la tabla para mostrar el nuevo registro
        } else {
          const errorServidor = await respuesta.json();
          alert("Error de validación: " + errorServidor.error);
        }
      } catch (error) { 
        alert("Error de conexión con el servidor"); 
      }
    },
    // Elimina un curso permanentemente tras confirmación del usuario
    async eliminarCurso(id) {
      if (!confirm(`¿Estás seguro de eliminar el curso ${id}?`)) return;
      
      try {
        const respuesta = await fetch(`${this.apiUrl}/cursos/${id}?zusuario=${this.zusuario}`, { 
          method: 'DELETE' 
        });
        
        if (respuesta.ok) {
          this.cargarCursos(); // Actualizamos la vista
        } else {
          alert("No se ha podido eliminar el curso. Verifique si tiene dependencias.");
        }
      } catch (error) {
        console.error("Error al eliminar:", error);
      }
    }
  }
}
</script>

<style scoped>
.cursosAdmin { padding: 20px; }
.headerSection { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.btnNuevo { background: #6366f1; color: white; border: none; padding: 10px 15px; border-radius: 8px; cursor: pointer; font-weight: 500; }

.cardFormulario { background: white; padding: 20px; border-radius: 10px; border: 1px solid #e5e7eb; margin-bottom: 20px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.gridForm { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; align-items: end; }

.campo label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 5px; color: #4b5563; text-transform: uppercase; }
.campo input { width: 100%; padding: 10px; border: 1px solid #d1d5db; border-radius: 6px; box-sizing: border-box; }

.btnGuardar { background: #10b981; color: white; border: none; padding: 12px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.fullWidth { grid-column: 1 / -1; margin-top: 10px; }

.tablaGestion { width: 100%; border-collapse: collapse; background: white; border-radius: 8px; overflow: hidden; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
.tablaGestion th { background: #f9fafb; padding: 15px; text-align: left; border-bottom: 2px solid #edf2f7; color: #374151; }
.tablaGestion td { padding: 15px; border-bottom: 1px solid #edf2f7; }

.idBadge { background: #f3f4f6; padding: 4px 8px; border-radius: 4px; font-family: 'Courier New', monospace; font-weight: bold; color: #1f2937; }
.btnEliminar { background: #fee2e2; color: #ef4444; border: none; padding: 8px 12px; border-radius: 6px; cursor: pointer; font-size: 13px; }
.btnEliminar:hover { background: #fecaca; }
</style>