<template>
  <div>
    <div class="cabecera">
      <h2>Etapas Educativas</h2>
      <button @click="mostrarFormulario = !mostrarFormulario">
        {{ mostrarFormulario ? 'Cerrar' : 'Nueva Etapa' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="formulario">
      <h3>Añadir etapa</h3>
      <form @submit.prevent="guardar">
        <input v-model="form.nombre"      placeholder="Nombre (Ej: Bachillerato)" required>
        <input v-model="form.descripcion" placeholder="Descripción" required>
        <button type="submit">Guardar</button>
      </form>
    </div>

    <div class="lista">
      <div v-for="etapa in etapas" :key="etapa.id" class="tarjeta">
        <small>ID: {{ etapa.id }}</small>
        <h4>{{ etapa.nombre }}</h4>
        <p>{{ etapa.descripcion }}</p>
        <button @click="eliminar(etapa.id)" style="color: red;">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: 'http://100.27.173.196:3000/etapas',
      zusuario: 'acuesta',
      etapas: [],              
      mostrarFormulario: false,
      form: { nombre: '', descripcion: '' }
    }
  },

  mounted() {
    this.cargar()
  },

  methods: {
    async cargar() {
      try {
        let res = await fetch(`${this.url}?zusuario=${this.zusuario}`)
        this.etapas = await res.json()
      } catch (e) {
        console.error('Error al cargar etapas')
      }
    },

    async guardar() {
      let ids = this.etapas.map(e => parseInt(e.id)).filter(id => !isNaN(id))
      let siguienteId = ids.length > 0 ? Math.max(...ids) + 1 : 1

      let payload = { ...this.form, id: siguienteId.toString(), zusuario: this.zusuario }

      try {
        let res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        })
        if (res.ok) {
          this.mostrarFormulario = false
          this.form = { nombre: '', descripcion: '' }
          this.cargar()
        }
      } catch (e) {
        console.error('Error al guardar')
      }
    },

    async eliminar(id) {
      if (!confirm('¿Eliminar esta etapa?')) return
      await fetch(`${this.url}/${id}?zusuario=${this.zusuario}`, { method: 'DELETE' })
      this.cargar()
    }
  }
}
</script>

<style scoped>
.cabecera { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.formulario { background: #f9f9f9; padding: 15px; border: 1px solid #ccc; margin-bottom: 20px; max-width: 400px; }
input { display: block; width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; border: 1px solid #ccc; }
button { padding: 8px 12px; cursor: pointer; margin-top: 5px; }
.lista { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 15px; margin-top: 15px; }
.tarjeta { background: #fff; border: 1px solid #ddd; padding: 15px; text-align: center; }
.tarjeta small { color: #999; font-size: 12px; }
.tarjeta h4 { margin: 8px 0 5px; }
.tarjeta p { margin: 3px 0; font-size: 13px; color: #555; }
</style>