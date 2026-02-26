<template>
  <div class="panel-espacios">
    <div v-if="!espacioSeleccionado">
      <h2>Aulas Disponibles</h2>
      <div class="grid-aulas">
        <div v-for="e in espacios" :key="e.id" class="card-aula" @click="entrarEnEspacio(e)">
          <div class="card-header"><span class="id-tag">ID: {{ e.id }}</span></div>
          <h3>{{ e.nombre }}</h3>
          <p class="planta">📍 {{ e.ubicacion_planta }}</p>
          <button class="btn-acceder">Gestionar Reservas</button>
        </div>
      </div>
    </div>

    <div v-else class="detalle-container">
      <div class="detalle-header">
        <button @click="espacioSeleccionado = null" class="btn-back">← Volver al listado</button>
        <h2>{{ espacioSeleccionado.nombre }}</h2>
      </div>

      <table class="tabla-reservas">
        <thead>
          <tr>
            <th>Franja Horaria</th>
            <th>Estado</th>
            <th>Profesor</th>
            <th>Acción</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="h in horarios" :key="h.id">
            <td><b>{{ h.nombre }}</b> ({{ h.hora_inicio }} - {{ h.hora_fin }})</td>
            <td>
              <span :class="['pill', estaReservado(h.id) ? 'pill-busy' : 'pill-free']">
                {{ estaReservado(h.id) ? 'OCUPADO' : 'LIBRE' }}
              </span>
            </td>
            <td>{{ obtenerUsuario(h.id) }}</td>
            <td>
              <button v-if="!estaReservado(h.id)" @click="abrirModal(h.id)" class="btn-reserva">Asignarme</button>
              <button v-else disabled class="btn-disabled">No disponible</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="mostrarModal" class="modal-overlay">
      <div class="modal-content">
        <h3>Confirmar Reserva</h3>
        <p>Vas a reservar una franja horaria en <b>{{ espacioSeleccionado.nombre }}</b></p>
        
        <div class="form-group">
          <label>Fecha de la reserva:</label>
          <input type="date" v-model="nuevaReserva.fecha">
        </div>

        <div class="form-group">
          <label>Motivo / Asignatura:</label>
          <input type="text" v-model="nuevaReserva.motivo" placeholder="Ej: Clase de Programación">
        </div>

        <div class="modal-actions">
          <button @click="cerrarModal" class="btn-cancelar">Cancelar</button>
          <button @click="crearReserva" class="btn-confirmar">Confirmar Reserva</button>
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
      espacios: [], horarios: [], reservas: [],
      espacioSeleccionado: null,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      
      // Estado del Modal
      mostrarModal: false,
      horarioAEnviar: null,
      nuevaReserva: {
        fecha: new Date().toISOString().split('T')[0], // Hoy por defecto
        motivo: ''
      }
    }
  },
  mounted() { this.cargarDatos(); },
  methods: {
    async cargarDatos() {
      try {
        const [resE, resH] = await Promise.all([
          fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/horarios?zusuario=${this.zusuario}`)
        ]);
        this.espacios = await resE.json();
        this.horarios = await resH.json();
      } catch (e) { console.error("Error cargando datos base", e); }
    },
    async entrarEnEspacio(espacio) {
      this.espacioSeleccionado = espacio;
      await this.cargarReservas();
    },
    async cargarReservas() {
      try {
        const res = await fetch(`${this.apiUrl}/reservas?zusuario=${this.zusuario}`);
        const data = await res.json();
        this.reservas = data.filter(r => String(r.espacio_id) === String(this.espacioSeleccionado.id));
      } catch (e) { console.error("Error cargando reservas", e); }
    },
    estaReservado(horarioId) {
      return this.reservas.some(r => String(r.horario_id) === String(horarioId));
    },
    obtenerUsuario(horarioId) {
      const r = this.reservas.find(r => String(r.horario_id) === String(horarioId));
      return r ? r.usuario_login : '--'; 
    },

    // Lógica del Modal
    abrirModal(horarioId) {
      this.horarioAEnviar = horarioId;
      this.nuevaReserva.motivo = ''; // Limpiamos el motivo
      this.mostrarModal = true;
    },
    cerrarModal() {
      this.mostrarModal = false;
      this.horarioAEnviar = null;
    },

    async crearReserva() {
      if (!this.nuevaReserva.motivo) {
        alert("Por favor, introduce un motivo.");
        return;
      }

      const idAleatorio = Math.floor(10000 + Math.random() * 90000);
      const nombreProfesor = this.usuarioActivo ? this.usuarioActivo.usuario : "Prof";

      const payload = {
        id: idAleatorio,
        espacio_id: String(this.espacioSeleccionado.id),
        horario_id: String(this.horarioAEnviar),
        usuario_login: nombreProfesor,
        fecha_reserva: this.nuevaReserva.fecha,
        motivo_reserva: this.nuevaReserva.motivo,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const res = await fetch(`${this.apiUrl}/reservas?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          alert(`Reserva creada para ${nombreProfesor}`);
          this.cerrarModal();
          await this.cargarReservas(); 
        }
      } catch (err) {
        console.error("Error al reservar:", err);
      }
    }
  }
}
</script>

<style scoped>
/* ... Tus estilos anteriores se mantienen ... */

/* ESTILOS DEL MODAL */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex; justify-content: center; align-items: center;
  z-index: 1000;
}
.modal-content {
  background: white; padding: 30px; border-radius: 12px;
  width: 400px; box-shadow: 0 10px 25px rgba(0,0,0,0.2);
}
.form-group { margin-top: 20px; text-align: left; }
.form-group label { display: block; font-size: 14px; margin-bottom: 5px; color: #666; }
.form-group input { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; }

.modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 30px; }
.btn-cancelar { background: #eee; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; }
.btn-confirmar { background: #28a745; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; }

/* Reutilizando tus estilos de grid y tabla */
.grid-aulas { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; margin-top: 20px; }
.card-aula { background: white; padding: 20px; border-radius: 12px; border: 1px solid #ddd; cursor: pointer; transition: 0.3s; }
.card-aula:hover { border-color: #007bff; transform: translateY(-5px); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
.btn-acceder { background: #007bff; color: white; border: none; padding: 8px 15px; border-radius: 6px; width: 100%; margin-top: 15px; cursor: pointer; }
.detalle-container { background: white; padding: 30px; border-radius: 12px; }
.tabla-reservas { width: 100%; border-collapse: collapse; }
.tabla-reservas td, .tabla-reservas th { padding: 12px; border-bottom: 1px solid #eee; }
.pill { padding: 4px 10px; border-radius: 20px; font-size: 11px; font-weight: bold; }
.pill-free { background: #e8f5e9; color: #2e7d32; }
.pill-busy { background: #ffebee; color: #c62828; }
.btn-reserva { background: #28a745; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
</style>