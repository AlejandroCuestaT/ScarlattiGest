<template>
  <div class="login-wrapper">
    <div class="login-card">
      <h2>Escarlatti-Gest</h2>
      <p>Inicia sesión con tu cuenta de Administrador</p>
      <form @submit.prevent="login">
        <input v-model="usuario" placeholder="Usuario" required>
        <input v-model="password" type="password" placeholder="Contraseña" required>
        <button type="submit">Entrar al Sistema</button>
      </form>
      <p v-if="error" class="error-msg">{{ error }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() { return { usuario: '', password: '', error: '' } },
  methods: {
    async login() {
      try {
        const res = await fetch('http://100.27.173.196:3000/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            login: this.usuario,
            password: this.password, // Corregido según error de API
            zusuario: 'acuesta'
          })
        });
        const data = await res.json();
        if (res.ok) {
          this.$emit('logueado', { login: data.login, rol_id: data.rol_id });
        } else { this.error = data.error; }
      } catch (e) { this.error = "No hay conexión con el servidor"; }
    }
  }
}
</script>

<style scoped>
.login-wrapper { height: 80vh; display: flex; justify-content: center; align-items: center; }
.login-card { background: white; padding: 40px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); width: 320px; text-align: center; }
input { width: 100%; padding: 12px; margin-bottom: 15px; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
button { width: 100%; padding: 12px; background: #1a1a1a; color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; }
.error-msg { color: #ff4d4f; font-size: 13px; margin-top: 15px; font-weight: bold; }
</style>