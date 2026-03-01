<template>
  <div class="panel-usuarios">
    <div class="header-seccion">
      <h2>🔐 Gestión de Accesos y Usuarios</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btn-nuevo">
        {{ mostrarFormulario ? '✖ Cancelar' : '➕ Crear Usuario' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="card-formulario">
      <div class="grid-form">
        <input v-model="nuevo.login" placeholder="Nombre de usuario (Login)">
        <input v-model="nuevo.password_hash" type="password" placeholder="Contraseña">
        <input v-model="nuevo.ref_identidad_fk" placeholder="NIA del Alumno o DNI">

        <select v-model="nuevo.rol_id">
          <option value="" disabled>Seleccionar Rol</option>
          <option value="Admin_a">Administrador</option>
          <option value="Prof_a">Profesor</option>
          <option value="Alum_a">Alumno</option>
          <option value="Tic_a">Personal TIC</option> </select>

        <button @click="crearUsuario" class="btn-guardar" :disabled="cargando">
          {{ cargando ? 'Guardando...' : 'Vincular y Crear Acceso' }}
        </button>
      </div>
    </div>

    <table class="tabla-alumnos">
      <thead>
        <tr>
          <th>Login</th>
          <th>Rol</th>
          <th>ID Identidad</th>
          <th>Última Conexión</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="u in usuarios" :key="u.login">
          <td><strong>{{ u.login }}</strong></td>
          <td><span :class="['tag-rol', getRolClass(u.rol_id)]">{{ u.rol_id }}</span></td>
          <td><span class="badge-nia">{{ u.ref_identidad_fk }}</span></td>
          <td>{{ u.ultimo_acceso ? u.ultimo_acceso.split('T')[0] : 'Nunca' }}</td>
          <td>
            <button @click="eliminarUsuario(u.login)" class="btn-eliminar">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'UsuariosComponent',
  data() {
    return {
      usuarios: [],
      mostrarFormulario: false,
      cargando: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      nuevo: {
        login: '',
        password_hash: '',
        rol_id: '',
        ref_identidad_fk: '',
        estado_id: 'Act_a'
      }
    }
  },
  mounted() {
    this.cargarUsuarios();
  },
  methods: {
    async cargarUsuarios() {
      try {
        const res = await fetch(`${this.apiUrl}/usuarios?zusuario=${this.zusuario}`);
        this.usuarios = await res.json();
      } catch (e) {
        console.error("Error al cargar usuarios", e);
      }
    },
    async crearUsuario() {
      if (!this.nuevo.login || !this.nuevo.password_hash || !this.nuevo.rol_id) {
        return alert("Por favor, rellena todos los campos obligatorios.");
      }

      this.cargando = true;
      const payload = {
        ...this.nuevo,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const res = await fetch(`${this.apiUrl}/usuarios?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        const data = await res.json();

        if (res.ok) {
          alert("Usuario creado con éxito");
          this.resetForm();
          this.mostrarFormulario = false;
          await this.cargarUsuarios();
        } else {
          // El error de FK que veías saldrá aquí si el ID es incorrecto
          alert("Error del servidor: " + (data.error || "No se pudo crear el usuario"));
        }
      } catch (e) {
        alert("Error de conexión con la API");
      } finally {
        this.cargando = false;
      }
    },
    async eliminarUsuario(login) {
      if (!confirm(`¿Estás seguro de que deseas eliminar el acceso de ${login}?`)) return;
      try {
        const res = await fetch(`${this.apiUrl}/usuarios/${login}?zusuario=${this.zusuario}`, { 
          method: 'DELETE' 
        });
        if (res.ok) this.cargarUsuarios();
      } catch (e) {
        alert("Error al eliminar");
      }
    },
    resetForm() {
      this.nuevo = { login: '', password_hash: '', rol_id: '', ref_identidad_fk: '', estado_id: 'Act_a' };
    },
    getRolClass(rol) {
      if (rol === 'Admin_a') return 'rol-admin';
      if (rol === 'Tic_a') return 'rol-tic';
      if (rol === 'Prof_a') return 'rol-prof';
      return 'rol-alum';
    }
  }
}
</script>

<style scoped>
.panel-usuarios { background: #fff; padding: 25px; border-radius: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.btn-nuevo { background: #1e293b; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: bold; }
.card-formulario { background: #f8fafc; padding: 20px; border-radius: 10px; border: 1px solid #e2e8f0; margin-bottom: 25px; }
.grid-form { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; }
.grid-form input, .grid-form select { padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; outline-color: #3b82f6; }
.btn-guardar { background: #3b82f6; color: white; border: none; padding: 12px; border-radius: 8px; cursor: pointer; grid-column: 1 / -1; font-weight: bold; }
.btn-guardar:disabled { background: #94a3b8; cursor: not-allowed; }

.tabla-alumnos { width: 100%; border-collapse: collapse; margin-top: 10px; }
.tabla-alumnos th { text-align: left; padding: 12px; border-bottom: 2px solid #f1f5f9; color: #64748b; font-size: 14px; }
.tabla-alumnos td { padding: 15px 12px; border-bottom: 1px solid #f1f5f9; }

.badge-nia { font-family: monospace; background: #f1f5f9; padding: 5px 10px; border-radius: 6px; font-weight: bold; color: #475569; }

/* Estilos de etiquetas de rol */
.tag-rol { padding: 4px 12px; border-radius: 20px; font-size: 12px; font-weight: bold; }
.rol-admin { background: #fee2e2; color: #991b1b; }
.rol-tic { background: #f3e8ff; color: #6b21a8; }
.rol-prof { background: #dcfce7; color: #166534; }
.rol-alum { background: #dbeafe; color: #1e40af; }

.btn-eliminar { background: #fee2e2; color: #ef4444; border: none; padding: 8px 15px; border-radius: 6px; cursor: pointer; font-weight: bold; transition: 0.2s; }
.btn-eliminar:hover { background: #ef4444; color: white; }
</style>