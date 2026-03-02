<template>
  <div class="panelEspacios">
    
    <div v-if="!espacioSeleccionado">
      <h2>Aulas Disponibles</h2>
      <div class="gridAulas">
        <div v-for="espacio in espacios" :key="espacio.id" class="cardAula" @click="entrarEnEspacio(espacio)">
          <div class="cardHeader">
            <span class="idTag">ID: {{ espacio.id }}</span>
          </div>
          <h3>{{ espacio.nombre }}</h3>
          <p class="planta">Ubicación: {{ espacio.ubicacion_planta }}</p>
          <button class="btnAcceder">Gestionar Reservas</button>
        </div>
      </div>
    </div>

    <div v-else class="detalleContainer">
      <div class="detalleHeader">
        <button @click="espacioSeleccionado = null" class="btnBack">Volver al listado</button>
        <h2>{{ espacioSeleccionado.nombre }}</h2>
      </div>

      <div class="filtroFecha">
        <label><b>Día a consultar o reservar:</b></label>
        <input type="date" v-model="fechaConsulta" @change="cargarReservas" class="inputFecha">
      </div>

      <table class="tablaReservas">
        <thead>
          <tr>
            <th>Franja Horaria</th>
            <th>Estado</th>
            <th>Profesor / Usuario</th>
            <th>Acción</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="horario in horarios" :key="horario.id">
            <td>
              <b>{{ horario.nombre }}</b> 
              <span class="txtSecundario">({{ horario.hora_inicio }} - {{ horario.hora_fin }})</span>
            </td>
            <td>
              <span :class="['pill', estaReservado(horario.id) ? 'pillBusy' : 'pillFree']">
                {{ estaReservado(horario.id) ? 'OCUPADO' : 'LIBRE' }}
              </span>
            </td>
            <td>{{ obtenerUsuario(horario.id) }}</td>
            <td>
              <button v-if="!estaReservado(horario.id)" @click="abrirModal(horario.id)" class="btnReserva">
                Reservar
              </button>
              <button v-else disabled class="btnDisabled">No disponible</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="mostrarModal" class="modalOverlay">
      <div class="modalContent">
        <h3>Confirmar Reserva</h3>
        <p>Aula: <b>{{ espacioSeleccionado.nombre }}</b></p>
        
        <div class="formGroup">
          <label>Fecha de la reserva:</label>
          <input type="date" :value="fechaConsulta" disabled class="inputDisabled">
        </div>

        <div class="formGroup">
          <label>Motivo o Asignatura:</label>
          <input type="text" v-model="motivoReserva" placeholder="Ej: Uso de ordenadores necesarios">
        </div>

        <div class="modalActions">
          <button @click="cerrarModal" class="btnCancelar">Cancelar</button>
          <button @click="crearReserva" class="btnConfirmar">Confirmar Reserva</button>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: 'EspaciosComponent',
  props: ['usuarioActivo'],
  data() {
    return {
      // Datos maestros de la API
      espacios: [],
      horarios: [],
      reservas: [],
      // Estado de la interfaz
      espacioSeleccionado: null,
      mostrarModal: false,
      // Configuración de red invariable
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Lógica de reserva
      fechaConsulta: new Date().toISOString().split('T')[0], 
      horarioSeleccionadoId: null,
      motivoReserva: ''
    }
  },
  mounted() {
    this.cargarDatosBase();
  },
  methods: {
    // Carga inicial de aulas y franjas horarias
    async cargarDatosBase() {
      try {
        const [resEspacios, resHorarios] = await Promise.all([
          fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/horarios?zusuario=${this.zusuario}`)
        ]);
        this.espacios = await resEspacios.json();
        this.horarios = await resHorarios.json();
      } catch (e) {
        console.error("Error cargando datos base:", e);
      }
    },

    // Selecciona un aula y busca sus reservas para la fecha elegida
    async entrarEnEspacio(espacio) {
      this.espacioSeleccionado = espacio;
      await this.cargarReservas();
    },

    // Obtiene las reservas y las filtra por aula y fecha seleccionada
    async cargarReservas() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/reservas?zusuario=${this.zusuario}`);
        const todasLasReservas = await respuesta.json();
        
        // Filtramos para mostrar solo las del aula actual y el día que el usuario está mirando
        this.reservas = todasLasReservas.filter(reserva => {
          const mismoEspacio = String(reserva.espacio_id) === String(this.espacioSeleccionado.id);
          const fechaLimpiaDB = reserva.fecha_reserva ? reserva.fecha_reserva.split('T')[0] : '';
          return mismoEspacio && (fechaLimpiaDB === this.fechaConsulta);
        });
      } catch (e) {
        console.error("Error al filtrar reservas:", e);
      }
    },

    // Comprueba si una franja horaria específica ya está ocupada
    estaReservado(horarioId) {
      return this.reservas.some(r => String(r.horario_id) === String(horarioId));
    },

    // Devuelve el nombre del profesor que ha reservado el hueco
    obtenerUsuario(horarioId) {
      const reserva = this.reservas.find(r => String(r.horario_id) === String(horarioId));
      return reserva ? reserva.usuario_login : '--';
    },

    // Lógica del Modal
    abrirModal(horarioId) {
      this.horarioSeleccionadoId = horarioId;
      this.motivoReserva = '';
      this.mostrarModal = true;
    },
    cerrarModal() {
      this.mostrarModal = false;
      this.horarioSeleccionadoId = null;
    },

    // Registra la nueva reserva en el servidor
    async crearReserva() {
      if (!this.motivoReserva) return alert("Es obligatorio indicar un motivo para la reserva");

      // Generamos un ID para la reserva y detectamos al usuario logueado
      const idUnico = Math.floor(10000 + Math.random() * 90000);
      const loginUsuario = this.usuarioActivo ? this.usuarioActivo.usuario : "Prof";

      const payload = {
        id: idUnico,
        espacio_id: String(this.espacioSeleccionado.id),
        horario_id: String(this.horarioSeleccionadoId),
        usuario_login: loginUsuario,
        fecha_reserva: this.fechaConsulta,
        motivo_reserva: this.motivoReserva,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}/reservas?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          alert("Reserva guardada correctamente");
          this.cerrarModal();
          await this.cargarReservas(); // Recargamos para que el hueco pase a OCUPADO
        }
      } catch (error) {
        console.error("Error en la reserva:", error);
      }
    }
  }
}
</script>

