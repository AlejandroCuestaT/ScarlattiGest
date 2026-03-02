<template>
  <div class="panelProfesores">
    <div class="headerSeccion">
      <h2>Gestión de Claustro</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btnNuevo">
        {{ mostrarFormulario ? 'Cancelar Registro' : 'Alta Nuevo Profesor' }}
      </button>
    </div>

    <transition name="fade">
      <div v-if="mostrarFormulario" class="cardFormulario">
        <h3>Registrar Nuevo Instructor</h3>
        <div class="gridForm">
          <div class="campoInput">
            <label>Identificación (DNI/NIE)</label>
            <input 
              v-model="formProfesor.dni_nie" 
              placeholder="Ej: 56437845T" 
              maxlength="10"
              @input="formProfesor.dni_nie = formProfesor.dni_nie.toUpperCase().trim()"
            >
          </div>
          <div class="campoInput">
            <label>Nombre</label>
            <input v-model="formProfesor.nombre" placeholder="Nombre">
          </div>
          <div class="campoInput">
            <label>Apellidos</label>
            <input v-model="formProfesor.apellidos" placeholder="Apellido">
          </div>
          <div class="campoInput">
            <label>Correo Institucional</label>
            <input v-model="formProfesor.correo_institucional" type="email" placeholder="usuario@scarlatti.es">
          </div>
          
          <div class="campoInput">
            <label>Departamento</label>
            <select v-model="formProfesor.departamento_id">
              <option value="10">Dpto. Software (10)</option>
              <option value="IA">Inteligencia Artificial</option>
              <option value="SISTEMAS">Sistemas y Redes</option>
            </select>
          </div>

          <div class="campoInput">
            <label>Rol de Sistema</label>
            <select v-model="formProfesor.rol_id">
              <option value="Prof_a">Profesor</option>
            </select>
          </div>

          <button @click="guardarProfesor" class="btnGuardar">Sincronizar con Plantilla</button>
        </div>
      </div>
    </transition>

    <div class="tablaContainer">
      <table class="tablaMaestra">
        <thead>
          <tr>
            <th>Documento</th>
            <th>Nombre Completo</th>
            <th>Departamento</th>
            <th>Contacto</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="profesor in listaProfesores" :key="profesor.dni_nie">
            <td><span class="badgeDni">{{ profesor.dni_nie }}</span></td>
            <td><strong>{{ profesor.nombre }} {{ profesor.apellidos }}</strong></td>
            <td><span class="tagDpto">{{ profesor.departamento_id }}</span></td>
            <td class="txtEmail">{{ profesor.correo_institucional }}</td>
            <td>
              <button @click="eliminarRegistro(profesor.dni_nie)" class="btnEliminar">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="listaProfesores.length === 0" class="noData">
      No hay personal docente registrado en la base de datos actual.
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProfesoresComponent',
  data() {
    return {
      // Estado de datos y UI
      listaProfesores: [],
      mostrarFormulario: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Modelo reactivo para el formulario
      formProfesor: {
        dni_nie: '',
        nombre: '',
        apellidos: '',
        correo_institucional: '',
        departamento_id: '10',
        rol_id: 'Prof_a'
      }
    }
  },
  mounted() {
    this.obtenerProfesores();
  },
  methods: {
    // Recupera la colección completa de docentes
    async obtenerProfesores() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/profesores?zusuario=${this.zusuario}`);
        const datos = await respuesta.json();
        this.listaProfesores = Array.isArray(datos) ? datos : [];
      } catch (error) {
        console.error("Fallo al obtener la plantilla docente:", error);
      }
    },

    // Procesa el alta de un nuevo profesor con auditoría
    async guardarProfesor() {
      if (!this.formProfesor.dni_nie || !this.formProfesor.nombre) {
        return alert("Los campos 'DNI' y 'Nombre' son requeridos para el alta.");
      }

      const payload = {
        ...this.formProfesor,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}/profesores?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          alert("Profesor registrado correctamente.");
          this.limpiarFormulario();
          this.mostrarFormulario = false;
          this.obtenerProfesores();
        } else {
          const errorApi = await respuesta.json();
          alert("Error de registro: " + (errorApi.error || "Datos duplicados o inválidos"));
        }
      } catch (err) {
        alert("Error crítico de comunicación con el servidor.");
      }
    },

    // Elimina un docente tras validación de seguridad
    async eliminarRegistro(dni) {
      if (!confirm(`¿Confirmas la baja permanente del profesor con identificación ${dni}?`)) return;
      
      try {
        const urlDelete = `${this.apiUrl}/profesores/${dni}?zusuario=${this.zusuario}`;
        const respuesta = await fetch(urlDelete, { method: 'DELETE' });
        
        if (respuesta.ok) {
          this.obtenerProfesores();
        }
      } catch (error) {
        console.error("Error al procesar la baja:", error);
      }
    },

    limpiarFormulario() {
      this.formProfesor = { 
        dni_nie: '', nombre: '', apellidos: '', 
        correo_institucional: '', departamento_id: '10', rol_id: 'Prof_a' 
      };
    }
  }
}
</script>

<style scoped>
.panelProfesores { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.02); }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; border-bottom: 2px solid #f8fafc; padding-bottom: 15px; }

.btnNuevo { background: #1e293b; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: 600; transition: background 0.2s; }
.btnNuevo:hover { background: #334155; }

.cardFormulario { background: #f8fafc; padding: 25px; border-radius: 12px; border: 1px solid #e2e8f0; margin-bottom: 30px; }
.gridForm { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 15px; }

.campoInput label { display: block; font-size: 12px; font-weight: 700; color: #64748b; margin-bottom: 6px; text-transform: uppercase; }
.gridForm input, .gridForm select { width: 100%; padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; box-sizing: border-box; background: white; }

.btnGuardar { background: #2563eb; color: white; border: none; padding: 14px; border-radius: 8px; cursor: pointer; grid-column: 1 / -1; font-weight: bold; margin-top: 10px; transition: opacity 0.2s; }
.btnGuardar:hover { opacity: 0.9; }

.tablaContainer { overflow-x: auto; }
.tablaMaestra { width: 100%; border-collapse: collapse; }
.tablaMaestra th { text-align: left; padding: 15px; background: #f1f5f9; color: #475569; font-size: 13px; text-transform: uppercase; border-bottom: 2px solid #e2e8f0; }
.tablaMaestra td { padding: 15px; border-bottom: 1px solid #f1f5f9; font-size: 14px; }

.badgeDni { font-family: 'Courier New', monospace; background: #e2e8f0; padding: 4px 10px; border-radius: 6px; color: #1e293b; font-weight: bold; }
.tagDpto { background: #dcfce7; color: #166534; padding: 5px 12px; border-radius: 20px; font-size: 11px; font-weight: 800; }
.txtEmail { color: #64748b; }

.btnEliminar { background: none; border: 1px solid #fee2e2; color: #ef4444; padding: 6px 14px; border-radius: 6px; cursor: pointer; font-weight: 600; transition: all 0.2s; }
.btnEliminar:hover { background: #ef4444; color: white; border-color: #ef4444; }

.noData { text-align: center; padding: 40px; color: #94a3b8; font-style: italic; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter, .fade-leave-to { opacity: 0; }
</style>