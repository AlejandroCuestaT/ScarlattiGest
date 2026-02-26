<template>
  <div class="login-wrapper">
    <div class="login-card">
      <div class="login-header">
        <h1>Escarlatti-Gest</h1>
        <p>Sprint 4 Final - IES Domenico Scarlatti</p>
      </div>

      <div class="form-group">
        <label>Usuario</label>
        <input v-model="credenciales.login" type="text" placeholder="Admin">
      </div>

      <div class="form-group">
        <label>Contraseña</label>
        <input v-model="credenciales.password" type="password" placeholder="••••">
      </div>

      <button @click="acceder" class="btn-login" :disabled="cargando">
        {{ cargando ? 'Autenticando...' : 'Entrar' }}
      </button>

      <div v-if="error" class="error-msg">{{ error }}</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      credenciales: {
        login: '',
        password: '',
        zusuario: 'acuesta'
      },
      url: "http://44.207.19.239:3000/auth/login", 
      error: null,
      cargando: false
    }
  },
  methods: {
    async acceder() {
      this.cargando = true;
      this.error = null;
      try {
        const res = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.credenciales)
        });

        const data = await res.json();

        if (res.ok) {
          // Emitimos el objeto que devuelve la API (usuario, rol, nombre, apellidos)
          this.$emit('logueado', data);
        } else {
          this.error = data.error || "Error en el acceso";
        }
      } catch (e) {
        this.error = "Error de conexión con la API";
      } finally {
        this.cargando = false;
      }
    }
  }
}
</script>

<style scoped>
.login-wrapper { height: 100vh; display: flex; align-items: center; justify-content: center; background: #1a1a1a; }
.login-card { background: white; padding: 40px; border-radius: 8px; width: 320px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); }
.login-header h1 { color: #1877f2; text-align: center; margin-bottom: 5px; }
.login-header p { text-align: center; color: #666; font-size: 12px; margin-bottom: 25px; }
.form-group { margin-bottom: 15px; }
.form-group label { display: block; font-size: 14px; margin-bottom: 5px; }
.form-group input { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
.btn-login { width: 100%; padding: 12px; background: #1877f2; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; }
.error-msg { margin-top: 15px; color: #ff4d4f; text-align: center; font-size: 13px; background: #fff1f0; padding: 8px; border-radius: 4px; }
</style>