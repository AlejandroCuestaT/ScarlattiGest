<template>
  <div>
    <div class="cabecera">
      <h2>Turnos</h2>
      <button @click="mostrarFormulario = !mostrarFormulario">
        {{ mostrarFormulario ? 'Cerrar' : 'Nuevo Turno' }}
      </button>
    </div>

    <div v-if="mensaje" :class="['aviso', tipoMensaje]">{{ mensaje }}</div>

    <div v-if="mostrarFormulario" class="formulario">
      <h3>Añadir turno</h3>
      <form @submit.prevent="guardar">
        <input v-model="form.nombre" placeholder="Nombre (Ej: Mañana, Tarde)" required>
        <button type="submit">Guardar</button>
      </form>
    </div>

    <div class="lista">
      <div v-for="turno in turnos" :key="turno.id" class="tarjeta">
        <small>ID: {{ turno.id }}</small>
        <h4>{{ turno.nombre }}</h4>
        <button @click="eliminar(turno.id)" style="color: red;">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: 'http://100.27.173.196:3000/turnos',
      zusuario: 'acuesta',
      turnos: [],               
      mostrarFormulario: false,
      mensaje: '',
      tipoMensaje: '',
      form: { nombre: '' }
    }
  },

  mounted() {
    this.cargar()
  },

  methods: {
    async cargar() {
      try {
        let res = await fetch(`${this.url}?zusuario=${this.zusuario}`)
        this.turnos = await res.json()
      } catch (e) {
        this.mostrarMensaje('Error al cargar turnos', 'error')
      }
    },

    async guardar() {
      let ids = this.turnos.map(t => parseInt(t.id))
      let siguienteId = ids.length > 0 ? Math.max(...ids) + 1 : 1

      let payload = { id: siguienteId.toString(), nombre: this.form.nombre, zusuario: this.zusuario }

      try {
        let res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        })
        if (res.ok) {
          this.mostrarMensaje('Turno guardado', 'exito')
          this.mostrarFormulario = false
          this.form = { nombre: '' }
          this.cargar()
        }
      } catch (e) {
        this.mostrarMensaje('Error al guardar', 'error')
      }
    },

    async eliminar(id) {
      if (!confirm('¿Eliminar este turno?')) return
      await fetch(`${this.url}/${id}?zusuario=${this.zusuario}`, { method: 'DELETE' })
      this.cargar()
    },

    mostrarMensaje(texto, tipo) {
      this.mensaje = texto
      this.tipoMensaje = tipo
      setTimeout(() => this.mensaje = '', 3000)
    }
  }
}
</script>

<style scoped>
.cabecera { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.formulario { background: #f9f9f9; padding: 15px; border: 1px solid #ccc; margin-bottom: 20px; max-width: 350px; }
input { display: block; width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; border: 1px solid #ccc; }
button { padding: 8px 12px; cursor: pointer; margin-top: 5px; }
.lista { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 15px; margin-top: 15px; }
.tarjeta { background: #fff; border: 1px solid #ddd; padding: 15px; text-align: center; }
.tarjeta small { color: #999; font-size: 12px; }
.tarjeta h4 { margin: 8px 0 10px; }
.aviso { padding: 10px; margin-bottom: 10px; border: 1px solid #ccc; }
.exito { background: #d4edda; color: #155724; }
.error { background: #f8d7da; color: #721c24; }
</style>