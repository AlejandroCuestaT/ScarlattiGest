<template>
  <div class="panel-incidencias">
    <div class="header-seccion">
      <h2>🛠️ Soporte Técnico</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-nuevo">
        {{ mostrarForm ? '✖ Ver Listado' : '⚠️ Nueva Incidencia' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="card-formulario">
      <div class="grid-form">
        <div class="campo">
          <label>Aula / Espacio</label>
          <select v-model="nueva.espacio_id">
            <option value="" disabled>Seleccionar Aula...</option>
            <option v-for="e in espacios" :key="e.id" :value="e.id">
              {{ e.nombre }} ({{ e.id }})
            </option>
          </select>
        </div>

        <div class="campo">
          <label>Descripción del fallo</label>
          <textarea 
            v-model="nueva.descripcion_problema" 
            placeholder="Describa el problema (ej: El proyector no enciende)..."
            rows="4"
          ></textarea>
        </div>
        
        <button @click="crearIncidencia" class="btn-guardar">Enviar a Soporte</button>
      </div>
    </div>

    <div class="tabla-container">
      <table class="tabla-alumnos">
        <thead>
          <tr>
            <th>ID</th>
            <th>Aula</th>
            <th>Descripción</th>
            <th>Estado</th>
            <th>Reportado por</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="i in incidenciasFiltradas" :key="i.id">
            <td>#{{ i.id }}</td>
            <td><strong>{{ i.espacio_id }}</strong></td>
            <td>{{ i.descripcion_problema }}</td>
            <td>
              <span :class="['tag-estado', 'estado-' + i.estado_incidencia_id]">
                {{ i.estado_incidencia_id === '11111' ? 'Abierta' : (i.estado_incidencia_id === '22222' ? 'En Proceso' : 'Resuelta') }}
              </span>
            </td>
            <td>{{ i.usuario_login }}</td>
          </tr>
          <tr v-if="incidenciasFiltradas.length === 0">
            <td colspan="5" style="text-align: center; padding: 20px; color: #666;">
              No hay incidencias registradas.
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
      incidencias: [],
      espacios: [],
      mostrarForm: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      nueva: {
        espacio_id: '',
        descripcion_problema: '',
        estado_incidencia_id: '11111' // ID para "Abierta" creado en Postman
      }
    }
  },
  computed: {
    incidenciasFiltradas() {
      // Los alumnos solo ven sus propias incidencias
      if (this.usuarioActivo.rol === 'Alum_a') {
        return this.incidencias.filter(i => i.usuario_login === this.usuarioActivo.usuario);
      }
      return this.incidencias;
    }
  },
  mounted() {
    this.cargarDatos();
  },
  methods: {
    async cargarDatos() {
      try {
        const [resInc, resEsp] = await Promise.all([
          fetch(`${this.apiUrl}/incidencias?zusuario=${this.zusuario}`),
          fetch(`${this.apiUrl}/espacios?zusuario=${this.zusuario}`)
        ]);
        this.incidencias = await resInc.json();
        this.espacios = await resEsp.json();
      } catch (e) {
        console.error("Error cargando datos:", e);
      }
    },
    async crearIncidencia() {
      if (!this.nueva.espacio_id || !this.nueva.descripcion_problema) {
        return alert("Por favor, selecciona un aula y describe el problema.");
      }

      // Generamos un ID de 5 cifras para evitar el error de 'duplicate key'
      const idAleatorio = Math.floor(10000 + Math.random() * 90000).toString();

      const payload = {
        id: idAleatorio, // Enviado manualmente para evitar fallos de autoincremento
        espacio_id: this.nueva.espacio_id,
        usuario_login: this.usuarioActivo.usuario, // Mapeado al campo de login
        descripcion_problema: this.nueva.descripcion_problema, // Campo exacto de la BD
        estado_incidencia_id: this.nueva.estado_incidencia_id,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const res = await fetch(`${this.apiUrl}/incidencias?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          alert(`Incidencia #${idAleatorio} creada con éxito.`);
          this.nueva.descripcion_problema = '';
          this.mostrarForm = false;
          this.cargarDatos();
        } else {
          const err = await res.json();
          alert("Error del servidor: " + (err.error || err.message));
        }
      } catch (e) {
        alert("Error de conexión con la API.");
      }
    }
  }
}
</script>

<style scoped>
.panel-incidencias { background: #fff; padding: 25px; border-radius: 15px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
.card-formulario { background: #f8fafc; padding: 25px; border-radius: 12px; border: 1px solid #e2e8f0; margin-bottom: 30px; }
.grid-form { display: flex; flex-direction: column; gap: 20px; }
.campo label { display: block; margin-bottom: 8px; font-weight: bold; color: #475569; }
.grid-form select, .grid-form textarea { padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 14px; width: 100%; box-sizing: border-box; }
.btn-nuevo { background: #64748b; color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; }
.btn-guardar { background: #2563eb; color: white; padding: 14px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: background 0.2s; }
.btn-guardar:hover { background: #1d4ed8; }

.tag-estado { padding: 4px 12px; border-radius: 20px; font-size: 11px; font-weight: bold; text-transform: uppercase; }
.estado-11111 { background: #fee2e2; color: #dc2626; } /* Abierta */
.estado-22222 { background: #fef9c3; color: #ca8a04; } /* En Proceso */
.estado-33333 { background: #dcfce7; color: #16a34a; } /* Resuelta */

.tabla-container { margin-top: 20px; overflow-x: auto; }
.tabla-alumnos { width: 100%; border-collapse: collapse; }
.tabla-alumnos th { text-align: left; padding: 12px; border-bottom: 2px solid #e2e8f0; color: #64748b; }
.tabla-alumnos td { padding: 12px; border-bottom: 1px solid #e2e8f0; font-size: 14px; }
</style>