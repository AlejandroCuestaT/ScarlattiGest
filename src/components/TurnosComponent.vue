<template>
  <div class="panelTurnos">
    <div class="headerSeccion">
      <h2>Gestión de Turnos Horarios</h2>
      <button @click="alternarFormulario" class="btnAccion">
        {{ formularioVisible ? 'Cancelar' : 'Nuevo Turno' }}
      </button>
    </div>

    <transition name="fade">
      <div v-if="formularioVisible" class="cardFormulario">
        <div class="gridTurnos">
          <div class="campoInput">
            <label>Código ID (Máx. 10 carac.)</label>
            <input 
              v-model="modeloTurno.id" 
              placeholder="Ej: 1M" 
              maxlength="10"
              @input="modeloTurno.id = modeloTurno.id.toUpperCase().trim()"
            >
          </div>
          <div class="campoInput">
            <label>Nombre del Turno</label>
            <input v-model="modeloTurno.nombre" placeholder="Ej: Mañana - Primer Tramo">
          </div>
          <button @click="ejecutarGuardado" class="btnConfirmar">Registrar Turno</button>
        </div>
      </div>
    </transition>

    <div class="tablaWrapper">
      <table class="tablaTurnos">
        <thead>
          <tr>
            <th>Código de Sistema</th>
            <th>Descripción del Turno</th>
            <th>Operaciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="turno in listaTurnos" :key="turno.id">
            <td><span class="badgeTurno">{{ turno.id }}</span></td>
            <td class="txtNombre">{{ turno.nombre }}</td>
            <td>
              <button @click="solicitarEliminacion(turno.id)" class="btnBorrar">
                Eliminar Registro
              </button>
            </td>
          </tr>
          
          <tr v-if="listaTurnos.length === 0">
            <td colspan="3" class="sinDatos">No se han definido turnos en el sistema.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TurnosComponent',
  data() {
    return {
      // Estado de datos
      listaTurnos: [],
      formularioVisible: false,
      // Configuración de API
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Modelo para nuevos registros
      modeloTurno: { 
        id: '', 
        nombre: '' 
      }
    }
  },
  mounted() { 
    this.cargarColeccion(); 
  },
  methods: {
    // Obtiene todos los turnos disponibles
    async cargarColeccion() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/turnos?zusuario=${this.zusuario}`);
        this.listaTurnos = await respuesta.json();
      } catch (error) { 
        console.error("Error al sincronizar turnos:", error); 
      }
    },

    alternarFormulario() {
      this.formularioVisible = !this.formularioVisible;
      if (!this.formularioVisible) this.limpiarModelo();
    },

    // Envía el nuevo turno validando restricciones de la BD
    async ejecutarGuardado() {
      if (!this.modeloTurno.id || this.modeloTurno.id.length > 10) {
        return alert("El ID es obligatorio y debe tener un máximo de 10 caracteres.");
      }

      const payload = {
        ...this.modeloTurno,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}/turnos?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          this.formularioVisible = false;
          this.limpiarModelo();
          this.cargarColeccion();
        } else {
          const errorApi = await respuesta.json();
          alert("No se pudo guardar: " + (errorApi.error || "ID duplicado"));
        }
      } catch (error) { 
        alert("Error de conexión con el servidor central."); 
      }
    },

    // Proceso de borrado con advertencia de integridad
    async solicitarEliminacion(id) {
      if (!confirm(`¿Deseas eliminar el turno ${id}? Asegúrate de que no existan cursos vinculados.`)) return;
      
      try {
        const respuesta = await fetch(`${this.apiUrl}/turnos/${id}?zusuario=${this.zusuario}`, { 
          method: 'DELETE' 
        });
        
        if (respuesta.ok) {
          this.cargarColeccion();
        }
      } catch (error) { 
        alert("Error al intentar procesar la baja."); 
      }
    },

    limpiarModelo() {
      this.modeloTurno = { id: '', nombre: '' };
    }
  }
}
</script>

<style scoped>
.panelTurnos { padding: 30px; background: white; border-radius: 15px; box-shadow: 0 4px 20px rgba(0,0,0,0.03); }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; border-bottom: 1px solid #f1f5f9; padding-bottom: 15px; }

.btnAccion { background: #4f46e5; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: 600; }

.cardFormulario { background: #f8fafc; padding: 25px; border-radius: 12px; border: 1px solid #e2e8f0; margin-bottom: 30px; }
.gridTurnos { display: grid; grid-template-columns: 1fr 2fr auto; gap: 20px; align-items: flex-end; }

.campoInput label { display: block; font-size: 11px; font-weight: 800; color: #64748b; margin-bottom: 8px; text-transform: uppercase; }
.campoInput input { width: 100%; padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; box-sizing: border-box; background: white; }

.btnConfirmar { background: #10b981; color: white; border: none; padding: 13px 25px; border-radius: 8px; font-weight: bold; cursor: pointer; transition: opacity 0.2s; }
.btnConfirmar:hover { opacity: 0.9; }

.tablaWrapper { border: 1px solid #e2e8f0; border-radius: 12px; overflow: hidden; }
.tablaTurnos { width: 100%; border-collapse: collapse; }
.tablaTurnos th { background: #f1f5f9; padding: 15px; text-align: left; color: #475569; font-size: 13px; text-transform: uppercase; letter-spacing: 0.5px; }
.tablaTurnos td { padding: 15px; border-top: 1px solid #f1f5f9; font-size: 14px; }

.badgeTurno { background: #e0e7ff; color: #4338ca; padding: 5px 12px; border-radius: 6px; font-weight: bold; font-family: 'Courier New', monospace; font-size: 12px; }
.txtNombre { color: #1e293b; font-weight: 500; }

.btnBorrar { color: #ef4444; background: none; border: 1px solid transparent; cursor: pointer; font-size: 13px; font-weight: 600; padding: 4px 8px; border-radius: 4px; transition: all 0.2s; }
.btnBorrar:hover { border-color: #fee2e2; background: #fef2f2; }

.sinDatos { text-align: center; color: #94a3b8; padding: 40px !important; font-style: italic; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter, .fade-leave-to { opacity: 0; }
</style>