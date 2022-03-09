<template>
  <q-page>
    <div class="row">
      <h3 class="full-width text-center q-my-md">Todo List</h3>
      <div class="col-4 offset-4">
        <q-input
          dense
          @keydown.enter="addTask"
          outlined
          class="q-mb-lg"
          v-model="todo"
          type="text"
        >
          <template v-slot:append>
            <q-btn
              unelevated
              flat
              dense
              round
              color="primary"
              icon="assignment"
              @click="addTask"
            />
          </template>
        </q-input>
      </div>
      <q-card
        class="col-4 offset-4 q-my-sm"
        style="position: relative"
        v-for="(item, index) in items"
        :key="index"
      >
        <div class="row">
          <q-input
            autofocus
            @focus="$event.target.select()"
            class="col-10"
            input-class="q-py-none"
            v-if="indexedit == index"
            borderless
            color="white"
            v-model="editval"
            type="text"
            input-style="margin-left: 20px; font-size:24px; word-wrap:normal"
          />
          <div
            v-else
            class="col-10"
            :style="
              item.status == 'pending'
                ? ' font-size: 24px; margin-top: 2px; padding-left: 20px'
                : 'text-decoration-line:line-through; text-decoration-color:orange;font-size: 24px; margin-top: 2px; padding-left: 20px'
            "
          >
            {{ item.title }}
          </div>

          <div class="flex flex-center col-2 q-mt-xs q-pr-md">
            <!-- <q-icon size="sm" name="edit" />
            <q-icon size="sm" name="done" /> -->
            <q-btn
              unelevated
              flat
              dense
              round
              color="secondary"
              :icon="indexedit != index ? 'edit' : 'done'"
              @click="editTask(item, index)"
            />
            <q-btn
              v-if="indexedit == index"
              unelevated
              flat
              dense
              round
              color="secondary"
              icon="close"
              @click="indexedit = null"
            />
            <q-btn
              v-else
              style="font-weight: 500"
              unelevated
              dense
              round
              flat
              :color="item.status == 'pending' ? 'positive' : 'red'"
              :icon="
                item.status == 'pending'
                  ? 'mdi-check-bold'
                  : 'mdi-delete-variant'
              "
              @click="doneTask(item, index)"
            />
          </div>
        </div>
      </q-card>
    </div>
    <q-btn color="red" icon="check" label="OK" @click="logout" />
  </q-page>
</template>

<script>
import { ref } from "@vue/reactivity";
import axios from "axios";

const token = localStorage.getItem("token");
const myheader = {
  authorization: "Bearer " + token,
};
const items = ref([]);
const todo = ref(null);
const editval = ref(null);
const indexedit = ref(null);

export default {
  setup() {
    return {
      items,
      todo,
      addTask,
      editTask,
      doneTask,
      editval,
      logout,
      indexedit,
    };
  },
};
const api = "http://localhost:8000/api/";
async function login() {
  const test = await axios.post("http://localhost:8000/api/login", {
    email: "user1@gmail.com",
    password: "password",
  });

  localStorage.setItem("token", test.data.token);
}

async function logout() {
  try {
    const log = await axios.post(
      "http://localhost:8000/api/logout",
      {},
      {
        headers: {
          authorization: "Bearer " + token,
        },
      }
    );
  } catch (error) {
    console.log("erorrr");
  }
}
getItems();
async function getItems() {
  const res = await axios.get(api + "todos", {
    headers: {
      authorization: "Bearer " + token,
    },
  });
  console.log(res.data);
  items.value = res.data;
  return res.data;
}

async function addTask() {
  if (!todo.value) return;
  // items.value.push(todo.value);

  try {
    const res = await axios.post(
      api + "todos",
      {
        title: todo.value,
        status: "pending",
      },
      {
        headers: {
          authorization: "Bearer " + token,
        },
      }
    );

    console.log(res.data);

    items.value = await getItems();
  } catch (error) {
    console.log(error.response);
  }

  todo.value = null;
}
async function editTask(item, index) {
  if (indexedit.value == index) {
    try {
      const res = await axios.put(
        api + "todos/" + item.id,
        {
          title: editval.value,
        },
        { headers: myheader }
      );

      console.log(res.data);

      items.value = await getItems();
    } catch (error) {}

    indexedit.value = null;
    return;
  }
  editval.value = item.title;
  indexedit.value = index;
}

async function doneTask(item, index) {
  if (item.status == "pending") {
    //set item status to done
    try {
      const res = await axios.put(
        api + "todos-status/" + item.id,
        { status: "done" },
        {
          headers: myheader,
        }
      );
      items.value = await getItems();
    } catch (error) {
      console.log(error.response);
    }
  }

  if (item.status == "done") {
    //delete item from database

    try {
      const res = await axios.delete(api + "todos/" + item.id, {
        headers: {
          authorization: "Bearer " + token,
        },
      });
    } catch (error) {
      console.log(error.response);
    }
    items.value = await getItems();
  }
}

items.value = ["asdf", "fffff", "aaaaa", "ggggg", "tttt", "rrrrr"];
</script>
