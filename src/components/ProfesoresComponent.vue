<template>
  <div class="panel-profesores">
    <div class="header-seccion">
      <h2>Panel de Instructores (Staff Dev)</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btn-nuevo">
        {{ mostrarFormulario ? '✖ Cancelar' : '➕ Alta Profesor' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="card-formulario">
      <h3>Registrar Nuevo Instructor</h3>
      <div class="grid-form">
        <input v-model="nuevo.dni_nie" placeholder="DNI/NIE (Ej: 56437845T)" maxlength="10">
        <input v-model="nuevo.nombre" placeholder="Nombre">
        <input v-model="nuevo.apellidos" placeholder="Apellidos">
        <input v-model="nuevo.correo_institucional" type="email" placeholder="ej@prueba.com">
        
        <select v-model="nuevo.departamento_id">
          <option value="10">Dpto. Software (10)</option>
          <option value="IA">Dpto. IA</option>
          <option value="SISTEMAS">Dpto. Sistemas</option>
        </select>

        <select v-model="nuevo.rol_id">
          <option value="Prof_a">Profesor</option>
          <option value="Admin_a">Administrador</option>
        </select>

        <button @click="crearProfesor" class="btn-guardar">Guardar en Plantilla</button>
      </div>
    </div>

    <table class="tabla-profesores">
      <thead>
        <tr>
          <th>DNI/NIE</th>
          <th>Nombre Completo</th>
          <th>Departamento</th>
          <th>Email</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="p in profesores" :key="p.dni_nie">
          <td><span class="badge-dni">{{ p.dni_nie }}</span></td>
          <td><b>{{ p.nombre }} {{ p.apellidos }}</b></td>
          <td><span class="tag-dpto">{{ p.departamento_id }}</span></td>
          <td>{{ p.correo_institucional }}</td>
          <td>
            <button @click="eliminarProfesor(p.dni_nie)" class="btn-eliminar">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'ProfesoresComponent',
  data() {
    return {
      profesores: [],
      mostrarFormulario: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta", // Tu usuario según el sistema
      nuevo: {
        dni_nie: '',
        nombre: '',
        apellidos: '',
        correo_institucional: '',
        departamento_id: '10', // Valor por defecto visto en tu tabla
        rol_id: 'Prof_a'
      }
    }
  },
  mounted() {
    this.cargarProfesores();
  },
  methods: {
    async cargarProfesores() {
      try {
        const res = await fetch(`${this.apiUrl}/profesores?zusuario=${this.zusuario}`);
        const data = await res.json();
        this.profesores = Array.isArray(data) ? data : [];
      } catch (e) {
        console.error("Error cargando profesores:", e);
      }
    },
    async crearProfesor() {
      // Validación básica antes de enviar
      if (!this.nuevo.dni_nie || !this.nuevo.nombre) {
        alert("El DNI y el Nombre son obligatorios");
        return;
      }

      const payload = {
        ...this.nuevo,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const res = await fetch(`${this.apiUrl}/profesores?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          alert("¡Profesor guardado con éxito!");
          this.resetForm();
          this.mostrarFormulario = false;
          this.cargarProfesores(); // Recarga la lista para ver el nuevo profesor
        } else {
          const errData = await res.json();
          alert("Error al guardar: " + (errData.error || "Error desconocido"));
        }
      } catch (err) {
        console.error("Error de red:", err);
        alert("No se pudo conectar con el servidor");
      }
    },
    async eliminarProfesor(dni) {
      if (!confirm(`¿Eliminar al profesor ${dni}?`)) return;
      try {
        await fetch(`${this.apiUrl}/profesores/${dni}?zusuario=${this.zusuario}`, { 
          method: 'DELETE' 
        });
        this.cargarProfesores();
      } catch (e) {
        console.error("Error al eliminar:", e);
      }
    },
    resetForm() {
      this.nuevo = { 
        dni_nie: '', nombre: '', apellidos: '', 
        correo_institucional: '', departamento_id: '10', rol_id: 'Prof_a' 
      };
    }
  }
}
</script>

<style scoped>
.panel-profesores { background: white; padding: 25px; border-radius: 12px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }

.card-formulario { background: #f1f5f9; padding: 20px; border-radius: 10px; border: 1px solid #e2e8f0; margin-bottom: 20px; }
.grid-form { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; }
.grid-form input, .grid-form select { padding: 10px; border: 1px solid #cbd5e1; border-radius: 6px; }

.btn-nuevo { background: #334155; color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.btn-guardar { background: #0f172a; color: white; border: none; padding: 12px; border-radius: 6px; cursor: pointer; grid-column: 1 / -1; font-weight: bold; margin-top: 10px; }

.tabla-profesores { width: 100%; border-collapse: collapse; }
.tabla-profesores th { text-align: left; padding: 12px; background: #f8fafc; color: #475569; border-bottom: 2px solid #e2e8f0; }
.tabla-profesores td { padding: 12px; border-bottom: 1px solid #f1f5f9; }

.badge-dni { font-family: monospace; background: #e2e8f0; padding: 4px 8px; border-radius: 4px; color: #1e293b; font-weight: bold; }
.tag-dpto { background: #ecfdf5; color: #065f46; padding: 4px 8px; border-radius: 4px; font-size: 11px; font-weight: bold; }
.btn-eliminar { background: #ef4444; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
</style>