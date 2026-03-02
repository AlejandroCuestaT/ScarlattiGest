<template>
  <div class="adminEspaciosContainer">
    <div v-if="!espacioSeleccionado">
      
      <div class="headerAdmin">
        <h2>Inventario de Espacios Técnicos</h2>
        <button @click="mostrarForm = !mostrarForm" class="btnNuevo">
          {{ mostrarForm ? 'Cancelar' : 'Registrar Aula' }}
        </button>
      </div>

      <div v-if="mostrarForm" class="cardFormularioAdmin">
        <div class="gridAdmin">
          <div class="campo">
            <label>ID Aula (Máx 10 car.)</label>
            <input v-model="nuevaAula.id" maxlength="10" placeholder="Ej: INF-01">
          </div>
          <div class="campo">
            <label>Nombre</label>
            <input v-model="nuevaAula.nombre" placeholder="Ej: Aula de Informática">
          </div>
          <button @click="crearAula" class="btnConfirmarAlta">Confirmar Alta</button>
        </div>
      </div>

      <div class="gridAulas">
        <div v-for="espacio in espacios" :key="espacio.id" class="cardAula">
          <div class="cardBadge">ID: {{ espacio.id }}</div>
          <h3>{{ espacio.nombre }}</h3>
          <p>Planta: {{ espacio.ubicacion_planta }} | Capacidad: {{ espacio.capacidad_max }}</p>
          
          <div class="accionesCard">
            <button @click="entrarEnEspacio(espacio)" class="btnVerReservas">Ver Reservas</button>
            <button @click="eliminarAula(espacio.id)" class="btnBorrar">Eliminar</button>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="detalleReservas">
      <button @click="espacioSeleccionado = null" class="btnBack">Volver al listado</button>
      <h2>Reservas del espacio: {{ espacioSeleccionado.nombre }}</h2>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AdminEspacios',
  props: ['usuarioActivo'],
  data() {
    return {
      // Listas de datos que vienen del servidor
      espacios: [],
      horarios: [],
      // Control de estado de la interfaz
      espacioSeleccionado: null,
      mostrarForm: false,
      // Configuracion de conexion con el servidor
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta", 
      // Objeto temporal para recoger los datos del nuevo formulario
      nuevaAula: { id: '', nombre: '', ubicacion_planta: '', capacidad_max: 20, equipamiento: '' }
    }
  },
  mounted() {
    // Nada más cargar el componente, pedimos los datos al servidor
    this.cargarDatosBase();
  },
  methods: {
    // Función para obtener la lista actualizada de espacios, la usamos para actualizar la pagina tambien
    async cargarDatosBase() {
      const respuesta = await fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`);
      this.espacios = await respuesta.json();
    },

    // Borra un aula usando su ID
    async eliminarAula(id) {
      if (!confirm(`¿Estás seguro de eliminar el aula con ID: ${id}?`)) return;

      try {
        // Hacemos una petición delete a la API incluyendo el ID en la URL
        const respuesta = await fetch(`${this.apiUrl}/espacios/${id}?zusuario=${this.zusuario}`, {
          method: 'DELETE'
        });

        if (respuesta.ok) {
          alert("Aula eliminada correctamente");
          await this.cargarDatosBase(); // Refrescamos la lista para que el cambio se vea
        } else {
          const errorServidor = await respuesta.json();
          alert("Error al eliminar: " + errorServidor.error);
        }
      } catch (e) {
        console.error("Error de comunicación:", e);
        alert("No se pudo conectar con el servidor para eliminar.");
      }
    },

    // Función para enviar los datos del formulario a la base de datos
    async crearAula() {
      if (this.nuevaAula.id.length > 10) return alert("El ID no puede superar los 10 caracteres");
      
      const respuesta = await fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        // Enviamos el objeto con el identificador de auditoría 
        body: JSON.stringify({ 
          //Con esto nos ahorramos el escribir todas las propiedades como id: this.nuevaAula.id
          ...this.nuevaAula, 
          zfecha: new Date().toISOString(), 
          zusuario: this.zusuario 
        })
      });

      //Si todo funciona, se cierra el formulario y hacemos que cargue la pagina directamente
      if (respuesta.ok) {
        this.mostrarForm = false; // Cerramos el formulario
        this.cargarDatosBase();   // Actualizamos la lista
      }
    },

    // Cambia la vista para ver el detalle de un aula
    entrarEnEspacio(espacio) {
      this.espacioSeleccionado = espacio;
    }
  }
}
</script>

<style scoped>
.btnBorrar { 
  background: #fee2e2; 
  color: #dc2626; 
  border: none; 
  padding: 10px; 
  border-radius: 6px; 
  cursor: pointer; 
}
.btnBorrar:hover { 
  background: #fecaca; 
}
.adminEspaciosContainer { padding: 20px; }
.headerAdmin { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.btnNuevo { background: #007bff; color: white; border: none; padding: 10px 20px; border-radius: 5px; cursor: pointer; }
.cardFormularioAdmin { background: white; padding: 20px; border-radius: 8px; margin-bottom: 20px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.gridAulas { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; }
.cardAula { background: white; padding: 15px; border-radius: 10px; border: 1px solid #eee; position: relative; }
.cardBadge { font-size: 10px; color: #888; text-transform: uppercase; }
.btnVerReservas { background: #e0f2fe; color: #0369a1; border: none; padding: 10px; border-radius: 6px; cursor: pointer; margin-right: 10px; }
</style>