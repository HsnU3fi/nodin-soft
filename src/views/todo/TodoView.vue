<template>
  <v-container class="fill-height">
    <v-row justify="center" align="center">
      <v-card
          style="  overflow: scroll;"
          class="mx-auto b-r br"
          width="70%"
          height="600"
          elevation="15"
          variant="outlined"
      >


        <v-data-table
            :dir="dir"
            height="600"
            :headers="headers"
            :items="desserts"
        >
          <template v-slot:top>
            <v-toolbar
                flat
            >
              <v-toolbar-title>{{ t('todo.todoList') }}</v-toolbar-title>

              <v-spacer></v-spacer>
              <v-dialog
                  v-model="dialog"
                  max-width="500px"
              >
                <template v-slot:activator="{ props }">
                  <v-btn
                      color="#4ED1F1"
                      dark
                      class="mb-2"
                      v-bind="props"
                      @click="changTitle(t('todo.formTitle'))"
                  >
                    <h1 style="font-size: 15px">
                      {{ t('todo.newItem') }}
                    </h1>
                  </v-btn>
                </template>
                <v-card>
                  <v-card-title :dir="dir">
                    <span
                          class="text-h5">{{ formTitle }}</span>
                  </v-card-title>

                  <v-card-text>
                    <v-container>
                      <v-row>

                        <v-textarea
                            :dir="dir"
                            v-model="todo"
                            variant="outlined"
                            :label="t('todo.todo')"
                        ></v-textarea>

                      </v-row>
                    </v-container>
                  </v-card-text>

                  <v-card-actions :dir="dir">
                    <v-spacer></v-spacer>
                    <v-btn
                        color="#4ED1F1"
                        variant="text"
                        @click="close"
                    >
                      <h1 style="font-size: 15px">
                        {{ t('todo.todoCancel') }}
                      </h1>
                    </v-btn>
                    <v-btn

                        color="#4ED1F1"
                        variant="text"
                        @click="save"
                    >
                      <h1  style="font-size: 15px">
                        {{ t('todo.todoSave') }}

                      </h1>
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
              <v-dialog
                        v-model="dialogDelete" max-width="500">
                <v-card class="b-r">
                  <v-card-title :dir="dir"
                                class="text-h5">{{ t('todo.textDelete') }}
                  </v-card-title>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn  color="#4ED1F1" variant="text" @click="closeDelete"><h1 style="font-size: 15px">
                      {{ t('todo.todoCancel') }}
                    </h1></v-btn>
                    <v-btn color="#4ED1F1" variant="text" @click="deleteItemConfirm"><h1 style="font-size: 15px">
                      {{ t('todo.todoSave') }}
                    </h1></v-btn>
                    <v-spacer></v-spacer>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
          <template v-slot:item.actions="{ item }">
            <img class="me-2"
                 @click="editItem(item)"
                 style="width: 40px;cursor: pointer" src="../../assets/img/edit.png">

            <img class="me-2"
                 @click="deleteItem(item)"
                 style="width: 40px;cursor: pointer" src="../../assets/img/trash.png">

          </template>
        </v-data-table>

      </v-card>
    </v-row>
    <loading :value="loading"/>

  </v-container>
</template>
<script setup>
//======================================================================================================================
import {ref, watch} from 'vue';
import {useTodoStore} from "../../stores/todo";
import {useI18n} from 'vue-i18n'
import {useProfileStore} from "../../stores/profile";
const {t} = useI18n()
const todoStore = useTodoStore();
const dialog = ref(false);
const loading = ref(true);
const todo = ref('');
const formTitle = ref('New Item');
const dialogDelete = ref(false);
const profileStore = useProfileStore();
const dir = ref('ltr')
//======================================================================================================================
const getDir = async ()=>{
  dir.value = await profileStore.Rtl()
}
getDir();
//======================================================================================================================
const headers = [
  {
    title: t('todo.headerTodo'),
    align: 'start',
    sortable: false,
    key: 'todo',
  },
  {title: t('todo.headerActions'), key: 'actions', sortable: false, align: 'end'},
];
const desserts = ref([]);
const editedIndex = ref(-1);
const editedItem = ref({
  todo: '',
});
const defaultItem = {
  todo: '',
};
//======================================================================================================================
const changTitle = (title) => {
  formTitle.value = title
}
//======================================================================================================================
const initialize =  () => {
  const data = JSON.parse(todoStore.getTodos()) || []
  desserts.value = data
};
//======================================================================================================================
const editItem = (item) => {
  changTitle(t('todo.editItem'))
  editedIndex.value = desserts.value.indexOf(item);
  editedItem.value = {...item};
  todo.value = item.todo
  dialog.value = true;
};
//======================================================================================================================
const deleteItem = (item) => {
  editedIndex.value = desserts.value.indexOf(item);
  editedItem.value = {...item};
  dialogDelete.value = true;
};
//======================================================================================================================
const deleteItemConfirm = () => {
  desserts.value.splice(editedIndex.value, 1);
  const storedDesserts = JSON.parse(localStorage.getItem("todo"));
  if (storedDesserts) {
    storedDesserts.splice(editedIndex.value, 1);
    localStorage.setItem("todo", JSON.stringify(storedDesserts));
  }
  closeDelete();
};
//======================================================================================================================
const close = () => {
  dialog.value = false;
  editedItem.value = {...defaultItem};
  editedIndex.value = -1;
};
//======================================================================================================================
const closeDelete = () => {
  dialogDelete.value = false;
  editedItem.value = {...defaultItem};
  editedIndex.value = -1;
};
//======================================================================================================================
const save = () => {
  if (formTitle.value === t('todo.newItem')) {
    loading.value = true
    const data = JSON.parse(todoStore.getTodos())
    if (data) {
      const items = [{todo: todo.value}]
      for (var i in data) {
        items.push(data[i])
      }
      todoStore.saveTodoInLocalStorage(items)
      initialize();
      dialog.value = false
      setTimeout(() => {
        loading.value = false
      }, 3000)
    } else {
      const item = [{todo: todo.value}]
      todoStore.saveTodoInLocalStorage(item)
      initialize();
      dialog.value = false
      setTimeout(() => {
        loading.value = false
      }, 3000)
    }
  } else {
    loading.value = true;
    const data = JSON.parse(todoStore.getTodos()) || [];
    const editedItemIndex = editedIndex.value
    if (editedItemIndex !== -1) {
      data[editedItemIndex].todo = todo.value;
      todoStore.saveTodoInLocalStorage(data);
      initialize();
      dialog.value = false;
      setTimeout(() => {
        loading.value = false;
      }, 3000);
    } else {
      console.error("Item to edit not found");
    }
  }
};
//======================================================================================================================
watch(dialog, (val) => {
  val || close();
});
//======================================================================================================================
watch(dialogDelete, (val) => {
  val || closeDelete();
});
//======================================================================================================================
initialize();
setTimeout(() => {
  loading.value = false
}, 3000)
</script>

<style scoped>

</style>