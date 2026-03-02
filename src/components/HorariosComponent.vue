<template>
  <div class="panelHorarios">
    <h2>Horarios Generales de Ocupación</h2>

    <div class="filtroFecha">
      <label><b>Consultar cronograma para el día:</b></label>
      <input type="date" v-model="fechaConsulta" @change="cargarDatosCronograma" class="inputFecha">
    </div>

    <div class="tablaContainer">
      <table class="tablaMaestra">
        <thead>
          <tr>
            <th class="colHora">Franja Horaria</th>
            <th v-for="espacio in espacios" :key="espacio.id" class="colEspacio">
              {{ espacio.nombre }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="horario in horarios" :key="horario.id">
            <td class="celdaHora">
              <span class="txtHorarioNombre">{{ horario.nombre }}</span>
              <br>
              <span class="txtHorarioRango">{{ horario.hora_inicio }} - {{ horario.hora_fin }}</span>
            </td>

            <td v-for="espacio in espacios" :key="espacio.id" class="celdaReserva">
              
              <div v-if="obtenerReserva(espacio.id, horario.id)" class="bloqueOcupado">
                <span class="userTag">{{ obtenerReserva(espacio.id, horario.id).usuario_login }}</span>
                <span class="motivoTag">{{ obtenerReserva(espacio.id, horario.id).motivo_reserva }}</span>
              </div>

              <div v-else class="bloqueLibre">LIBRE</div>
              
            </td>
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
      // Colecciones de datos maestros
      espacios: [],
      horarios: [],
      reservasFiltradas: [],
      // Configuración de red y usuario
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Control de tiempo
      fechaConsulta: new Date().toISOString().split('T')[0]
    }
  },
  mounted() {
    // Carga inicial del cuadrante
    this.cargarDatosCronograma();
  },
  methods: {
    // Función central que sincroniza espacios, horarios y reservas del dia
    async cargarDatosCronograma() {
      try {
        const [resEspacios, resHorarios, resReservas] = await Promise.all([
          fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/horarios?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/reservas?zusuario=${this.zusuario}`)
        ]);

        this.espacios = await resEspacios.json();
        this.horarios = await resHorarios.json();
        const todasLasReservas = await resReservas.json();

        // Filtramos para quedarnos solo con lo que ocurre en la fecha seleccionada
        this.reservasFiltradas = todasLasReservas.filter(reserva => {
          const fechaLimpiaDB = reserva.fecha_reserva ? reserva.fecha_reserva.split('T')[0] : '';
          return fechaLimpiaDB === this.fechaConsulta;
        });
      } catch (error) {
        console.error("Error cargando el cronograma maestro:", error);
      }
    },

    // Lógica de búsqueda para rellenar la celda de la tabla
    obtenerReserva(espacioId, horarioId) {
      return this.reservasFiltradas.find(reserva => 
        String(reserva.espacio_id) === String(espacioId) && 
        String(reserva.horario_id) === String(horarioId)
      );
    }
  }
}
</script>

<style scoped>
.panelHorarios { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }

.filtroFecha {
  background: #f8fafc;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  gap: 15px;
  border: 1px solid #e2e8f0;
}

.inputFecha { padding: 8px; border-radius: 6px; border: 1px solid #cbd5e1; outline: none; }
.inputFecha:focus { border-color: #3b82f6; }

.tablaContainer { overflow-x: auto; border-radius: 8px; border: 1px solid #e2e8f0; }
.tablaMaestra { width: 100%; border-collapse: collapse; min-width: 900px; }
.tablaMaestra th {
  background: #f1f5f9;
  color: #475569;
  padding: 15px;
  font-size: 13px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border-bottom: 2px solid #e2e8f0;
}

.colHora { 
  width: 160px; 
  background: #1e293b !important; 
  color: white !important; 
  position: sticky;
  left: 0;
  z-index: 2;
}
.celdaHora { 
  background: #f8fafc; 
  text-align: center; 
  border: 1px solid #e2e8f0; 
  padding: 12px;
  position: sticky;
  left: 0;
  z-index: 1;
}

.txtHorarioNombre { font-weight: bold; color: #1e293b; }
.txtHorarioRango { font-size: 11px; color: #64748b; }
.celdaReserva { 
  border: 1px solid #e2e8f0; 
  padding: 6px; 
  vertical-align: middle; 
  text-align: center; 
  height: 70px; 
  min-width: 140px;
}

.bloqueOcupado {
  background: #fef2f2;
  color: #991b1b;
  padding: 8px;
  border-radius: 6px;
  border-left: 4px solid #ef4444;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
  box-sizing: border-box;
}

.userTag { font-weight: 700; font-size: 12px; margin-bottom: 2px; }
.motivoTag { font-size: 10px; opacity: 0.8; font-style: italic; line-height: 1.2; }

.bloqueLibre { color: #16a34a; font-size: 10px; font-weight: bold; opacity: 0.4; text-transform: uppercase; }
</style>