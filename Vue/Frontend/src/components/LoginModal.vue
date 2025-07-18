<script setup>
import { ref, onMounted, watch } from "vue";
import { useRouter } from "vue-router";
import { useAuthStore } from "@/store/AuthStore";
import Swal from "sweetalert2";

const router = useRouter();
const authStore = useAuthStore();
const email = ref("");
const password = ref("");
const validEmail = ref(true);
const validPassword = ref(true);
const validData = ref(true);
watch(email, (newEmail) => {
  validEmail.value = !!newEmail.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
});
watch(password, (newPassword) => {
  validPassword.value = newPassword.length >= 1;
});

function getCssVar(name) {
  return getComputedStyle(document.documentElement)
    .getPropertyValue(name)
    .trim();
}

const onLogin = async (event) => {
  event.preventDefault();
  try {
    const response = await fetch(
      `${import.meta.env.VITE_BACKEND_URL}/Users/login/`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          email_user: email.value,
          password_user: password.value,
        }),
      }
    );

    const data = await response.json();

    if (response.ok) {
      authStore.setAuthData({
        user: data.user,
        token: data.token,
      });
      Swal.fire({
        icon: "success",
        title: "¡Inicio de sesión exitoso!",
        background: getCssVar("--color-primary"),
        color: getCssVar("--color-secondary"),
        showConfirmButton: false,
        timer: 1500,
        customClass: {
          popup: "rounded-xl",
          title: "font-bold",
          htmlContainer: "font-primary", // Usa tu fuente principal
        },
      });
      validData.value = true;
      onToggleLogin();
      router.push("/admin");
    } else {
      validData.value = false;
      Swal.fire({
        icon: "error",
        title: "Error",
        text: "Usuario o contraseña incorrectos",
        background: getCssVar("--color-primary"),
        color: getCssVar("--color-tertiary"),
        showConfirmButton: true,
        customClass: {
          popup: "rounded-xl",
          title: "font-bold",
          htmlContainer: "font-primary",
        },
      });
      console.error("Error al iniciar sesión:", data.message);
    }
  } catch (error) {
    validData.value = false;
    Swal.fire({
      icon: "error",
      title: "Error",
      text: "Ocurrió un error al intentar iniciar sesión",
      background: getCssVar("--color-primary"),
      color: getCssVar("--color-tertiary"),
      showConfirmButton: true,
      customClass: {
        popup: "rounded-xl",
        title: "font-bold",
        htmlContainer: "font-primary",
      },
    });
    console.error("Error al iniciar sesión:", error);
  }
};
const onToggleLogin = () => {
  const modal = document.getElementById("loginModal");
  modal.classList.toggle("hidden");
};
onMounted(() => {
  window.addEventListener("click", function (event) {
    const modal = document.getElementById("loginModal");
    if (event.target === modal) {
      modal.classList.add("hidden");
    }
  });
});
const onInput = () => {
  validData.value = true;
};
</script>

<template>
  <div
    id="loginModal"
    class="hidden fixed inset-0 bg-quinary/85 z-50 flex items-center justify-center font-primary"
  >
    <div class="bg-quaternary p-8 rounded-lg shadow-lg w-11/12 max-w-md">
      <div class="flex justify-between items-center mb-6">
        <h2 class="text-2xl font-bold text-secondary text-subtitle">
          Acceso Administrador
        </h2>
        <button
          @click="onToggleLogin"
          class="text-quinary hover:text-secondary cursor-pointer"
        >
          <i class="fas fa-times"></i>
        </button>
      </div>
      <form>
        <div class="mb-4">
          <label
            class="block text-quinary text-sm font-bold mb-1.5 text-paragraph"
            >Correo</label
          >
          <input
            type="text"
            v-model="email"
            @input="onInput"
            class="w-full px-3 py-2 border border-quinary/25 rounded-md focus:outline-none focus:border-secondary bg-primary/50 text-quinary"
          />
          <span
            v-if="!validEmail"
            class="text-quinary text-paragraph pt-1 block"
            >Correo electrónico inválido</span
          >
        </div>
        <div class="mb-6">
          <label
            class="block text-quinary text-sm font-bold mb-1.5 text-paragraph"
            >Contraseña</label
          >
          <input
            type="password"
            v-model="password"
            @input="onInput"
            class="w-full px-3 py-2 border border-quinary/25 rounded-md focus:outline-none focus:border-secondary bg-primary/50 text-quinary"
          />
          <span
            v-if="!validPassword"
            class="text-quinary text-paragraph pt-1 block"
            >Contraseña inválida</span
          >
        </div>
        <div class="flex justify-between flex-wrap">
          <button
            type="button"
            @click="onToggleLogin"
            class="bg-quinary/70 text-quaternary px-4 py-2 rounded-md hover:bg-quinary/50 cursor-pointer"
          >
            Cancelar
          </button>
          <button
            type="submit"
            @click="onLogin"
            class="bg-secondary text-quaternary px-4 py-2 rounded-md hover:bg-secondary/75 cursor-pointer"
          >
            Ingresar
          </button>
          <span
            v-if="!validData"
            class="text-quinary text-paragraph pt-1 block w-full text-center mt-0.5"
            >Usuario invalido</span
          >
        </div>
      </form>
    </div>
  </div>
</template>
