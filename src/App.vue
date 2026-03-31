<template>
  <div id="app">
    <h1>Registro / Perfil</h1>

    <div class="layout">
      <!-- FORMULARIO -->
      <form @submit.prevent="handleSubmit" novalidate>

        <!-- 1. DATOS BÁSICOS -->
        <fieldset>
          <legend>Datos básicos</legend>

          <div class="field">
            <label for="nombre">Nombre *</label>
            <input
              id="nombre"
              v-model.trim="form.nombre"
              type="text"
              :class="{ error: errors.nombre }"
              placeholder="Tu nombre"
            />
            <span v-if="errors.nombre" class="error-msg">{{ errors.nombre }}</span>
          </div>

          <div class="field">
            <label for="edad">Edad *</label>
            <input
              id="edad"
              v-model.number="form.edad"
              type="number"
              min="0"
              max="120"
              :class="{ error: errors.edad }"
              placeholder="0–120"
            />
            <span v-if="errors.edad" class="error-msg">{{ errors.edad }}</span>
          </div>

          <div class="field">
            <label for="bio">Biografía</label>
            <textarea
              id="bio"
              v-model.lazy="form.biografia"
              rows="4"
              placeholder="Escribe sobre ti…"
            ></textarea>
            <!-- El contador usa la longitud actual del textarea (v-model.lazy actualiza al perder foco) -->
            <small>{{ bioLength }} caracteres</small>
          </div>
        </fieldset>

        <!-- 2. PREFERENCIAS -->
        <fieldset>
          <legend>Preferencias</legend>

          <!-- Radio: nivel -->
          <div class="field">
            <label>Nivel</label>
            <div class="radio-group">
              <label v-for="nv in niveles" :key="nv.value">
                <input
                  type="radio"
                  v-model="form.nivel"
                  :value="nv.value"
                />
                {{ nv.label }}
              </label>
            </div>
          </div>

          <!-- Checkboxes: intereses -->
          <div class="field">
            <label>Intereses * <small>(mínimo 1)</small></label>
            <div class="checkbox-group" :class="{ error: errors.intereses }">
              <label v-for="int in interesesOpciones" :key="int">
                <input
                  type="checkbox"
                  v-model="form.intereses"
                  :value="int"
                />
                {{ int }}
              </label>
            </div>
            <span v-if="errors.intereses" class="error-msg">{{ errors.intereses }}</span>
          </div>

          <!-- Select: país (valor es objeto { code, name }) -->
          <div class="field">
            <label for="pais">País</label>
            <select id="pais" v-model="form.pais">
              <option :value="null" disabled>-- Selecciona --</option>
              <option v-for="p in paises" :key="p.code" :value="p">
                {{ p.name }}
              </option>
            </select>
          </div>

          <!-- Select múltiple: tecnologías -->
          <div class="field">
            <label for="techs">Tecnologías <small>(Ctrl+clic para múltiple)</small></label>
            <select id="techs" v-model="form.tecnologias" multiple size="5">
              <option v-for="t in tecnologiasOpciones" :key="t" :value="t">
                {{ t }}
              </option>
            </select>
          </div>
        </fieldset>

        <button type="submit" :disabled="!formValido">Enviar</button>
      </form>

      <!-- PANEL RESUMEN EN TIEMPO REAL -->
      <aside class="resumen">
        <h2>Vista previa</h2>
        <ul>
          <li><strong>Nombre:</strong> {{ form.nombre || '—' }}</li>
          <li><strong>Edad:</strong> {{ form.edad !== null && form.edad !== '' ? form.edad : '—' }}</li>
          <li><strong>País:</strong> {{ form.pais ? form.pais.name : '—' }}</li>
          <li><strong>Nivel:</strong> {{ form.nivel || '—' }}</li>
          <li><strong>Intereses:</strong> {{ form.intereses.length ? form.intereses.join(', ') : '—' }}</li>
          <li><strong>Tecnologías:</strong> {{ form.tecnologias.length ? form.tecnologias.join(', ') : '—' }}</li>
          <li><strong>Biografía ({{ bioLength }} car.):</strong> {{ form.biografia || '—' }}</li>
        </ul>
      </aside>
    </div>

    <!-- MODAL de confirmación -->
    <div v-if="modalVisible" class="modal-overlay" @click.self="modalVisible = false">
      <div class="modal">
        <h3>✅ Formulario enviado</h3>
        <pre>{{ modalPayload }}</pre>
        <button @click="modalVisible = false">Cerrar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',

  data() {
    return {
      form: {
        nombre: '',
        edad: '',
        biografia: '',
        nivel: 'junior',
        intereses: [],
        pais: null,
        tecnologias: [],
      },

      niveles: [
        { value: 'junior', label: 'Junior' },
        { value: 'semi-senior', label: 'Semi Senior' },
        { value: 'senior', label: 'Senior' },
      ],

      interesesOpciones: ['Frontend', 'Backend', 'DevOps', 'UX/UI', 'Data Science', 'Mobile'],

      // value binding con objetos: el país guardado es { code, name }
      paises: [
        { code: 'AR', name: 'Argentina' },
        { code: 'CL', name: 'Chile' },
        { code: 'CO', name: 'Colombia' },
        { code: 'ES', name: 'España' },
        { code: 'MX', name: 'México' },
        { code: 'PE', name: 'Perú' },
      ],

      tecnologiasOpciones: ['Vue', 'React', 'Angular', 'Svelte', 'Node.js', 'TypeScript'],

      modalVisible: false,
      modalPayload: '',
    }
  },

  computed: {
    bioLength() {
      return this.form.biografia.length
    },

    errors() {
      const e = {}
      if (!this.form.nombre) {
        e.nombre = 'El nombre es requerido.'
      }
      const edad = this.form.edad
      if (edad === '' || edad === null) {
        e.edad = 'La edad es requerida.'
      } else if (edad < 0 || edad > 120) {
        e.edad = 'La edad debe estar entre 0 y 120.'
      }
      if (this.form.intereses.length === 0) {
        e.intereses = 'Selecciona al menos un interés.'
      }
      return e
    },

    formValido() {
      return Object.keys(this.errors).length === 0
    },
  },

  methods: {
    handleSubmit() {
      if (!this.formValido) return
      const payload = {
        nombre: this.form.nombre,
        edad: this.form.edad,
        biografia: this.form.biografia,
        nivel: this.form.nivel,
        intereses: this.form.intereses,
        pais: this.form.pais,
        tecnologias: this.form.tecnologias,
      }
      console.log('Payload:', JSON.stringify(payload, null, 2))
      this.modalPayload = JSON.stringify(payload, null, 2)
      this.modalVisible = true
    },
  },
}
</script>

