<template>
  <div class="panelIncidencias">
    <div class="headerSeccion">
      <h2>Soporte Técnico e Incidencias</h2>
      <button @click="mostrarFormulario = !mostrarFormulario" class="btnNuevo">
        {{ mostrarFormulario ? 'Ver Listado' : 'Nueva Incidencia' }}
      </button>
    </div>

    <div v-if="mostrarFormulario" class="cardFormulario">
      <div class="gridForm">
        <div class="campoForm">
          <label>Aula o Espacio afectado</label>
          <select v-model="nuevaIncidencia.espacio_id">
            <option value="" disabled>Seleccionar Aula...</option>
            <option v-for="espacio in listaEspacios" :key="espacio.id" :value="espacio.id">
              {{ espacio.nombre }} ({{ espacio.id }})
            </option>
          </select>
        </div>

        <div class="campoForm">
          <label>Descripción detallada del fallo</label>
          <textarea 
            v-model="nuevaIncidencia.descripcion_problema" 
            placeholder="Ej: El proyector de la zona B parpadea o no tiene señal..."
            rows="4"
          ></textarea>
        </div>
        
        <button @click="crearIncidencia" class="btnGuardar">Enviar a Soporte Técnico</button>
      </div>
    </div>

    <div class="tablaContainer">
      <table class="tablaIncidencias">
        <thead>
          <tr>
            <th>Ticket ID</th>
            <th>Aula</th>
            <th>Descripción del Problema</th>
            <th>Estado Actual</th>
            <th>Reportado por</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="incidencia in incidenciasFiltradas" :key="incidencia.id">
            <td><span class="ticketBadge">#{{ incidencia.id }}</span></td>
            <td><strong>{{ incidencia.espacio_id }}</strong></td>
            <td class="colDescripcion">{{ incidencia.descripcion_problema }}</td>
            <td>
              <span :class="['tagEstado', 'estado' + incidencia.estado_incidencia_id]">
                {{ interpretarEstado(incidencia.estado_incidencia_id) }}
              </span>
            </td>
            <td>{{ incidencia.usuario_login }}</td>
          </tr>
          
          <tr v-if="incidenciasFiltradas.length === 0">
            <td colspan="5" class="sinDatos">
              No hay incidencias registradas bajo este perfil.
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'IncidenciasComponent',
  props: ['usuarioActivo'],
  data() {
    return {
      // Colecciones de datos
      listaIncidencias: [],
      listaEspacios: [],
      // Control de UI
      mostrarFormulario: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Modelo de datos para reportes
      nuevaIncidencia: {
        espacio_id: '',
        descripcion_problema: '',
        estado_incidencia_id: '11111' // Estado inicial "Abierta" por defecto
      }
    }
  },
  computed: {
    //Los alumnos solo ven lo suyo, el resto ve todo
    incidenciasFiltradas() {
      if (this.usuarioActivo?.rol === 'Alum_a') {
        return this.listaIncidencias.filter(i => i.usuario_login === this.usuarioActivo.usuario);
      }
      return this.listaIncidencias;
    }
  },
  mounted() {
    this.cargarDatosIniciales();
  },
  methods: {
    // Carga paralela de incidencias y aulas disponibles
    async cargarDatosIniciales() {
      try {
        const [resInc, resEsp] = await Promise.all([
          fetch(`${this.apiUrl}/incidencias?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`)
        ]);
        this.listaIncidencias = await resInc.json();
        this.listaEspacios = await resEsp.json();
      } catch (error) {
        console.error("Error en la carga de soporte:", error);
      }
    },

    // Convierte los IDs de estado en texto legible para humanos
    interpretarEstado(id) {
      const estados = {
        '11111': 'Abierta',
        '22222': 'En Proceso',
        '33333': 'Resuelta'
      };
      return estados[id] || 'Desconocido';
    },

    // Registra la incidencia con ID manual para evitar conflictos de BD
    async crearIncidencia() {
      if (!this.nuevaIncidencia.espacio_id || !this.nuevaIncidencia.descripcion_problema) {
        return alert("Por favor, rellena todos los campos antes de enviar.");
      }

      // Generación de un ID de 5 dígitos como string para la API
      const idTicket = Math.floor(10000 + Math.random() * 90000).toString();

      const payload = {
        id: idTicket,
        espacio_id: this.nuevaIncidencia.espacio_id,
        usuario_login: this.usuarioActivo.usuario,
        descripcion_problema: this.nuevaIncidencia.descripcion_problema,
        estado_incidencia_id: this.nuevaIncidencia.estado_incidencia_id,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}/incidencias?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          alert(`Incidencia #${idTicket} enviada correctamente.`);
          this.resetearFormulario();
          this.cargarDatosIniciales();
        } else {
          const errorApi = await respuesta.json();
          alert("Error al procesar: " + (errorApi.error || "Datos no válidos"));
        }
      } catch (error) {
        alert("Error de conexión con el servicio de soporte.");
      }
    },

    resetearFormulario() {
      this.nuevaIncidencia = { espacio_id: '', descripcion_problema: '', estado_incidencia_id: '11111' };
      this.mostrarFormulario = false;
    }
  }
}
</script>

<style scoped>
.panelIncidencias { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; border-bottom: 1px solid #f1f5f9; padding-bottom: 15px; }

.cardFormulario { background: #f8fafc; padding: 25px; border-radius: 12px; border: 1px solid #e2e8f0; margin-bottom: 30px; }
.gridForm { display: flex; flex-direction: column; gap: 20px; }
.campoForm label { display: block; margin-bottom: 8px; font-weight: 600; color: #475569; font-size: 14px; }
.campoForm select, .campoForm textarea { padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 14px; width: 100%; box-sizing: border-box; background: white; }

.btnNuevo { background: #64748b; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: 500; }
.btnGuardar { background: #2563eb; color: white; padding: 14px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: background 0.2s; }
.btnGuardar:hover { background: #1d4ed8; }

.tablaContainer { margin-top: 20px; overflow-x: auto; }
.tablaIncidencias { width: 100%; border-collapse: collapse; }
.tablaIncidencias th { text-align: left; padding: 15px; border-bottom: 2px solid #f1f5f9; color: #64748b; font-size: 13px; text-transform: uppercase; }
.tablaIncidencias td { padding: 15px; border-bottom: 1px solid #f1f5f9; font-size: 14px; color: #1e293b; }

.ticketBadge { font-family: monospace; background: #f1f5f9; padding: 4px 8px; border-radius: 4px; font-weight: bold; color: #475569; }
.colDescripcion { max-width: 300px; color: #4b5563; }

.tagEstado { padding: 5px 12px; border-radius: 20px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; }
.estado11111 { background: #fee2e2; color: #dc2626; } /* Abierta - Rojo */
.estado22222 { background: #fef9c3; color: #ca8a04; } /* En Proceso - Amarillo */
.estado33333 { background: #dcfce7; color: #16a34a; } /* Resuelta - Verde */

.sinDatos { text-align: center; padding: 40px !important; color: #94a3b8; font-style: italic; }
</style>