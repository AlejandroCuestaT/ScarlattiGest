<template>
  <div>
    <div class="cabecera">
      <h2>Roles</h2>
      <button @click="mostrarFormulario = !mostrarFormulario">
        {{ mostrarFormulario ? 'Cerrar' : 'Nuevo Rol' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="formulario">
      <h3>Añadir rol</h3>
      <form @submit.prevent="guardar">
        <input v-model="form.id"          placeholder="ID del Rol (Ej: Admin, Prof)" required>
        <input v-model="form.nombre"      placeholder="Nombre (Ej: Coordinador)" required>
        <input v-model.number="form.nivel_privilegio" type="number" min="1" max="10" placeholder="Nivel de privilegio (1-10)" required>
        <input v-model="form.descripcion"     placeholder="Descripción del rol" required>
        <button type="submit">Guardar</button>
      </form>
    </div>

    <div class="lista">
      <div v-for="rol in roles" :key="rol.id" class="tarjeta">
        <small>ID: {{ rol.id }}</small>
        <h4>{{ rol.nombre }}</h4>
        <p>Nivel: {{ rol.nivel_privilegio }}</p>
        <p>{{ rol.descripcion }}</p>
        <button @click="eliminar(rol.id)" style="color: red;">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: 'http://100.27.173.196:3000/roles',
      zusuario: 'acuesta',
      roles: [],                
      mostrarFormulario: false,
      form: { id: '', nombre: '', nivel_privilegio: 1, descripcion: '' }
    }
  },

  mounted() {
    this.cargar()
  },

  methods: {
    async cargar() {
      try {
        let res = await fetch(`${this.url}?zusuario=${this.zusuario}`)
        this.roles = await res.json()
      } catch (e) {
        console.error('Error al cargar roles')
      }
    },

    async guardar() {
      // Como los roles ahora son strings (Admin, Prof, etc), lo cogemos del form
      let payload = { ...this.form, zusuario: this.zusuario }

      try {
        let res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        })
        if (res.ok) {
          this.mostrarFormulario = false
          this.form = { id: '', nombre: '', nivel_privilegio: 1, descripcion: '' }
          this.cargar()
        }
      } catch (e) {
        console.error('Error al guardar')
      }
    },

    async eliminar(id) {
      if (!confirm('¿Eliminar este rol? Puede afectar a usuarios asociados.')) return
      await fetch(`${this.url}/${id}?zusuario=${this.zusuario}`, { method: 'DELETE' })
      this.cargar()
    }
  }
}
</script>

<style scoped>
.cabecera { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.formulario { background: #f9f9f9; padding: 15px; border: 1px solid #ccc; margin-bottom: 20px; max-width: 450px; }
input { display: block; width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; border: 1px solid #ccc; }
button { padding: 8px 12px; cursor: pointer; margin-top: 5px; }
.lista { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 15px; margin-top: 15px; }
.tarjeta { background: #fff; border: 1px solid #ddd; padding: 15px; }
.tarjeta small { color: #999; font-size: 12px; }
.tarjeta h4 { margin: 8px 0 5px; }
.tarjeta p { margin: 3px 0; font-size: 13px; color: #555; }
</style>