<template>
  <div class="panel-incidencias">
    <div class="header-seccion">
      <h2>🛠️ Panel de Control TIC</h2>
      <div class="filtros">
        <button @click="filtro = 'todos'" :class="{active: filtro === 'todos'}">Todas</button>
        <button @click="filtro = '11111'" :class="{active: filtro === '11111'}">Pendientes</button>
        <button @click="filtro = '22222'" :class="{active: filtro === '22222'}">En Proceso</button>
        <button @click="filtro = '33333'" :class="{active: filtro === '33333'}">Resueltas</button>
      </div>
    </div>

    <div class="grid-incidencias">
      <div v-for="inc in incidenciasFiltradas" :key="inc.id" class="card-incidencia">
        <div class="card-header">
          <span :class="['status-badge', getStatusClass(inc.estado_incidencia_id)]">
            {{ getStatusText(inc.estado_incidencia_id) }}
          </span>
          <span class="fecha">{{ inc.zfecha ? inc.zfecha.split('T')[0] : 'S/F' }}</span>
        </div>
        
        <h3>Ticket #{{ inc.id }}</h3>
        <p class="descripcion">{{ inc.descripcion_problema || 'Sin descripción' }}</p>
        
        <div class="info-footer">
          <span>📍 <strong>Espacio:</strong> {{ inc.espacio_id }}</span>
          <span>👤 <strong>User:</strong> {{ inc.usuario_login }}</span>
        </div>

        <div class="acciones-tic">
          <label>Actualizar Estado:</label>
          <select @change="actualizarEstado(inc.id, $event.target.value)" :value="inc.estado_incidencia_id">
            <option value="11111">Pendiente</option>
            <option value="22222">En Proceso</option>
            <option value="33333">Resuelta</option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'IncidenciasTIC',
  data() {
    return {
      incidencias: [],
      filtro: 'todos',
      // Base URL sin la barra final para construir la ruta dinámicamente
      apiUrl: "http://44.207.19.239:3000/incidencias",
      zusuario: "acuesta" 
    }
  },
  computed: {
    incidenciasFiltradas() {
      if (this.filtro === 'todos') return this.incidencias;
      return this.incidencias.filter(i => i.estado_incidencia_id === this.filtro);
    }
  },
  mounted() {
    this.cargarIncidencias();
  },
  methods: {
    async cargarIncidencias() {
      try {
        const res = await fetch(`${this.apiUrl}?zusuario=${this.zusuario}`);
        this.incidencias = await res.json();
      } catch (e) {
        console.error("Error cargando datos", e);
      }
    },
    async actualizarEstado(id, nuevoEstadoId) {
      // Construcción exacta según tu imagen: `${BASE_URL}/${id}${zID}`
      // Donde zID incluye el signo de interrogación y el usuario
      const zID = `?zusuario=${this.zusuario}`;
      const urlCompleta = `${this.apiUrl}/${id}${zID}`;
      
      try {
        const res = await fetch(urlCompleta, {
          method: 'PUT', // Método definido en tu código
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ 
            estado_incidencia_id: nuevoEstadoId, // Cambiamos el ID del estado
            zusuario: this.zusuario,
            zfecha: new Date().toISOString()
          })
        });

        if (res.ok) {
          await this.cargarIncidencias();
        } else {
          console.error("Error en la respuesta de la API");
          this.cargarIncidencias();
        }
      } catch (e) {
        alert("Error de conexión");
      }
    },
    getStatusClass(id) {
      if (id === '11111') return 'bg-error';
      if (id === '22222') return 'bg-warning';
      return 'bg-success';
    },
    getStatusText(id) {
      const mapeo = { '11111': 'PENDIENTE', '22222': 'EN PROCESO', '33333': 'RESUELTA' };
      return mapeo[id] || id;
    }
  }
}
</script>

<style scoped>
.panel-incidencias { background: #f8fafc; padding: 20px; min-height: 100vh; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.filtros button { padding: 8px 16px; margin-left: 10px; border: 1px solid #cbd5e1; background: white; border-radius: 8px; cursor: pointer; }
.filtros button.active { background: #3b82f6; color: white; border-color: #3b82f6; }

.grid-incidencias { display: grid; grid-template-columns: repeat(auto-fill, minmax(350px, 1fr)); gap: 20px; }
.card-incidencia { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); border-top: 4px solid #3b82f6; }

.card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.status-badge { padding: 4px 10px; border-radius: 8px; font-size: 11px; font-weight: bold; }
.bg-error { background: #fee2e2; color: #dc2626; }
.bg-warning { background: #fef3c7; color: #d97706; }
.bg-success { background: #dcfce7; color: #16a34a; }

.descripcion { color: #475569; margin: 15px 0; min-height: 40px; font-size: 14px; }
.info-footer { display: flex; justify-content: space-between; font-size: 12px; color: #64748b; border-top: 1px solid #f1f5f9; padding-top: 10px; }

.acciones-tic { margin-top: 20px; }
.acciones-tic label { display: block; font-size: 11px; font-weight: bold; color: #94a3b8; margin-bottom: 5px; }
.acciones-tic select { width: 100%; padding: 10px; border-radius: 8px; border: 1px solid #e2e8f0; background: #f9fafb; font-weight: bold; }
</style>