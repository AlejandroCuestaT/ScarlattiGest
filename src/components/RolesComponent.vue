<template>
  <div class="roles-panel">
    <div class="cabecera">
      <h2>🛡️ Gestión de Roles</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btn-main">
        {{ mostrarFormulario ? '✖️ Cancelar' : '➕ Nuevo Rol' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="formulario-card">
      <h3>Registrar Perfil de Acceso</h3>
      <form @submit.prevent="guardar">
        <div class="grid-inputs">
          <input v-model="form.id" placeholder="ID técnico (Ej: Admin_a, Prof_a)" required>
          <input v-model="form.nombre" placeholder="Nombre público (Ej: Administrador)" required>
          <input v-model.number="form.nivel_privilegio" type="number" min="1" max="10" placeholder="Nivel (1-10)" required>
          <input v-model="form.descripcion" placeholder="Descripción breve" required>
        </div>
        <button type="submit" class="btn-save">💾 Guardar Rol</button>
      </form>
    </div>

    <div class="lista-grid">
      <div v-for="rol in roles" :key="rol.id" class="tarjeta-rol">
        <div class="rol-header">
          <span class="badge-nivel">Nivel {{ rol.nivel_privilegio }}</span>
          <small class="id-tag">#{{ rol.id }}</small>
        </div>
        <h4>{{ rol.nombre }}</h4>
        <p>{{ rol.descripcion }}</p>
        <div class="acciones">
          <button @click="eliminar(rol.id)" class="btn-del">🗑️ Eliminar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'RolesComponent',
  data() {
    return {
      // IP corregida para conectar con la base de datos real
      url: 'http://44.207.19.239:3000/roles',
      zusuario: 'acuesta',
      roles: [],                
      mostrarFormulario: false,
      form: { id: '', nombre: '', nivel_privilegio: 1, descripcion: '' }
    }
  },
  mounted() {
    this.cargar();
  },
  methods: {
    async cargar() {
      try {
        // Añadimos el parámetro de auditoría obligatorio
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        if (!res.ok) throw new Error("Error en servidor");
        this.roles = await res.json();
      } catch (e) {
        console.error('Error al conectar con la API de roles:', e);
      }
    },
    async guardar() {
      // Validación básica para evitar registros corruptos
      if (!this.form.id.trim()) return alert("El ID del rol es obligatorio");

      const payload = { 
        ...this.form, 
        zusuario: this.zusuario,
        zfecha: new Date().toISOString()
      };

      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          this.mostrarFormulario = false;
          this.form = { id: '', nombre: '', nivel_privilegio: 1, descripcion: '' };
          this.cargar();
        } else {
          alert("Error: Verifica que el ID no esté duplicado.");
        }
      } catch (e) {
        console.error('Fallo de red al guardar:', e);
      }
    },
    async eliminar(id) {
      if (!confirm('¿Eliminar este rol? Podría afectar a los usuarios vinculados.')) return;

      // Construcción de URL dinámica: /roles/ID?zusuario=USUARIO
      const urlDelete = `${this.url}/${id}?zusuario=${this.zusuario}`;

      try {
        const res = await fetch(urlDelete, { method: 'DELETE' });
        if (res.ok) {
          this.cargar();
        } else {
          // Captura el 404 mostrado en consola
          alert("No se pudo eliminar. El rol no existe o está protegido.");
        }
      } catch (e) {
        console.error('Error al eliminar:', e);
      }
    }
  }
}
</script>

<style scoped>
.roles-panel { padding: 25px; background: #f8fafc; min-height: 100vh; }
.cabecera { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }
.btn-main { background: #6366f1; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: bold; }

.formulario-card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); margin-bottom: 30px; border: 1px solid #e2e8f0; }
.grid-inputs { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 15px; }
.grid-inputs input { padding: 10px; border: 1px solid #cbd5e1; border-radius: 6px; }
.btn-save { background: #10b981; color: white; border: none; padding: 12px; width: 100%; border-radius: 6px; font-weight: bold; cursor: pointer; }

.lista-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); gap: 20px; }
.tarjeta-rol { background: white; border-radius: 10px; padding: 20px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); border-top: 4px solid #6366f1; }
.rol-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.badge-nivel { background: #e0e7ff; color: #4338ca; font-size: 11px; font-weight: bold; padding: 2px 8px; border-radius: 12px; }
.id-tag { color: #94a3b8; font-family: monospace; }
.tarjeta-rol h4 { margin: 10px 0; color: #1e293b; }
.tarjeta-rol p { font-size: 13px; color: #64748b; margin-bottom: 15px; }
.acciones { border-top: 1px solid #f1f5f9; padding-top: 12px; text-align: right; }
.btn-del { background: none; border: none; color: #ef4444; cursor: pointer; font-size: 12px; font-weight: bold; }
</style>