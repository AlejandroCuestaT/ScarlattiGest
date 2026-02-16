<template>
  <div class="login-box">
    <div class="login-card">
      <h2>Acceso al Sistema</h2>
      <p>Introduce tu usuario y contraseña</p>
      <div v-if="error" class="alerta-error">{{ error }}</div>
      <form @submit.prevent="login">
        <input v-model="usuarioTxt" placeholder="Nombre de usuario (login)" required>
        <input v-model="passwordTxt" type="password" placeholder="Contraseña" required>
        <button type="submit" class="btn-entrar">Acceder</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      usuarioTxt: '',
      passwordTxt: '',
      error: '',
      urlUsuarios: 'http://100.52.46.68:3000/usuarios'
    }
  },
  methods: {
    async login() {
      try {
        let res = await fetch(this.urlUsuarios);
        let lista = await res.json();
        let encontrado = lista.find(u => u.login === this.usuarioTxt && u.password_hash === this.passwordTxt);
        if (encontrado) { this.$emit('logueado', encontrado); } 
        else { this.error = 'Usuario o clave incorrectos'; }
      } catch (e) { this.error = 'Error conectando con el servidor'; }
    }
  }
}
</script>

<style scoped>
.login-box { display: flex; justify-content: center; padding-top: 50px; }
.login-card { background: white; padding: 20px; border: 1px solid #ccc; width: 300px; text-align: center; }
input { display: block; width: 100%; padding: 10px; margin-bottom: 10px; box-sizing: border-box; }
.btn-entrar { width: 100%; padding: 10px; background: #007bff; color: white; border: none; cursor: pointer; }
.alerta-error { padding: 10px; background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; margin-bottom: 10px; font-size: 14px; }
</style>