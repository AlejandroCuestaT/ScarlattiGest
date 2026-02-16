<template>
  <div class="gestion-container">
    <div class="header-seccion">
      <h2>Control de Cuentas de Usuario</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-principal">
        {{ mostrarForm ? 'Cerrar' : 'Nuevo Usuario' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="formulario-card">
      <h3>Crear credenciales de acceso</h3>
      <form @submit.prevent="guardarUsuario">
        <div class="fila-form">
          <div class="grupo-input">
            <label>Nombre de Usuario (Login):</label>
            <input v-model="nuevoUsuario.login" placeholder="Ej: jgarcia22" required>
          </div>
          <div class="grupo-input">
            <label>Contraseña:</label>
            <input v-model="nuevoUsuario.password_hash" type="password" placeholder="****" required>
          </div>
        </div>

        <div class="fila-form">
          <div class="grupo-input">
            <label>Rol asignado (ID):</label>
            <input v-model="nuevoUsuario.rol_id" placeholder="1:Admin, 2:Prof, 3:Alum" required>
          </div>
          <div class="grupo-input">
            <label>Referencia Identidad (ID):</label>
            <input v-model="nuevoUsuario.ref_identidad_fk" placeholder="ID Alumno/Profesor" required>
          </div>
        </div>

        <button type="submit" class="btn-guardar">Crear Usuario</button>
      </form>
    </div>

    <div class="tabla-contenedor">
      <table class="tabla-minimal">
        <thead>
          <tr>
            <th>Login</th>
            <th>Rol</th>
            <th>ID Ref</th>
            <th>Estado</th>
            <th>Último Acceso</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="u in usuarios" :key="u.login">
            <td><strong>{{ u.login }}</strong></td>
            <td><span class="badge-rol">{{ tradRol(u.rol_id) }}</span></td>
            <td>{{ u.ref_identidad_fk }}</td>
            <td>{{ u.estado_id == '1' ? '🟢 Activo' : '🔴 Bloqueado' }}</td>
            <td class="fecha">{{ formatearFecha(u.ultimo_acceso) }}</td>
            <td>
              <button @click="eliminarUsuario(u.login)" class="btn-borrar-tabla">Eliminar</button>
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
      usuarios: [],
      mostrarForm: false,
      url: 'http://100.52.46.68:3000/usuarios',
      nuevoUsuario: {
        login: '',
        password_hash: '',
        rol_id: '',
        ref_identidad_fk: '',
        estado_id: '1'
      }
    }
  },
  mounted() { this.cargarUsuarios(); },
  methods: {
    async cargarUsuarios() {
      try {
        const res = await fetch(this.url);
        this.usuarios = await res.json();
      } catch (e) { console.error("Error al cargar usuarios"); }
    },
    async guardarUsuario() {
      const objetoEnvio = {
        login: this.nuevoUsuario.login,
        password_hash: this.nuevoUsuario.password_hash,
        rol_id: this.nuevoUsuario.rol_id.toString(),
        ref_identidad_fk: this.nuevoUsuario.ref_identidad_fk.toString(),
        estado_id: "1",
        ultimo_acceso: new Date().toISOString()
      };

      try {
        const res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(objetoEnvio)
        });
        if (res.ok) {
          this.mostrarForm = false;
          this.nuevoUsuario = { login: '', password_hash: '', rol_id: '', ref_identidad_fk: '', estado_id: '1' };
          this.cargarUsuarios();
        }
      } catch (e) { console.error("Error al crear usuario"); }
    },
    async eliminarUsuario(login) {
      if (confirm(`¿Eliminar al usuario ${login}?`)) {
        await fetch(`${this.url}/${login}`, { method: 'DELETE' });
        this.cargarUsuarios();
      }
    },
    tradRol(id) {
      const roles = { '1': 'Admin', '2': 'Profesor', '3': 'Alumno', '4': 'TIC' };
      return roles[id] || 'User';
    },
    formatearFecha(f) {
      if (!f) return 'Nunca';
      return new Date(f).toLocaleDateString();
    }
  }
}
</script>

<style scoped>
.gestion-container { padding: 20px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.formulario-card { background: #fff; padding: 20px; border: 1px solid #ddd; margin-bottom: 25px; max-width: 550px; }
.fila-form { display: flex; gap: 10px; margin-bottom: 10px; }
.grupo-input { flex: 1; display: flex; flex-direction: column; }
.grupo-input label { font-size: 0.75rem; font-weight: bold; margin-bottom: 3px; }
.grupo-input input { padding: 8px; border: 1px solid #ccc; border-radius: 3px; }
.btn-principal, .btn-guardar { background: #000; color: #fff; border: none; padding: 10px 15px; cursor: pointer; }
.btn-guardar { width: 100%; margin-top: 10px; }

.tabla-contenedor { background: #fff; border: 1px solid #eee; border-radius: 5px; }
.tabla-minimal { width: 100%; border-collapse: collapse; }
.tabla-minimal th { background: #f4f4f4; padding: 12px; text-align: left; font-size: 0.8rem; color: #666; }
.tabla-minimal td { padding: 12px; border-bottom: 1px solid #eee; font-size: 0.85rem; }
.badge-rol { background: #333; color: #fff; padding: 2px 6px; border-radius: 4px; font-size: 0.7rem; }
.fecha { color: #888; font-size: 0.75rem; }
.btn-borrar-tabla { background: none; border: 1px solid #ff4444; color: #ff4444; padding: 3px 7px; cursor: pointer; border-radius: 3px; }
</style>