<template>
  <div class="panel-horarios">
    <h2>Horarios Generales de Ocupación</h2>

    <div class="filtro-fecha">
      <label><b>Ver ocupación para el día:</b></label>
      <input type="date" v-model="fechaConsulta" @change="cargarDatos" class="input-fecha">
    </div>

    <div class="tabla-container">
      <table class="tabla-maestra">
        <thead>
          <tr>
            <th class="col-hora">Franja Horaria</th>
            <th v-for="e in espacios" :key="e.id">{{ e.nombre }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="h in horarios" :key="h.id">
            <td class="celda-hora">
              <b>{{ h.nombre }}</b><br>
              <small>{{ h.hora_inicio }} - {{ h.hora_fin }}</small>
            </td>
            <td v-for="e in espacios" :key="e.id" class="celda-reserva">
              <div v-if="obtenerReserva(e.id, h.id)" class="bloque-ocupado">
                <span class="user-tag">{{ obtenerReserva(e.id, h.id).usuario_login }}</span>
                <small class="motivo-tag">{{ obtenerReserva(e.id, h.id).motivo_reserva }}</small>
              </div>
              <div v-else class="bloque-libre">LIBRE</div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HorariosGeneral',
  data() {
    return {
      espacios: [],
      horarios: [],
      reservas: [],
      fechaConsulta: new Date().toISOString().split('T')[0],
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta"
    }
  },
  mounted() {
    this.cargarDatos();
  },
  methods: {
    async cargarDatos() {
      try {
        const [resE, resH, resR] = await Promise.all([
          fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/horarios?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/reservas?zusuario=${this.zusuario}`)
        ]);

        this.espacios = await resE.json();
        this.horarios = await resH.json();
        const todasLasReservas = await resR.json();

        // Filtramos las reservas por la fecha seleccionada (limpiando el ISO de la BD)
        this.reservas = todasLasReservas.filter(r => {
          const fechaLimpiaDB = r.fecha_reserva ? r.fecha_reserva.split('T')[0] : '';
          return fechaLimpiaDB === this.fechaConsulta;
        });
      } catch (e) {
        console.error("Error cargando cronograma", e);
      }
    },
    obtenerReserva(espacioId, horarioId) {
      // Buscamos si existe una reserva que coincida con ese aula y esa hora
      return this.reservas.find(r => 
        String(r.espacio_id) === String(espacioId) && 
        String(r.horario_id) === String(horarioId)
      );
    }
  }
}
</script>

<style scoped>
.panel-horarios { background: white; padding: 30px; border-radius: 12px; }
.filtro-fecha {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  gap: 15px;
  border: 1px solid #eee;
}
.input-fecha { padding: 8px; border-radius: 6px; border: 1px solid #ccc; }

.tabla-container { overflow-x: auto; }
.tabla-maestra { width: 100%; border-collapse: collapse; min-width: 800px; }
.tabla-maestra th {
  background: #007bff;
  color: white;
  padding: 15px;
  border: 1px solid #0069d9;
  font-size: 14px;
}
.col-hora { width: 150px; background: #343a40 !important; }

.celda-hora { background: #f8f9fa; text-align: center; border: 1px solid #dee2e6; padding: 10px; }
.celda-reserva { border: 1px solid #dee2e6; padding: 5px; vertical-align: middle; text-align: center; height: 60px; }

.bloque-ocupado {
  background: #ffebee;
  color: #c62828;
  padding: 8px;
  border-radius: 6px;
  border-left: 4px solid #ef5350;
  display: flex;
  flex-direction: column;
}
.user-tag { font-weight: bold; font-size: 13px; }
.motivo-tag { font-size: 11px; opacity: 0.8; font-style: italic; }

.bloque-libre { color: #2e7d32; font-size: 11px; font-weight: bold; opacity: 0.5; }
</style>