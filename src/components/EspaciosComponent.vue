<template>
  <div class="gestion-container">
    <div class="header-seccion">
      <h2>Gestión de Espacios (Aulas)</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-principal">
        {{ mostrarForm ? 'Cerrar formulario' : 'Nuevo Espacio' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="formulario-card">
      <h3>Datos del nuevo espacio</h3>
      <form @submit.prevent="guardarEspacio">
        <div class="grupo-input">
          <label>Nombre:</label>
          <input v-model="nuevoEspacio.nombre" placeholder="Ej: Aula 103" required>
        </div>
        <div class="grupo-input">
          <label>Ubicación (Planta):</label>
          <input v-model="nuevoEspacio.ubicacion_planta" placeholder="Ej: Primera planta" required>
        </div>
        <div class="grupo-input">
          <label>Capacidad Máxima:</label>
          <input v-model.number="nuevoEspacio.capacidad_max" type="number" placeholder="Ej: 30" required>
        </div>
        <div class="grupo-input">
          <label>Equipamiento:</label>
          <input v-model="nuevoEspacio.equipamiento" placeholder="Ej: Pizarra, Proyector">
        </div>
        <button type="submit" class="btn-guardar">Guardar Espacio</button>
      </form>
    </div>

    <div class="grid-listado">
      <div v-for="espacio in espacios" :key="espacio.id" class="tarjeta">
        <span class="badge-id">ID: {{ espacio.id }}</span>
        <h4>{{ espacio.nombre }}</h4>
        <p><strong>Planta:</strong> {{ espacio.ubicacion_planta }}</p>
        <p><strong>Capacidad:</strong> {{ espacio.capacidad_max }} personas</p>
        <p><strong>Equipamiento:</strong> {{ espacio.equipamiento || 'Ninguno' }}</p>
        <p><strong>Estado:</strong> {{ espacio.estado_operativo == 'true' || espacio.estado_operativo == '1' ? '✅ Operativo' : '❌ No disponible' }}</p>
        <button @click="eliminarEspacio(espacio.id)" class="btn-eliminar">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      espacios: [],
      mostrarForm: false,
      url: 'http://100.52.46.68:3000/espacios',
      nuevoEspacio: {
        nombre: '',
        ubicacion_planta: '',
        capacidad_max: '',
        equipamiento: '',
        estado_operativo: '1'
      }
    }
  },
  mounted() {
    this.cargarEspacios();
  },
  methods: {
    async cargarEspacios() {
      try {
        const respuesta = await fetch(this.url);
        this.espacios = await respuesta.json();
      } catch (error) {
        console.error("Error al cargar:", error);
      }
    },
    async guardarEspacio() {
      const maxId = this.espacios.length > 0 
        ? Math.max(...this.espacios.map(e => parseInt(e.id))) 
        : 0;
      
      const objetoAEnviar = {
        id: (maxId + 1).toString(),
        nombre: this.nuevoEspacio.nombre,
        ubicacion_planta: this.nuevoEspacio.ubicacion_planta,
        capacidad_max: this.nuevoEspacio.capacidad_max,
        equipamiento: this.nuevoEspacio.equipamiento,
        estado_operativo: "1" 
      };

      try {
        const respuesta = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(objetoAEnviar)
        });

        if (respuesta.ok) {
          this.mostrarForm = false;
          this.nuevoEspacio = { nombre: '', ubicacion_planta: '', capacidad_max: '', equipamiento: '', estado_operativo: '1' };
          this.cargarEspacios();
        } else {
          alert("Error 500: Verifica que todos los campos coincidan con el JSON del servidor.");
        }
      } catch (error) {
        console.error("Error en el POST:", error);
      }
    },
    async eliminarEspacio(id) {
      if (confirm("¿Eliminar este espacio?")) {
        try {
          await fetch(`${this.url}/${id}`, { method: 'DELETE' });
          this.cargarEspacios();
        } catch (error) {
          console.error("Error al borrar:", error);
        }
      }
    }
  }
}
</script>

<style scoped>
/* Estilos consistentes con Departamentos */
.gestion-container { padding: 20px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.formulario-card { background: #fff; padding: 20px; border: 1px solid #ccc; margin-bottom: 25px; max-width: 500px; }
.grupo-input { margin-bottom: 12px; }
.grupo-input label { display: block; font-weight: bold; font-size: 0.85rem; margin-bottom: 4px; }
.grupo-input input { width: 100%; padding: 8px; border: 1px solid #ccc; box-sizing: border-box; }
.btn-principal { background: #007bff; color: #fff; border: none; padding: 10px 15px; cursor: pointer; }
.btn-guardar { width: 100%; background: #007bff; color: white; border: none; padding: 10px; cursor: pointer; margin-top: 10px; }
.grid-listado { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 15px; }
.tarjeta { border: 1px solid #eee; padding: 15px; border-radius: 5px; background: #fff; position: relative; }
.badge-id { background: #007bff; color: white; padding: 2px 6px; font-size: 0.7rem; border-radius: 4px; }
.btn-eliminar { background: #ff4444; color: white; border: none; padding: 5px; width: 100%; cursor: pointer; margin-top: 10px; }
</style>