<template>
    <v-data-table :headers="headers" :items="mergedData" :sort-by="[{ key: 'id', order: 'asc' }]" class="elevation-1">
        <template v-slot:top>
            <v-toolbar flat>
                <v-toolbar-title>My CRUD</v-toolbar-title>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ props }">
                        <v-btn color="primary" dark class="mb-2" v-bind="props">
                            New Item
                        </v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="text-h5">{{ formTitle }}</span>
                        </v-card-title>
                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.id" label="ID"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.calories" label="Calories"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.fat" label="Fat (g)"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4"></v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue-darken-1" variant="text" @click="close">Cancel</v-btn>
                            <v-btn color="blue-darken-1" variant="text" @click="save">Save</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDelete" max-width="500px">
                    <v-card>
                        <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancel</v-btn>
                            <v-btn color="blue-darken-1" variant="text" @click="deleteItemConfirm">OK</v-btn>
                            <v-spacer></v-spacer>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
        </template>
        <template v-slot:item.actions="{ item }">
            <v-icon size="small" class="me-2" @click="editItem(item.raw)">mdi-pencil</v-icon>
            <v-icon size="small" @click="deleteItem(item.raw)">mdi-delete</v-icon>
        </template>
    </v-data-table>
</template>
  
<script setup>
import { ref, computed, onMounted } from 'vue';

const dialog = ref(false);
const dialogDelete = ref(false);
const editedIndex = ref(-1);

const headers = [
    {
        title: 'Autores',
        align: 'start',
        sortable: false,
        key: 'name',
    },
    {
        title: 'Titulo',
        sortable: false,
        key: 'calories',
    },
    {
        title: 'Post',
        sortable: false,
        key: 'fat',
    },
    { 
        title: 'Actions', 
        key: 'actions', 
        sortable: false 
    },
];

const editedItem = ref({
    id: 0,
    name: '',
    calories: 0,
    fat: 0,
});

const defaultItem = {
    id: 0,
    name: '',
    calories: 0,
    fat: 0,
};

const formTitle = computed(() => {
    return editedIndex.value === -1 ? 'New Item' : 'Edit Item';
});

const close = () => {
    dialog.value = false;
    editedItem.value = { ...defaultItem };
    editedIndex.value = -1;
};

const closeDelete = () => {
    dialogDelete.value = false;
    editedItem.value = { ...defaultItem };
    editedIndex.value = -1;
};

const editItem = (item) => {
    editedIndex.value = mergedData.value.indexOf(item);
    editedItem.value = { ...item };
    dialog.value = true;
};

const deleteItem = (item) => {
    editedIndex.value = mergedData.value.indexOf(item);
    editedItem.value = { ...item };
    dialogDelete.value = true;
};

const deleteItemConfirm = () => {
    mergedData.value.splice(editedIndex.value, 1);
    closeDelete();
};

const save = () => {
    if (editedIndex.value > -1) {
        Object.assign(mergedData.value[editedIndex.value], editedItem.value);
    } else {
        mergedData.value.push({ ...editedItem.value });
    }
    close();
};

const posts = ref([]);
const users = ref([]);

async function obtenerInfo() {
    const response = await fetch("https://jsonplaceholder.typicode.com/posts/");
    const data = await response.json();
    posts.value = data;
    console.log(posts.value)
}

async function obtenerUsuario() {
    const response = await fetch("https://jsonplaceholder.typicode.com/users/");
    const data = await response.json();
    users.value = data;
    console.log(users.value)
}

const mergedData = computed(() => {
  const merged = [];
  for (const post of posts.value) {
    const user = users.value.find((u) => u.id === post.userId);
    if (user) {
      merged.push({
        id: users.id,
        name: user.name,
        calories: post.title,
        fat: post.body,
      });
    }
  }
  return merged;
});

onMounted(() => {
    obtenerInfo();
    obtenerUsuario();
});
</script>
  