<template>
  <div class="panelIncidenciasTics">
    <div class="headerSeccion">
      <h2>Gestión de Soporte TIC</h2>
      
      <div class="filtrosEstado">
        <button @click="filtroActual = 'todos'" :class="{ active: filtroActual === 'todos' }">Todas</button>
        <button @click="filtroActual = '11111'" :class="{ active: filtroActual === '11111' }">Pendientes</button>
        <button @click="filtroActual = '22222'" :class="{ active: filtroActual === '22222' }">En Proceso</button>
        <button @click="filtroActual = '33333'" :class="{ active: filtroActual === '33333' }">Resueltas</button>
      </div>
    </div>

    <div class="gridIncidencias">
      <div v-for="incidencia in incidenciasFiltradas" :key="incidencia.id" class="cardIncidencia">
        <div class="cardHeader">
          <span :class="['statusBadge', obtenerClaseEstado(incidencia.estado_incidencia_id)]">
            {{ obtenerTextoEstado(incidencia.estado_incidencia_id) }}
          </span>
          <span class="txtFecha">{{ incidencia.zfecha ? incidencia.zfecha.split('T')[0] : 'Sin fecha' }}</span>
        </div>
        
        <h3>Ticket #{{ incidencia.id }}</h3>
        <p class="txtDescripcion">{{ incidencia.descripcion_problema || 'El usuario no ha proporcionado una descripción.' }}</p>
        
        <div class="infoMeta">
          <span>Ubicación: <strong>{{ incidencia.espacio_id }}</strong></span>
          <span>Usuario: <strong>{{ incidencia.usuario_login }}</strong></span>
        </div>

        <div class="accionesTecnicas">
          <label>Cambiar Estado del Ticket:</label>
          <select @change="actualizarEstadoTicket(incidencia.id, $event.target.value)" :value="incidencia.estado_incidencia_id">
            <option value="11111">PENDIENTE (Sin asignar)</option>
            <option value="22222">EN PROCESO (Revisión)</option>
            <option value="33333">RESUELTA (Cerrado)</option>
          </select>
        </div>
      </div>
    </div>

    <div v-if="incidenciasFiltradas.length === 0" class="sinResultados">
      No se han encontrado incidencias con el filtro seleccionado.
    </div>
  </div>
</template>

<script>
export default {
  name: 'IncidenciasTIC',
  data() {
    return {
      listaIncidencias: [],
      filtroActual: 'todos',
      // Endpoint base configurado para operaciones CRUD
      apiUrl: "http://44.207.19.239:3000/incidencias",
      zusuario: "acuesta" 
    }
  },
  computed: {
    // Lógica de filtrado reactivo
    incidenciasFiltradas() {
      if (this.filtroActual === 'todos') return this.listaIncidencias;
      return this.listaIncidencias.filter(i => i.estado_incidencia_id === this.filtroActual);
    }
  },
  mounted() {
    this.cargarIncidencias();
  },
  methods: {
    // Obtiene el volcado completo de incidencias para el equipo técnico
    async cargarIncidencias() {
      try {
        const respuesta = await fetch(`${this.apiUrl}?zusuario=${this.zusuario}`);
        if (!respuesta.ok) throw new Error("Error de red");
        this.listaIncidencias = await respuesta.json();
      } catch (error) {
        console.error("Error al sincronizar con el servidor TIC:", error);
      }
    },

    // Actualiza el registro mediante el método PUT
    async actualizarEstadoTicket(id, nuevoEstadoId) {
      // Formato de URL según requerimientos de la API: url/id?zusuario=...
      const urlDestino = `${this.apiUrl}/${id}?zusuario=${this.zusuario}`;
      
      try {
        const respuesta = await fetch(urlDestino, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ 
            estado_incidencia_id: nuevoEstadoId,
            zusuario: this.zusuario,
            zfecha: new Date().toISOString()
          })
        });

        if (respuesta.ok) {
          // Recarga silenciosa para confirmar el cambio en la UI
          await this.cargarIncidencias();
        } else {
          alert("No se pudo actualizar el estado. Inténtalo de nuevo.");
          this.cargarIncidencias(); // Revertimos cambios locales
        }
      } catch (error) {
        console.error("Error de conexión en el proceso PUT:", error);
      }
    },

    // Ayudantes visuales
    obtenerClaseEstado(id) {
      const clases = { '11111': 'bgError', '22222': 'bgWarning', '33333': 'bgSuccess' };
      return clases[id] || '';
    },
    
    obtenerTextoEstado(id) {
      const textos = { '11111': 'PENDIENTE', '22222': 'EN PROCESO', '33333': 'RESUELTA' };
      return textos[id] || id;
    }
  }
}
</script>

<style scoped>
.panelIncidenciasTics { background: #f8fafc; padding: 30px; min-height: 100vh; }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }

.filtrosEstado button { 
  padding: 10px 18px; 
  margin-left: 10px; 
  border: 1px solid #cbd5e1; 
  background: white; 
  border-radius: 8px; 
  cursor: pointer; 
  font-weight: 500;
  transition: all 0.2s;
}

.filtrosEstado button.active { background: #3b82f6; color: white; border-color: #3b82f6; box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.3); }

.gridIncidencias { display: grid; grid-template-columns: repeat(auto-fill, minmax(360px, 1fr)); gap: 20px; }
.cardIncidencia { 
  background: white; 
  padding: 25px; 
  border-radius: 12px; 
  box-shadow: 0 4px 6px rgba(0,0,0,0.05); 
  border-top: 5px solid #3b82f6; 
  transition: transform 0.2s;
}
.cardIncidencia:hover { transform: translateY(-4px); }

.cardHeader { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
.statusBadge { padding: 5px 12px; border-radius: 6px; font-size: 11px; font-weight: 800; letter-spacing: 0.5px; }

.bgError { background: #fee2e2; color: #dc2626; }
.bgWarning { background: #fef3c7; color: #d97706; }
.bgSuccess { background: #dcfce7; color: #16a34a; }

.txtFecha { font-size: 12px; color: #94a3b8; font-weight: 500; }
.txtDescripcion { color: #475569; margin: 20px 0; min-height: 50px; font-size: 14px; line-height: 1.6; }

.infoMeta { 
  display: flex; 
  justify-content: space-between; 
  font-size: 12px; 
  color: #64748b; 
  background: #f1f5f9; 
  padding: 10px;
  border-radius: 6px;
}

.accionesTecnicas { margin-top: 25px; }
.accionesTecnicas label { display: block; font-size: 11px; font-weight: 700; color: #64748b; margin-bottom: 8px; text-transform: uppercase; }
.accionesTecnicas select { 
  width: 100%; 
  padding: 12px; 
  border-radius: 8px; 
  border: 1px solid #e2e8f0; 
  background: #f9fafb; 
  font-weight: 600; 
  color: #1e293b;
  cursor: pointer;
}

.sinResultados { text-align: center; padding: 60px; color: #94a3b8; font-style: italic; }
</style>