<template>
  <div class="gestion-container">
    <div class="header-seccion">
      <h2>Gestión de Roles y Privilegios</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-principal">
        {{ mostrarForm ? 'Cerrar' : 'Nuevo Rol' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="formulario-card">
      <h3>Definir nuevo rol</h3>
      <form @submit.prevent="guardarRol">
        <div class="grupo-input">
          <label>Nombre del Rol:</label>
          <input v-model="nuevoRol.nombre" placeholder="Ej: Coordinador" required>
        </div>
        <div class="grupo-input">
          <label>Nivel de Privilegio (1-3):</label>
          <input v-model.number="nuevoRol.nivel_privilegio" type="number" min="1" max="3" required>
        </div>
        <div class="grupo-input">
          <label>Descripción de funciones:</label>
          <input v-model="nuevoRol.descripcion" placeholder="Ej: Gestión de biblioteca" required>
        </div>
        <button type="submit" class="btn-guardar">Guardar Rol</button>
      </form>
    </div>

    <div class="grid-listado">
      <div v-for="rol in roles" :key="rol.id" class="tarjeta">
        <span class="badge-id">ID: {{ rol.id }}</span>
        <h4>{{ rol.nombre }}</h4>
        <p class="privilegio">Nivel: <strong>{{ rol.nivel_privilegio }}</strong></p>
        <p class="desc">{{ rol.descripcion }}</p>
        <button @click="eliminarRol(rol.id)" class="btn-eliminar">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roles: [],
      mostrarForm: false,
      url: 'http://100.52.46.68:3000/roles',
      nuevoRol: {
        nombre: '',
        nivel_privilegio: 1,
        descripcion: ''
      }
    }
  },
  mounted() {
    this.cargarRoles();
  },
  methods: {
    async cargarRoles() {
      try {
        const res = await fetch(this.url);
        this.roles = await res.json();
      } catch (e) {
        console.error("Error al obtener roles");
      }
    },
    async guardarRol() {
      const maxId = this.roles.length > 0 
        ? Math.max(...this.roles.map(r => parseInt(r.id))) 
        : 0;

      const objetoEnvio = {
        id: (maxId + 1).toString(),
        nombre: this.nuevoRol.nombre,
        nivel_privilegio: this.nuevoRol.nivel_privilegio,
        descripcion: this.nuevoRol.descripcion
      };

      try {
        const res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(objetoEnvio)
        });

        if (res.ok) {
          this.mostrarForm = false;
          this.nuevoRol = { nombre: '', nivel_privilegio: 1, descripcion: '' };
          this.cargarRoles();
        }
      } catch (e) {
        console.error("Error al guardar rol");
      }
    },
    async eliminarRol(id) {
      if (confirm("¿Eliminar este rol? Esto puede afectar a los usuarios asociados.")) {
        try {
          await fetch(`${this.url}/${id}`, { method: 'DELETE' });
          this.cargarRoles();
        } catch (e) {
          console.error("Error al borrar");
        }
      }
    }
  }
}
</script>

<style scoped>
.gestion-container { padding: 20px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.formulario-card { background: #fff; padding: 20px; border: 1px solid #ccc; margin-bottom: 25px; max-width: 450px; }
.grupo-input { margin-bottom: 15px; }
.grupo-input label { display: block; font-weight: bold; font-size: 0.85rem; margin-bottom: 5px; }
.grupo-input input { width: 100%; padding: 8px; border: 1px solid #ddd; box-sizing: border-box; }
.btn-principal { background: #000; color: #fff; border: none; padding: 10px 20px; cursor: pointer; }
.btn-guardar { width: 100%; background: #000; color: #fff; border: none; padding: 12px; cursor: pointer; font-weight: bold; }
.grid-listado { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 20px; }
.tarjeta { border: 1px solid #eee; padding: 20px; background: #fff; position: relative; border-radius: 4px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
.badge-id { background: #007bff; color: #fff; padding: 2px 8px; font-size: 0.7rem; border-radius: 10px; position: absolute; top: 10px; left: 10px; }
.tarjeta h4 { margin-top: 15px; border-bottom: 1px solid #eee; padding-bottom: 8px; }
.privilegio { font-size: 0.85rem; color: #444; }
.desc { font-size: 0.8rem; color: #777; font-style: italic; height: 40px; }
.btn-eliminar { background: #ff4d4d; color: #fff; border: none; padding: 8px; width: 100%; cursor: pointer; margin-top: 10px; }
</style>