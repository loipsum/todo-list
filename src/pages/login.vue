<template>
  <q-page class="flex flex-center" padding>
    <q-form @submit="login" class="q-gutter-md">
      <div>
        <q-btn color="primary" icon="check" label="OK" type="submit" />
      </div>
    </q-form>
  </q-page>
</template>

<script>
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter();
const api = "http://localhost:8000/api/";
export default {
  setup() {
    return {
      login,
      register,
    };
  },
};

async function login() {
  console.log("asdf");
  try {
    const test = await axios.post("http://localhost:8000/api/login", {
      email: "user1@gmail.com",
      password: "password",
    });

    localStorage.setItem("token", test.data.token);
    router.push("/");
    console.log("after push");
  } catch (error) {
    console.log(error.response);
  }
}

function register() {
  try {
    const res = axios.post(api + "register", {
      email: "test@test",
      name: "testname",
      password: "password",
    });
  } catch (error) {
    console.log(error);
  }
}
</script>
