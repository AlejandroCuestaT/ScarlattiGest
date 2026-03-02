<template>
  <div class="loginWrapper">
    <div class="loginCard">
      <div class="loginHeader">
        <h1>Escarlatti-Gest</h1>
        <p class="tagline">Sprint 4 Final • IES Domenico Scarlatti</p>
      </div>

      <div class="formGroup">
        <label>Usuario</label>
        <input 
          v-model="credenciales.login" 
          type="text" 
          placeholder="Admin"
          @keyup.enter="intentarAcceso"
        >
      </div>

      <div class="formGroup">
        <label>Contraseña</label>
        <input 
          v-model="credenciales.password" 
          type="password" 
          placeholder="••••"
          @keyup.enter="intentarAcceso"
        >
      </div>

      <button 
        @click="intentarAcceso" 
        class="btnLogin" 
        :disabled="estaCargando"
      >
        {{ estaCargando ? 'Autenticando...' : 'Iniciar Sesión' }}
      </button>

      <transition name="fade">
        <div v-if="mensajeError" class="errorMsg">
          {{ mensajeError }}
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoginComponent',
  data() {
    return {
      // Estructura de datos requerida por la API
      credenciales: {
        login: '',
        password: '',
        zusuario: 'acuesta'
      },
      // Configuración del servicio
      urlAuth: "http://44.207.19.239:3000/auth/login", 
      mensajeError: null,
      estaCargando: false
    }
  },
  methods: {
    // Gestiona la petición POST de autenticación
    async intentarAcceso() {
      if (!this.credenciales.login || !this.credenciales.password) {
        this.mensajeError = "Por favor, completa todos los campos";
        return;
      }

      this.estaCargando = true;
      this.mensajeError = null;

      try {
        const respuesta = await fetch(this.urlAuth, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.credenciales)
        });

        const datosUsuario = await respuesta.json();

        if (respuesta.ok) {
          // Si el login es correcto, emitimos los datos al componente App.vue
          // Estos incluyen: usuario, rol, nombre y apellidos.
          this.$emit('logueado', datosUsuario);
        } else {
          this.mensajeError = datosUsuario.error || "Credenciales incorrectas";
        }
      } catch (error) {
        this.mensajeError = "Sin conexión con el servicio de autenticación";
      } finally {
        this.estaCargando = false;
      }
    }
  }
}
</script>

<style scoped>
.loginWrapper { 
  height: 100vh; 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  background: radial-gradient(circle, #2c3e50 0%, #000000 100%); 
}

.loginCard { 
  background: white; 
  padding: 40px; 
  border-radius: 12px; 
  width: 100%;
  max-width: 350px; 
  box-shadow: 0 10px 25px rgba(0,0,0,0.5); 
}

.loginHeader { text-align: center; margin-bottom: 30px; }
.loginHeader h1 { color: #1d4ed8; margin: 0; font-size: 28px; letter-spacing: -1px; }
.tagline { color: #94a3b8; font-size: 11px; margin-top: 5px; font-weight: 600; text-transform: uppercase; }

.formGroup { margin-bottom: 20px; }
.formGroup label { display: block; font-size: 13px; font-weight: 600; margin-bottom: 6px; color: #475569; }
.formGroup input { 
  width: 100%; 
  padding: 12px; 
  border: 1px solid #e2e8f0; 
  border-radius: 8px; 
  box-sizing: border-box; 
  font-size: 14px;
  transition: border-color 0.2s;
}
.formGroup input:focus { outline: none; border-color: #3b82f6; }

.btnLogin { 
  width: 100%; 
  padding: 14px; 
  background: #2563eb; 
  color: white; 
  border: none; 
  border-radius: 8px; 
  cursor: pointer; 
  font-weight: bold; 
  font-size: 15px;
  transition: background 0.2s;
}
.btnLogin:hover:not(:disabled) { background: #1d4ed8; }
.btnLogin:disabled { background: #94a3b8; cursor: not-allowed; }

.errorMsg { 
  margin-top: 20px; 
  color: #b91c1c; 
  text-align: center; 
  font-size: 13px; 
  background: #fef2f2; 
  padding: 10px; 
  border-radius: 6px; 
  border: 1px solid #fee2e2;
}

.fade-enter-active { transition: opacity 0.5s; }
.fade-enter { opacity: 0; }
</style>