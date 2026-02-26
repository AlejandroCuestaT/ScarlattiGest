<template>
  <div class="panel-horarios">
    <div class="header-flex">
      <h2>Cronograma de Horarios</h2>
      <button @click="obtenerHorarios" class="btn-refresh">Actualizar Lista</button>
    </div>

    <div class="tabla-container">
      <table class="tabla-gestion">
        <thead>
          <tr>
            <th>Descripción de la Franja</th>
            <th>Inicio</th>
            <th>Fin</th>
            <th>Turno</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="h in horarios" :key="h.id">
            <td class="nombre-franja">{{ h.nombre }}</td>
            <td><span class="badge-hora">{{ h.hora_inicio }}</span></td>
            <td><span class="badge-hora">{{ h.hora_fin }}</span></td>
            <td>
              <span :class="['turno-tag', h.turno_id === '1m' ? 'mañana' : 'tarde']">
                {{ h.turno_id === '1m' ? 'MAÑANA' : 'TARDE' }}
              </span>
            </td>
          </tr>
          <tr v-if="horarios.length === 0">
            <td colspan="4" class="no-data">No hay horarios registrados. Pulsa actualizar.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HorariosComponent',
  data() {
    return {
      horarios: [],
      url: "http://44.207.19.239:3000/horarios",
      zusuario: "acuesta"
    }
  },
  mounted() {
    // Se ejecuta automáticamente al cargar el componente
    this.obtenerHorarios();
  },
  methods: {
    async obtenerHorarios() {
      try {
        const res = await fetch(`${this.url}?zusuario=${this.zusuario}`);
        if (res.ok) {
          this.horarios = await res.json();
        } else {
          console.error("Error al obtener datos");
        }
      } catch (error) {
        console.error("Error de conexión:", error);
      }
    }
  }
}
</script>

<style scoped>
.panel-horarios { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
.header-flex { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }

.tabla-container { overflow-x: auto; }
.tabla-gestion { width: 100%; border-collapse: collapse; font-size: 14px; }
.tabla-gestion th { background: #f8f9fa; padding: 12px; text-align: left; border-bottom: 2px solid #dee2e6; color: #495057; }
.tabla-gestion td { padding: 12px; border-bottom: 1px solid #eee; vertical-align: middle; }

.nombre-franja { font-weight: 500; color: #2c3e50; }
.badge-hora { background: #f0f4f8; padding: 5px 10px; border-radius: 4px; font-family: monospace; font-weight: bold; color: #34495e; border: 1px solid #dcdfe6; }

.turno-tag { padding: 4px 10px; border-radius: 12px; font-size: 11px; font-weight: bold; text-transform: uppercase; }
.turno-tag.mañana { background: #fff4e5; color: #d35400; }
.turno-tag.tarde { background: #e3f2fd; color: #0d47a1; }

.btn-refresh { background: #1877f2; color: white; border: none; padding: 8px 16px; border-radius: 5px; cursor: pointer; font-weight: bold; }
.btn-refresh:hover { background: #1565c0; }
.no-data { text-align: center; padding: 40px; color: #999; }
</style>