<template>
  <div class="gestion-container">
    <div class="header-seccion">
      <h2>Gestión de Departamentos</h2>
      <button @click="mostrarForm = !mostrarForm" class="btn-principal">
        {{ mostrarForm ? 'Cerrar formulario' : 'Nuevo Departamento' }}
      </button>
    </div>

    <div v-if="mostrarForm" class="formulario-card">
      <h3>Datos del nuevo departamento</h3>
      <form @submit.prevent="guardarDepartamento">
        <div class="grupo-input">
          <label>Nombre del Departamento:</label>
          <input v-model="nuevoDep.nombre" placeholder="Ej: Matemáticas" required>
        </div>
        <div class="grupo-input">
          <label>Ubicación:</label>
          <input v-model="nuevoDep.ubicacion" placeholder="Ej: Edificio A, planta 1" required>
        </div>
        <div class="grupo-input">
          <label>Correo de contacto:</label>
          <input v-model="nuevoDep.correo_contacto" type="email" placeholder="ejemplo@escarlatti.es" required>
        </div>
        <button type="submit" class="btn-guardar">Guardar Departamento</button>
      </form>
    </div>

    <div class="grid-listado">
      <div v-for="dep in departamentos" :key="dep.id" class="tarjeta">
        <span class="badge-id">ID: {{ dep.id }}</span>
        <h4>{{ dep.nombre }}</h4>
        <p><strong>Ubicación:</strong> {{ dep.ubicacion }}</p>
        <p><strong>Email:</strong> {{ dep.correo_contacto }}</p>
        <button @click="eliminarDepartamento(dep.id)" class="btn-eliminar">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      departamentos: [],
      mostrarForm: false,
      // URL de tu API según la imagen capturada
      url: 'http://100.52.46.68:3000/departamentos',
      nuevoDep: {
        nombre: '',
        ubicacion: '',
        correo_contacto: ''
      }
    }
  },
  mounted() {
    this.cargarDepartamentos();
  },
  methods: {
    async cargarDepartamentos() {
      try {
        const respuesta = await fetch(this.url);
        this.departamentos = await respuesta.json();
      } catch (error) {
        console.error("Error al cargar:", error);
      }
    },
    async guardarDepartamento() {
      const maxId = this.departamentos.length > 0 
        ? Math.max(...this.departamentos.map(d => parseInt(d.id))) 
        : 0;
      
      const objetoAEnviar = {
        id: (maxId + 1).toString(),
        nombre: this.nuevoDep.nombre,
        ubicacion: this.nuevoDep.ubicacion,
        correo_contacto: this.nuevoDep.correo_contacto
      };

      try {
        const respuesta = await fetch(this.url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(objetoAEnviar)
        });

        if (respuesta.ok) {
          this.mostrarForm = false;
          this.nuevoDep = { nombre: '', ubicacion: '', correo_contacto: '' };
          this.cargarDepartamentos();
        } else {
          alert("Error en el servidor al guardar");
        }
      } catch (error) {
        console.error("Error en la petición:", error);
      }
    },
    async eliminarDepartamento(id) {
      if (confirm("¿Estás seguro de eliminar este departamento?")) {
        try {
          await fetch(`${this.url}/${id}`, { method: 'DELETE' });
          this.cargarDepartamentos();
        } catch (error) {
          console.error("Error al eliminar:", error);
        }
      }
    }
  }
}
</script>

<style scoped>
.gestion-container { padding: 20px; }
.header-seccion { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }

.formulario-card { 
  background: #fff; 
  padding: 25px; 
  border: 1px solid #ddd; 
  max-width: 500px; 
  margin-bottom: 30px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}

.grupo-input { margin-bottom: 15px; }
.grupo-input label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 0.9rem; }
.grupo-input input { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; }

.btn-principal { background: #007bff; color: white; border: none; padding: 10px 20px; cursor: pointer; border-radius: 4px; }
.btn-guardar { width: 100%; background: #007bff; color: white; border: none; padding: 12px; cursor: pointer; font-weight: bold; margin-top: 10px; }

.grid-listado { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
.tarjeta { background: #fff; border: 1px solid #eee; border-radius: 8px; padding: 20px; position: relative; transition: transform 0.2s; }
.tarjeta:hover { transform: translateY(-5px); box-shadow: 0 4px 15px rgba(0,0,0,0.1); }

.badge-id { background: #007bff; color: white; padding: 2px 8px; border-radius: 10px; font-size: 0.7rem; position: absolute; top: 15px; left: 15px; }
.tarjeta h4 { margin: 25px 0 10px 0; font-size: 1.2rem; border-bottom: 1px solid #eee; padding-bottom: 10px; }
.tarjeta p { font-size: 0.9rem; color: #555; margin: 5px 0; }

.btn-eliminar { background: #ff4d4d; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; margin-top: 15px; width: 100%; }
</style>