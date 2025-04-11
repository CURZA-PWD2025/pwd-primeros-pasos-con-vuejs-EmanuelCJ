<template>
    <div class="formulario-container">
        <h2 class="titulo">Formulario por pasos</h2>
        <div class="progreso-container">
            <div class="pasos">
                <span v-for="(paso, index) in pasos" :key="index" :class="{ 'colorPaso': contadorPasos >= index }">
                    {{ paso.label }}
                </span>
            </div>
            <div class="barra-container">
                <div class="barra-progreso" :style="{ width: `${(contadorPasos / (pasos.length - 1)) * 100}%` }"></div>
            </div>
        </div>

        <form @submit.prevent="submitForm" class="formulario">
            <div v-show="contadorPasos === 0" class="campo">
                <label for="name">Nombre</label>
                <input type="text" id="name" v-model="Usuario.name" placeholder="Ingresa tu nombre"
                    @keyup.enter="nextPaso" />
                <p v-if="errors.name" class="error">{{ errors.name }}</p>
            </div>

            <div v-show="contadorPasos === 1" class="campo">
                <label for="email">Email</label>
                <input type="email" id="email" v-model="Usuario.email" placeholder="Ingresa tu email"
                    @keyup.enter="nextPaso" />
                <p v-if="errors.email" class="error">{{ errors.email }}</p>
            </div>

            <div v-show="contadorPasos === 2" class="campo">
                <label for="date">Fecha</label>
                <input type="date" id="date" v-model="Usuario.date" @keyup.enter="nextPaso" />
                <p v-if="errors.date" class="error">{{ errors.date }}</p>
            </div>

            <div v-show="contadorPasos === 3" class="campo">
                <label for="password">Contraseña</label>
                <div class="password-container">
                    <input :type="mostrarPassword ? 'text' : 'password'" id="password" v-model="Usuario.password"
                        placeholder="Ingresa tu contraseña" @keyup.enter="nextPaso" />
                    <button class="boton" type="button" @click="mostrarPassword = !mostrarPassword">
                        {{ mostrarPassword ? 'Ocultar' : 'Mostrar' }}
                    </button>
                </div>
                <p v-if="errors.password" class="error">{{ errors.password }}</p>
            </div>

            <div class="botones">
                <button v-if="contadorPasos > 0" type="button" @click="prevPaso" class="boton">Anterior</button>
                <button v-if="contadorPasos < pasos.length - 1" type="button" @click="nextPaso"
                    class="boton">Siguiente</button>
                <button v-else type="submit" class="boton boton-enviar">Enviar</button>
            </div>
        </form>

        <div v-if="enviado" class="resumen">
            <h3>¡Formulario enviado con éxito!</h3>
            <p><strong>Nombre: </strong> {{ Usuario.name }}</p>
            <p><strong>Email: </strong> {{ Usuario.email }}</p>
            <p><strong>Fecha: </strong> {{ Usuario.date }}</p>
            <p><strong>Años: </strong>{{ calcularEdad() }}</p>
            <p><strong>Contraseña: </strong>{{ Usuario.password }}</p>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';


// Definición de interfaces para los pasos y los datos del formulario
interface Paso {
    label: string;
    campo: keyof Errors;
}

interface FormData {
    name: string;
    email: string;
    date: string;
    password: string;
    edad: number;
}

interface Errors {
    name: string;
    email: string;
    date: string;
    password: string;
}

const pasos: Paso[] = [
    { label: 'Nombre', campo: 'name' },
    { label: 'Email', campo: 'email' },
    { label: 'Fecha', campo: 'date' },
    { label: 'Contraseña', campo: 'password' }
];

const contadorPasos = ref<number>(0);
const mostrarPassword = ref<boolean>(false);
const enviado = ref<boolean>(false);

const Usuario = reactive<FormData>({
    name: '',
    email: '',
    date: '',
    password: '',
    edad: 0
});

//Calcular edad a partir de la fecha de nacimiento 
function calcularEdad(): number {
    const hoy = new Date();
    const fechaNacimiento = new Date(Usuario.date);
    let edad = hoy.getFullYear() - fechaNacimiento.getFullYear();
    const mes = hoy.getMonth() - fechaNacimiento.getMonth();

    if (mes < 0 || (mes === 0 && hoy.getDate() < fechaNacimiento.getDate())) {
        edad--;
    }
    
    Usuario.edad = edad;
    return edad;
}

const errors = reactive<Errors>({
    name: '',
    email: '',
    date: '',
    password: ''
});

const validarCampo = (campo: keyof Errors): boolean => {
    errors[campo] = '';
    switch (campo) {
        case 'name':
            if (!Usuario.name.trim()) {
                errors.name = 'El nombre es obligatorio';
                return false;
            }
            break;
        case 'email':
            if (!Usuario.email.trim()) {
                errors.email = 'El email es obligatorio';
                return false;
            }
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test(Usuario.email)) {
                errors.email = 'Ingresa un email válido';
                return false;
            }
            break;
        case 'date':
            if (!Usuario.date) {
                errors.date = 'La fecha es obligatoria';
                return false;
            }
            break;
        case 'password':
            if (!Usuario.password) {
                errors.password = 'La contraseña es obligatoria';
                return false;
            }
            if (Usuario.password.length < 6) {
                errors.password = 'La contraseña debe tener al menos 6 caracteres';
                return false;
            }
            break;
    }
    return true;
};

const nextPaso = () => {
    const campo = pasos[contadorPasos.value].campo;
    if (validarCampo(campo) && contadorPasos.value < pasos.length - 1) {
        contadorPasos.value++;
    }
};

const prevPaso = () => {
    if (contadorPasos.value > 0) {
        contadorPasos.value--;
    }
};

const submitForm = () => {
    const campo = pasos[contadorPasos.value].campo;
    if (validarCampo(campo)) {
        //falta crear usuario
        console.log('Formulario enviado:', Usuario);
        enviado.value = true;
    }
};

</script>

<style scoped>
.formulario-container {
    max-width: 500px;
    margin: auto;
    padding: 20px;
    border-radius: 8px;
    background: #f9f9f9;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.titulo {
    text-align: center;
}

.pasos {
    display: flex;
    justify-content: space-between;
    font-weight: bold;

}

.colorPaso {
    color: #4CAF50;
    text-decoration: underline;
}

.barra-container {
    height: 10px;
    background: #ddd;
    border-radius: 5px;
    margin: 15px 0 15px 0;
}

.barra-progreso {
    height: 10px;
    background: #4CAF50;
    border-radius: 5px;
}

.boton {
    margin: 10px;
}

.campo {
    display: flex;
    flex-direction: column;
    margin-bottom: 15px;
}

.campo label {
    font-weight: bold;
    margin-bottom: 5px;
}

.campo input {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 16px;
}

.campo input:focus {
    border-color: #4CAF50;
    outline: none;
    box-shadow: 0 0 5px rgba(76, 175, 80, 0.5);
}

.botones {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
}

.boton {
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    background-color: #4CAF50;
    color: white;
    font-size: 16px;
}

.boton:hover {
    background-color: #45a049;
}

.boton-enviar {
    background-color: #008CBA;
}
</style>
