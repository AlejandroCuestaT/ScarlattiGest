<template>
  <div class="panelUsuarios">
    <div class="headerSeccion">
      <h2>Control de Accesos</h2>
      <button @click="alternarFormulario" class="btnAccion">
        {{ formularioVisible ? 'Cancelar Registro' : 'Crear Nuevo Acceso' }}
      </button>
    </div>

    <transition name="slide">
      <div v-if="formularioVisible" class="cardFormulario">
        <div class="gridCampos">
          <div class="grupoInput">
            <label>Nombre de Usuario (Login)</label>
            <input v-model="modeloUsuario.login" placeholder="Ej: Prof2">
          </div>
          <div class="grupoInput">
            <label>Contraseña Temporal</label>
            <input v-model="modeloUsuario.password_hash" type="password" placeholder="••••">
          </div>
          <div class="grupoInput">
            <label>ID de Identidad (NIA o DNI)</label>
            <input v-model="modeloUsuario.ref_identidad_fk" placeholder="Vinculacion con Profesor o Alumno">
          </div>
          <div class="grupoInput">
            <label>Perfil de Usuario</label>
            <select v-model="modeloUsuario.rol_id">
              <option value="" disabled>Seleccionar Perfil...</option>
              <option value="Admin_a">Admin</option>
              <option value="Prof_a">Profesor</option>
              <option value="Alum_a">Alumno</option>
              <option value="Tic_a">TIC</option>
            </select>
          </div>

          <button 
            @click="ejecutarCreacion" 
            class="btnGuardar" 
            :disabled="estaCargando"
          >
            {{ estaCargando ? 'Sincronizando...' : 'Vincular y Generar Acceso' }}
          </button>
        </div>
      </div>
    </transition>

    <div class="tablaContainer">
      <table class="tablaUsuarios">
        <thead>
          <tr>
            <th>Identificador Acceso</th>
            <th>Nivel de Perfil</th>
            <th>Referencia de Identidad</th>
            <th>Actividad Reciente</th>
            <th>Gestión</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="usuario in listaUsuarios" :key="usuario.login">
            <td><strong class="txtLogin">{{ usuario.login }}</strong></td>
            <td>
              <span :class="['tagRol', obtenerEstiloRol(usuario.rol_id)]">
                {{ usuario.rol_id }}
              </span>
            </td>
            <td><code class="badgeIdentidad">{{ usuario.ref_identidad_fk }}</code></td>
            <td class="txtFecha">
              {{ usuario.ultimo_acceso ? usuario.ultimo_acceso.split('T')[0] : 'Sin registros' }}
            </td>
            <td>
              <button @click="solicitarBaja(usuario.login)" class="btnEliminar">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UsuariosComponent',
  data() {
    return {
      // Estado de datos
      listaUsuarios: [],
      formularioVisible: false,
      estaCargando: false,
      // Configuración de servicio
      apiUrl: "http://44.207.19.239:3000",
      zusuario: "acuesta",
      // Modelo de datos inicial
      modeloUsuario: {
        login: '',
        password_hash: '',
        rol_id: '',
        ref_identidad_fk: '',
        estado_id: 'Act_a'
      }
    }
  },
  mounted() {
    this.cargarDatos();
  },
  methods: {
    // Obtiene el censo completo de cuentas de acceso
    async cargarDatos() {
      try {
        const respuesta = await fetch(`${this.apiUrl}/usuarios?zusuario=${this.zusuario}`);
        this.listaUsuarios = await respuesta.json();
      } catch (error) {
        console.error("Fallo al obtener la lista de cuentas:", error);
      }
    },

    alternarFormulario() {
      this.formularioVisible = !this.formularioVisible;
      if (!this.formularioVisible) this.resetearModelo();
    },

    // Procesa el alta de la cuenta vinculándola a una identidad física
    async ejecutarCreacion() {
      if (!this.modeloUsuario.login || !this.modeloUsuario.rol_id || !this.modeloUsuario.ref_identidad_fk) {
        return alert("Error: Todos los campos de vinculación son obligatorios.");
      }

      this.estaCargando = true;
      const payload = {
        ...this.modeloUsuario,
        zfecha: new Date().toISOString(),
        zusuario: this.zusuario
      };

      try {
        const respuesta = await fetch(`${this.apiUrl}/usuarios?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        const datosRespuesta = await respuesta.json();

        if (respuesta.ok) {
          alert("Cuenta vinculada con éxito.");
          this.resetearModelo();
          this.formularioVisible = false;
          await this.cargarDatos();
        } else {
          // Captura errores de Foreign Key o duplicidad de Login
          alert("Error de Sistema: " + (datosRespuesta.error || "La identidad vinculada no existe."));
        }
      } catch (error) {
        alert("Error de red: No se pudo conectar con el servidor de autenticación.");
      } finally {
        this.estaCargando = false;
      }
    },

    // Baja lógica/física del acceso
    async solicitarBaja(login) {
      if (!confirm(`¿Confirmas la revocación total del acceso para el usuario ${login}?`)) return;
      
      try {
        const respuesta = await fetch(`${this.apiUrl}/usuarios/${login}?zusuario=${this.zusuario}`, { 
          method: 'DELETE' 
        });
        if (respuesta.ok) this.cargarDatos();
      } catch (error) {
        alert("No se pudo procesar la baja.");
      }
    },

    resetearModelo() {
      this.modeloUsuario = { login: '', password_hash: '', rol_id: '', ref_identidad_fk: '', estado_id: 'Act_a' };
    },

    // Ayudantes visuales
    obtenerEstiloRol(rol) {
      const mapping = {
        'Admin_a': 'rolAdmin',
        'Tic_a': 'rolTic',
        'Prof_a': 'rolProf',
        'Alum_a': 'rolAlum'
      };
      return mapping[rol] || '';
    }
  }
}
</script>

<style scoped>
.panelUsuarios { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.02); }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }

.btnAccion { background: #0f172a; color: white; border: none; padding: 12px 24px; border-radius: 8px; cursor: pointer; font-weight: 600; }

.cardFormulario { background: #f8fafc; padding: 30px; border-radius: 12px; border: 1px solid #e2e8f0; margin-bottom: 35px; }
.gridCampos { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 20px; }

.grupoInput label { display: block; font-size: 11px; font-weight: 800; color: #64748b; margin-bottom: 8px; text-transform: uppercase; }
.gridCampos input, .gridCampos select { width: 100%; padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; box-sizing: border-box; background: white; }

.btnGuardar { 
  background: #2563eb; 
  color: white; 
  border: none; 
  padding: 15px; 
  border-radius: 8px; 
  cursor: pointer; 
  grid-column: 1 / -1; 
  font-weight: bold; 
  margin-top: 10px;
  transition: background 0.2s;
}
.btnGuardar:hover:not(:disabled) { background: #1d4ed8; }
.btnGuardar:disabled { background: #94a3b8; cursor: not-allowed; }

.tablaContainer { overflow-x: auto; }
.tablaUsuarios { width: 100%; border-collapse: collapse; }
.tablaUsuarios th { text-align: left; padding: 15px; background: #f1f5f9; color: #475569; font-size: 12px; text-transform: uppercase; border-bottom: 2px solid #e2e8f0; }
.tablaUsuarios td { padding: 18px 15px; border-bottom: 1px solid #f1f5f9; font-size: 14px; }

.txtLogin { color: #1e293b; font-size: 15px; }
.badgeIdentidad { font-family: 'Courier New', monospace; background: #f1f5f9; padding: 5px 10px; border-radius: 6px; font-weight: bold; color: #475569; border: 1px solid #e2e8f0; }
.txtFecha { color: #94a3b8; }

/* Estilos dinámicos de Perfiles */
.tagRol { padding: 5px 12px; border-radius: 20px; font-size: 11px; font-weight: 800; text-transform: uppercase; }
.rolAdmin { background: #fee2e2; color: #991b1b; }
.rolTic { background: #f3e8ff; color: #6b21a8; }
.rolProf { background: #dcfce7; color: #166534; }
.rolAlum { background: #dbeafe; color: #1e40af; }

.btnEliminar { 
  background: #fff; 
  color: #ef4444; 
  border: 1px solid #fee2e2; 
  padding: 8px 16px; 
  border-radius: 6px; 
  cursor: pointer; 
  font-weight: 700; 
  font-size: 12px;
  transition: all 0.2s; 
}
.btnEliminar:hover { background: #ef4444; color: white; border-color: #ef4444; }

.slide-enter-active, .slide-leave-active { transition: all 0.3s ease; }
.slide-enter { opacity: 0; transform: translateY(-10px); }
</style>