<template>
  <div class="panelRoles">
    <div class="headerSeccion">
      <h2>Definición de Roles y Permisos</h2>
      <button @click="alternarFormulario" class="btnAccion">
        {{ formularioVisible ? 'Cerrar Registro' : 'Crear Nuevo Perfil' }}
      </button>
    </div>

    <transition name="slide">
      <div v-if="formularioVisible" class="cardRegistro">
        <h3>Configurar Nuevo Perfil de Acceso</h3>
        <form @submit.prevent="ejecutarGuardado">
          <div class="gridCampos">
            <div class="grupoInput">
              <label>ID</label>
              <input v-model="modeloRol.id" placeholder="Ej: Secret_a" required>
            </div>
            <div class="grupoInput">
              <label>Nombre</label>
              <input v-model="modeloRol.nombre" placeholder="Ej: Secretario" required>
            </div>
            <div class="grupoInput">
              <label>Privilegio (1-10)</label>
              <input v-model.number="modeloRol.nivel_privilegio" type="number" min="1" max="10" required>
            </div>
            <div class="grupoInput">
              <label>Descripcion</label>
              <input v-model="modeloRol.descripcion" placeholder="Que rol hace" required>
            </div>
          </div>
          <button type="submit" class="btnGuardar">Sincronizar Privilegios</button>
        </form>
      </div>
    </transition>

    <div class="gridRoles">
      <div v-for="rol in coleccionRoles" :key="rol.id" class="tarjetaRol">
        <div class="rolMeta">
          <span class="badgePrivilegio">Prioridad {{ rol.nivel_privilegio }}</span>
          <code class="idSistema">#{{ rol.id }}</code>
        </div>
        
        <h4 class="rolTitulo">{{ rol.nombre }}</h4>
        <p class="rolInfo">{{ rol.descripcion }}</p>
        
        <div class="rolFooter">
          <button @click="solicitarEliminacion(rol.id)" class="btnBorrar">
            Revocar Permisos
          </button>
        </div>
      </div>
    </div>

    <div v-if="coleccionRoles.length === 0" class="estadoVacio">
      Cargando configuración de seguridad...
    </div>
  </div>
</template>

<script>
export default {
  name: 'RolesComponent',
  data() {
    return {
      // Configuración de servicio
      urlBase: 'http://44.207.19.239:3000/roles',
      zusuario: 'acuesta',
      // Estado reactivo
      coleccionRoles: [],                
      formularioVisible: false,
      // Modelo de datos para nuevos registros
      modeloRol: { 
        id: '', 
        nombre: '', 
        nivel_privilegio: 1, 
        descripcion: '' 
      }
    }
  },
  mounted() {
    this.sincronizarRoles();
  },
  methods: {
    // Obtención de datos maestros de la API
    async sincronizarRoles() {
      try {
        const respuesta = await fetch(`${this.urlBase}?zusuario=${this.zusuario}`);
        if (!respuesta.ok) throw new Error("Fallo en la comunicación con el servidor");
        this.coleccionRoles = await respuesta.json();
      } catch (error) {
        console.error('Error de sincronización:', error);
      }
    },

    alternarFormulario() {
      this.formularioVisible = !this.formularioVisible;
      if (!this.formularioVisible) this.resetearModelo();
    },

    // Envío de nuevo rol con metadatos de auditoría
    async ejecutarGuardado() {
      const payload = { 
        ...this.modeloRol, 
        zusuario: this.zusuario,
        zfecha: new Date().toISOString()
      };

      try {
        const respuesta = await fetch(`${this.urlBase}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        if (respuesta.ok) {
          this.formularioVisible = false;
          this.resetearModelo();
          this.sincronizarRoles();
        } else {
          alert("Error: El Identificador ya existe o el formato no es válido.");
        }
      } catch (error) {
        console.error('Error al guardar el perfil:', error);
      }
    },

    // Borrado físico del registro (requiere confirmación)
    async solicitarEliminacion(id) {
      if (!confirm('¿Estás seguro de eliminar este perfil de acceso? Los usuarios vinculados podrían perder acceso al sistema.')) return;

      const urlFinal = `${this.urlBase}/${id}?zusuario=${this.zusuario}`;

      try {
        const respuesta = await fetch(urlFinal, { method: 'DELETE' });
        if (respuesta.ok) {
          this.sincronizarRoles();
        } else {
          alert("Acción denegada: Este rol podría estar protegido o vinculado a usuarios activos.");
        }
      } catch (error) {
        console.error('Error en el proceso de borrado:', error);
      }
    },

    resetearModelo() {
      this.modeloRol = { id: '', nombre: '', nivel_privilegio: 1, descripcion: '' };
    }
  }
}
</script>

<style scoped>
.panelRoles { padding: 30px; background: #f8fafc; border-radius: 15px; }
.headerSeccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }

.btnAccion { 
  background: #4f46e5; 
  color: white; 
  border: none; 
  padding: 12px 24px; 
  border-radius: 10px; 
  cursor: pointer; 
  font-weight: 600; 
  transition: transform 0.2s;
}
.btnAccion:hover { transform: translateY(-2px); background: #4338ca; }

.cardRegistro { 
  background: white; 
  padding: 25px; 
  border-radius: 12px; 
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); 
  margin-bottom: 40px; 
  border: 1px solid #e2e8f0; 
}

.gridCampos { display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; margin-bottom: 20px; }
.grupoInput label { display: block; font-size: 12px; font-weight: 700; color: #64748b; margin-bottom: 8px; text-transform: uppercase; }
.grupoInput input { width: 100%; padding: 12px; border: 1px solid #cbd5e1; border-radius: 8px; box-sizing: border-box; }

.btnGuardar { background: #10b981; color: white; border: none; padding: 14px; width: 100%; border-radius: 8px; font-weight: 700; cursor: pointer; }

.gridRoles { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 25px; }
.tarjetaRol { 
  background: white; 
  border-radius: 12px; 
  padding: 25px; 
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); 
  border-top: 5px solid #6366f1; 
  display: flex;
  flex-direction: column;
}

.rolMeta { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 15px; }
.badgePrivilegio { background: #eef2ff; color: #4f46e5; font-size: 11px; font-weight: 800; padding: 4px 10px; border-radius: 6px; }
.idSistema { color: #94a3b8; font-size: 12px; font-weight: 500; }

.rolTitulo { margin: 10px 0; color: #1e293b; font-size: 18px; }
.rolInfo { font-size: 14px; color: #64748b; line-height: 1.5; flex-grow: 1; margin-bottom: 20px; }

.rolFooter { border-top: 1px solid #f1f5f9; padding-top: 15px; display: flex; justify-content: flex-end; }
.btnBorrar { background: none; border: none; color: #f43f5e; cursor: pointer; font-size: 13px; font-weight: 700; transition: color 0.2s; }
.btnBorrar:hover { color: #be123c; text-decoration: underline; }
.estadoVacio { text-align: center; color: #94a3b8; padding: 50px; font-style: italic; }

/* Animación de apertura */
.slide-enter-active, .slide-leave-active { transition: all 0.3s ease; }
.slide-enter, .slide-leave-to { opacity: 0; transform: translateY(-20px); }
</style>