<style>
* { box-sizing: border-box; }

body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background: #f4f6f9;
  color: #2c3e50;
  margin: 0;
}

#app {
  max-width: 960px;
  margin: 40px auto;
  padding: 0 16px;
}

h1 { text-align: center; margin-bottom: 24px; }

.layout {
  display: flex;
  gap: 24px;
  align-items: flex-start;
}

@media (max-width: 640px) {
  .layout {
    flex-direction: column;
  }

  .resumen {
    flex: 1 1 auto;
    width: 100%;
    position: static;
  }
}

form { flex: 1; }

fieldset {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 16px;
}

legend {
  font-weight: bold;
  padding: 0 8px;
}

.field {
  margin-bottom: 14px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

label { font-weight: 600; font-size: 0.9rem; }

input[type="text"],
input[type="number"],
textarea,
select {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 0.95rem;
  width: 100%;
}

input.error, .checkbox-group.error { border-color: #e74c3c; }

.error-msg { color: #e74c3c; font-size: 0.8rem; }

.radio-group, .checkbox-group {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  padding: 8px;
  border: 1px solid transparent;
  border-radius: 6px;
}

.radio-group label, .checkbox-group label {
  font-weight: normal;
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: pointer;
}

button[type="submit"] {
  width: 100%;
  padding: 10px;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s;
}

button[type="submit"]:disabled {
  background: #aaa;
  cursor: not-allowed;
}

button[type="submit"]:not(:disabled):hover { background: #369f6e; }

/* Resumen */
.resumen {
  flex: 0 0 260px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  position: sticky;
  top: 20px;
}

.resumen h2 { margin-top: 0; font-size: 1.1rem; }

.resumen ul { list-style: none; padding: 0; margin: 0; }

.resumen li { margin-bottom: 8px; font-size: 0.88rem; word-break: break-word; }

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal {
  background: white;
  border-radius: 10px;
  padding: 24px;
  max-width: 480px;
  width: 90%;
}

.modal pre {
  background: #f4f6f9;
  padding: 12px;
  border-radius: 6px;
  font-size: 0.8rem;
  overflow: auto;
  max-height: 300px;
}

.modal button {
  margin-top: 12px;
  padding: 8px 20px;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
</style>
