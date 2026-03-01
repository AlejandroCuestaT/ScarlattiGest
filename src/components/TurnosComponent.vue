<template>
  <div class="turnos-container">
    <div class="header-section">
      <h2>⏱ Gestión de Turnos del Sistema</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-nuevo">
        {{ mostrarForm ? '✖ Cancelar' : '➕ Crear Nuevo Turno' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="card-formulario">
      <div class="grid-turnos">
        <div class="campo">
          <label>Código Turno (ID - Máx 10)</label>
          <input v-model="nuevoTurno.id" placeholder="Ej: 1m" maxlength="10">
        </div>
        <div class="campo">
          <label>Nombre/Descripción</label>
          <input v-model="nuevoTurno.nombre" placeholder="Ej: Mañana Primera">
        </div>
        <button @click="guardarTurno" class="btn-confirmar">Registrar Turno</button>
      </div>
    </div>

    <div class="tabla-wrapper">
      <table class="tabla-turnos">
        <thead>
          <tr>
            <th>ID (Código)</th>
            <th>Descripción del Turno</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="t in turnos" :key="t.id">
            <td><span class="badge-turno">{{ t.id }}</span></td>
            <td>{{ t.nombre }}</td>
            <td>
              <button @click="eliminarTurno(t.id)" class="btn-borrar">🗑 Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      turnos: [],
      mostrarForm: false,
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      nuevoTurno: { id: '', nombre: '' }
    }
  },
  mounted() { this.cargarTurnos(); },
  methods: {
    async cargarTurnos() {
      try {
        const res = await fetch(`${this.apiUrl}/turnos?zusuario=${this.zusuario}`);
        this.turnos = await res.json();
      } catch (e) { console.error("Error al cargar turnos", e); }
    },
    async guardarTurno() {
      if (!this.nuevoTurno.id || this.nuevoTurno.id.length > 10) {
        return alert("El ID es obligatorio y no puede superar los 10 caracteres.");
      }

      try {
        const res = await fetch(`${this.apiUrl}/turnos?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            ...this.nuevoTurno,
            zfecha: new Date().toISOString(),
            zusuario: this.zusuario
          })
        });

        if (res.ok) {
          this.mostrarForm = false;
          this.nuevoTurno = { id: '', nombre: '' };
          this.cargarTurnos();
        } else {
          const err = await res.json();
          alert("Error al guardar: " + err.error);
        }
      } catch (e) { alert("Error de conexión con la API"); }
    },
    async eliminarTurno(id) {
      if (!confirm(`¿Eliminar el turno ${id}? Esto podría afectar a los cursos vinculados.`)) return;
      
      try {
        await fetch(`${this.apiUrl}/turnos/${id}?zusuario=${this.zusuario}`, { method: 'DELETE' });
        this.cargarTurnos();
      } catch (e) { alert("Error al eliminar"); }
    }
  }
}
</script>

<style scoped>
.turnos-container { padding: 20px; }
.header-section { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.btn-nuevo { background: #4f46e5; color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; }

.card-formulario { background: white; padding: 20px; border-radius: 10px; border: 1px solid #e5e7eb; margin-bottom: 20px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
.grid-turnos { display: grid; grid-template-columns: 1fr 2fr auto; gap: 15px; align-items: end; }
.campo label { display: block; font-size: 12px; font-weight: bold; color: #6b7280; margin-bottom: 5px; }
.campo input { width: 100%; padding: 10px; border: 1px solid #d1d5db; border-radius: 6px; }

.btn-confirmar { background: #10b981; color: white; border: none; padding: 11px 20px; border-radius: 6px; font-weight: bold; cursor: pointer; }

.tabla-wrapper { background: white; border-radius: 10px; border: 1px solid #e5e7eb; overflow: hidden; }
.tabla-turnos { width: 100%; border-collapse: collapse; }
.tabla-turnos th { background: #f9fafb; padding: 15px; text-align: left; color: #4b5563; font-size: 13px; }
.tabla-turnos td { padding: 15px; border-top: 1px solid #f3f4f6; }

.badge-turno { background: #e0e7ff; color: #4338ca; padding: 4px 10px; border-radius: 4px; font-weight: bold; font-family: monospace; }
.btn-borrar { color: #dc2626; background: none; border: none; cursor: pointer; font-size: 13px; }
.btn-borrar:hover { text-decoration: underline; }
</style>