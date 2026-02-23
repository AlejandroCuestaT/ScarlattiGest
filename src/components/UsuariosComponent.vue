<template>
  <div>
    <div class="cabecera">
      <h2>Usuarios</h2>
      <button @click="mostrarFormulario = !mostrarFormulario">
        {{ mostrarFormulario ? 'Cerrar' : 'Nuevo Usuario' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="formulario">
      <h3>Crear usuario</h3>
      <form @submit.prevent="guardar">
        <input v-model="form.login"            placeholder="Nombre de usuario" required>
        <input v-model="form.password_hash"    placeholder="Contraseña" type="password" required>
        <input v-model="form.rol_id"           placeholder="ID Rol (Admin, Prof, Alum, TIC)" required>
        <input v-model="form.ref_identidad_fk" placeholder="ID Referencia (DNI del profesor o NIA del alumno)" required>
        <select v-model="form.estado_id">
          <option value="Act">Activo</option>
          <option value="Baj">Baja</option>
          <option value="Bloq">Bloqueado</option>
        </select>
        <button type="submit">Crear Usuario</button>
      </form>
    </div>

    <table class="tabla">
      <thead>
        <tr>
          <th>Login</th>
          <th>Rol</th>
          <th>ID Referencia</th>
          <th>Estado</th>
          <th>Último acceso</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="usuario in usuarios" :key="usuario.login">
          <td>{{ usuario.login }}</td>
          <td>{{ nombreRol(usuario.rol_id) }}</td>
          <td>{{ usuario.ref_identidad_fk }}</td>
          <td>
             <span v-if="usuario.estado_id === 'Act'">🟢 Activo</span>
             <span v-else-if="usuario.estado_id === 'Bloq'">🔴 Bloqueado</span>
             <span v-else>⚪ {{ usuario.estado_id }}</span>
          </td>
          <td>{{ formatearFecha(usuario.ultimo_acceso) }}</td>
          <td>
            <button @click="eliminar(usuario.login)" style="color: red;">Eliminar</button>
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
      url: 'http://100.27.173.196:3000/usuarios',
      zusuario: 'acuesta',
      usuarios: [],             
      mostrarFormulario: false,
      form: {
        login: '', password_hash: '',
        rol_id: '', ref_identidad_fk: '',
        estado_id: 'Act'
      }
    }
  },

  mounted() {
    this.cargar()
  },

  methods: {
    async cargar() {
      try {
        let res = await fetch(`${this.url}?zusuario=${this.zusuario}`)
        this.usuarios = await res.json()
      } catch (e) {
        console.error('Error al cargar usuarios')
      }
    },

    async guardar() {
      let payload = {
        login: this.form.login,
        password_hash: this.form.password_hash,
        rol_id: this.form.rol_id,
        ref_identidad_fk: this.form.ref_identidad_fk,
        estado_id: this.form.estado_id,
        ultimo_acceso: new Date().toISOString(),
        zusuario: this.zusuario
      }

      try {
        let res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        })
        if (res.ok) {
          this.mostrarFormulario = false
          this.form = { login: '', password_hash: '', rol_id: '', ref_identidad_fk: '', estado_id: 'Act' }
          this.cargar()
        }
      } catch (e) {
        console.error('Error al crear usuario')
      }
    },

    async eliminar(login) {
      if (!confirm(`¿Eliminar al usuario ${login}?`)) return
      await fetch(`${this.url}/${login}?zusuario=${this.zusuario}`, { method: 'DELETE' })
      this.cargar()
    },

    nombreRol(id) {
      const roles = { 'Admin': 'Admin', 'Prof': 'Profesor', 'Alum': 'Alumno', 'TIC': 'TIC' }
      return roles[id] || id
    },

    formatearFecha(fecha) {
      if (!fecha) return 'Nunca'
      return new Date(fecha).toLocaleDateString()
    }
  }
}
</script>

<style scoped>
.cabecera { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.formulario { background: #f9f9f9; padding: 15px; border: 1px solid #ccc; margin-bottom: 20px; max-width: 500px; }
input, select { display: block; width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; border: 1px solid #ccc; }
button { padding: 8px 12px; cursor: pointer; margin-top: 5px; }
.tabla { width: 100%; border-collapse: collapse; margin-top: 15px; }
.tabla th { background: #f4f4f4; padding: 10px; text-align: left; border-bottom: 2px solid #ddd; font-size: 13px; }
.tabla td { padding: 10px; border-bottom: 1px solid #eee; font-size: 13px; }
</style>