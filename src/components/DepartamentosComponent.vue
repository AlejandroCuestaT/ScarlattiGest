<template>
  <div class="dataContainer">
    <div class="sectionHeader">
      <h2>Gestión de Departamentos</h2>
      
      <div class="quickAdd">
        <input v-model="nuevoDepartamento.id" placeholder="ID (ej: 25)" maxlength="10">
        <input v-model="nuevoDepartamento.nombre" placeholder="Nombre (ej: Base de datos)">
        <button @click="crearDepartamento" class="btnAdd">Añadir Departamento</button>
      </div>
    </div>

    <table class="dataTable">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre del Departamento</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="departamento in listaDepartamentos" :key="departamento.id">
          <td><span class="idBadge">{{ departamento.id }}</span></td>
          <td>{{ departamento.nombre }}</td>
          <td>
            <button @click="eliminarDepartamento(departamento.id)" class="btnDel">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'DepartamentosComponent',
  data() {
    return {
      // Lista de departamentos que recuperamos del servidor
      listaDepartamentos: [],
      // Objeto reactivo para el alta de nuevos departamentos
      nuevoDepartamento: { id: '', nombre: '' },
      // Configuracion de conexión unificada con el proyecto
      apiUrl: 'http://44.207.19.239:3000/departamentos',
      zusuario: 'acuesta'
    }
  },
  mounted() {
    // Carga inicial de datos al mostrar el componente
    this.cargarDepartamentos();
  },
  methods: {
    // Obtiene todos los departamentos registrados bajo el usuario activo
    async cargarDepartamentos() {
      try {
        // Incluimos el parámetro zusuario para la auditoría de la API
        const respuesta = await fetch(`${this.apiUrl}?zusuario=${this.zusuario}`);
        if (!respuesta.ok) throw new Error("Fallo en la comunicación con el servidor");
        
        this.listaDepartamentos = await respuesta.json();
      } catch (error) {
        console.error("Error al conectar con la API de departamentos:", error);
      }
    },

    // Envía un nuevo departamento a la base de datos
    async crearDepartamento() {
      // Validación simple para asegurar que los campos no viajen vacíos
      if (!this.nuevoDepartamento.id || !this.nuevoDepartamento.nombre) {
        return alert("Por favor, rellena el ID y el nombre antes de continuar");
      }

      try {
        const respuesta = await fetch(`${this.apiUrl}?zusuario=${this.zusuario}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            ...this.nuevoDepartamento, // Esparcimos las propiedades del objeto nuevo
            zusuario: this.zusuario,
            zfecha: new Date().toISOString()
          })
        });

        if (respuesta.ok) {
          this.cargarDepartamentos(); // Refrescamos la lista para ver el cambio
          this.resetFormulario();     // Limpiamos los campos del input
        }
      } catch (error) {
        alert("Error de red al intentar crear el departamento");
      }
    },

    // Elimina un departamento mediante su ID
    async eliminarDepartamento(id) {
      if (!confirm(`¿Estás seguro de que quieres borrar el departamento ${id}?`)) return;

      try {
        // Construimos la URL con el ID del recurso y el zusuario
        const urlEliminar = `${this.apiUrl}/${id}?zusuario=${this.zusuario}`;
        const respuesta = await fetch(urlEliminar, { method: 'DELETE' });
        
        if (respuesta.ok) {
          this.cargarDepartamentos(); // Actualizamos la vista tras borrar
        } else {
          alert("No se pudo eliminar. Es posible que el departamento tenga datos asociados.");
        }
      } catch (error) {
        console.error("Error en la petición DELETE", error);
      }
    },

    // Limpia el estado del objeto nuevoDepartamento
    resetFormulario() {
      this.nuevoDepartamento = { id: '', nombre: '' };
    }
  }
}
</script>

<style scoped>
.dataContainer { 
  background: white; 
  padding: 30px; 
  border-radius: 12px; 
  box-shadow: 0 4px 6px rgba(0,0,0,0.05); 
}

.sectionHeader { 
  display: flex; 
  justify-content: space-between; 
  align-items: center; 
  margin-bottom: 30px; 
}

.quickAdd { 
  display: flex; 
  gap: 10px; 
}

.quickAdd input { 
  padding: 10px; 
  border: 1px solid #ddd; 
  border-radius: 6px; 
  outline: none; 
}

.quickAdd input:focus { 
  border-color: #007bff; 
}

.btnAdd { 
  background: #34a853; 
  color: white; 
  border: none; 
  padding: 10px 20px; 
  border-radius: 6px; 
  cursor: pointer; 
  font-weight: bold; 
}

.dataTable { 
  width: 100%; 
  border-collapse: collapse; 
}

.dataTable th { 
  text-align: left; 
  padding: 15px; 
  background: #f8f9fa; 
  border-bottom: 2px solid #eee; 
  color: #555;
}

.dataTable td { 
  padding: 15px; 
  border-bottom: 1px solid #f0f0f0; 
}

.idBadge {
  font-family: monospace;
  background: #e9ecef;
  padding: 4px 8px;
  border-radius: 4px;
  font-weight: bold;
}

.btnDel { 
  background: #fff1f0; 
  color: #ff4d4f; 
  border: 1px solid #ffccc7; 
  padding: 6px 12px; 
  border-radius: 4px; 
  cursor: pointer; 
  transition: all 0.2s;
}

.btnDel:hover { 
  background: #ff4d4f; 
  color: white; 
}
</style>