<style scoped>
.panelEspacios { padding: 10px; }

.gridAulas { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; margin-top: 20px; }
.cardAula { background: white; padding: 20px; border-radius: 12px; border: 1px solid #ddd; cursor: pointer; transition: all 0.2s; }
.cardAula:hover { transform: translateY(-3px); box-shadow: 0 5px 15px rgba(0,0,0,0.1); border-color: #007bff; }
.idTag { font-size: 11px; color: #888; font-weight: bold; }
.btnAcceder { background: #007bff; color: white; border: none; padding: 10px; border-radius: 6px; width: 100%; margin-top: 15px; cursor: pointer; }

.detalleContainer { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
.detalleHeader { display: flex; align-items: center; gap: 20px; margin-bottom: 20px; }
.btnBack { background: none; border: none; color: #007bff; font-weight: bold; cursor: pointer; font-size: 14px; }
.filtroFecha { background: #f8fafc; padding: 15px; border-radius: 8px; margin-bottom: 25px; border: 1px solid #e2e8f0; display: flex; align-items: center; gap: 15px; }
.inputFecha { padding: 8px; border-radius: 6px; border: 1px solid #cbd5e1; }

.tablaReservas { width: 100%; border-collapse: collapse; }
.tablaReservas th { text-align: left; padding: 12px; border-bottom: 2px solid #f1f5f9; color: #64748b; }
.tablaReservas td { padding: 15px; border-bottom: 1px solid #f1f5f9; }
.txtSecundario { color: #94a3b8; font-size: 13px; font-weight: normal; }

.pill { padding: 5px 12px; border-radius: 15px; font-size: 11px; font-weight: bold; }
.pillFree { background: #dcfce7; color: #166534; }
.pillBusy { background: #fee2e2; color: #991b1b; }

.btnReserva { background: #22c55e; color: white; border: none; padding: 8px 15px; border-radius: 6px; cursor: pointer; }
.btnDisabled { background: #f1f5f9; color: #94a3b8; border: none; padding: 8px 15px; border-radius: 6px; cursor: not-allowed; }
.modalOverlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); display: flex; justify-content: center; align-items: center; z-index: 1000; }
.modalContent { background: white; padding: 30px; border-radius: 15px; width: 400px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); }
.formGroup { margin-top: 20px; }
.formGroup label { display: block; font-size: 13px; color: #475569; margin-bottom: 6px; }
.formGroup input { width: 100%; padding: 10px; border: 1px solid #d1d5db; border-radius: 6px; box-sizing: border-box; }
.inputDisabled { background-color: #f8fafc; color: #94a3b8; cursor: not-allowed; }
.modalActions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 30px; }
.btnCancelar { background: #f1f5f9; color: #475569; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; }
.btnConfirmar { background: #22c55e; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
</style>