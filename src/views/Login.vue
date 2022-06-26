<template>
<p style="margin-bottom: 30px; text-align: center; font-size:x-large;">Welcome!</p>
  <div v-if="!alreadyLoggedIn" class="login-form">
    <form @submit="logIn">
      <input
        v-model="emailInput"
        type="text"
        name="email"
        required
        placeholder="Email"
      />
      <input
        v-model="passwordInput"
        type="password"
        name="password"
        required
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  </div>
  <div v-else class="already-logged-in">
    <p>Already logged in, redirecting...</p>
  </div>
  <p
    v-if="correctCredentials != null && !correctCredentials"
    class="incorrect-credentials"
  >
    Email or password is incorrect.
  </p>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import { store } from "@/main";
import axios from "axios";

const router = useRouter();

const emailInput = ref();
const passwordInput = ref();
const correctCredentials = ref(null);
const alreadyLoggedIn = ref(false);

const logIn = async (e) => {
  e.preventDefault();

  const data = {
    email: emailInput.value,
    password: passwordInput.value,
  };

  try {
    const res = await axios.post("https://api.bajmovies.xyz/user/login", data);
    correctCredentials.value = true;
    store.user = res.data;
    console.log(store.user);
    localStorage.setItem("user", JSON.stringify(store.user));
    router.push({ name: "Home" });
  } catch (err) {
    console.log(err);
    if (err) correctCredentials.value = false;
  }
};

onMounted(() => {
  console.log(store.user);
  if (store.user != null) alreadyLoggedIn.value = true;
  else alreadyLoggedIn.value = false;

  if (alreadyLoggedIn.value) {
    setTimeout(() => {
      router.push({
        name: "Home",
      });
    }, 2000);
  }
});
</script>

<style lang="scss" scoped>
.incorrect-credentials {
  text-align: center;
  color: crimson;
}
.login-form {
  display: flex;
  justify-content: center;

  form {
    width: 400px;
    display: flex;
    flex-direction: column;
    align-items: center;

    input {
      width: 100%;
      padding: 12px;
      margin-bottom: 20px;
      border-radius: 100px;
      border: none;
    }

    button {
      padding: 13px 0px;
      width: 100px;

      background: none;
      color: white;
      border: 2px solid white;
      border-radius: 100px;
      transition: 0.2s ease-out;

      &:hover {
        cursor: pointer;
        color: rgb(27, 27, 27);
        background-color: white;
      }
    }
  }
}
</style>
