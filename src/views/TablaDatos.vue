<template>
    <v-data-table :headers="headers" :items="desserts" :sort-by="[{ key: 'id', order: 'asc' }]" class="elevation-1">
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
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue-darken-1" variant="text" @click="close">
                                Cancel
                            </v-btn>
                            <v-btn color="blue-darken-1" variant="text" @click="save">
                                Save
                            </v-btn>
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
            <v-icon size="small" class="me-2" @click="editItem(item.raw)">
                mdi-pencil
            </v-icon>
            <v-icon size="small" @click="deleteItem(item.raw)">
                mdi-delete
            </v-icon>
        </template>
        <template v-slot:no-data>
            <v-btn color="primary" @click="initialize">
                Reset
            </v-btn>
        </template>
    </v-data-table>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue';

const dialog = ref(false);
const dialogDelete = ref(false);
const headers = [
    {
        title: 'Dessert (100g serving)',
        align: 'start',
        sortable: false,
        key: 'name',
    },
    { title: 'Calories', key: 'calories' },
    { title: 'Fat (g)', key: 'fat' },
    { title: 'Actions', key: 'actions', sortable: false },
];
const desserts = ref([]);
const editedIndex = ref(-1);
const editedItem = reactive({
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

const editItem = async (item) => {
    // Asigna los valores del elemento editado a editedItem
    editedItem.id = item.id;
    editedItem.name = item.name;
    editedItem.calories = item.calories;
    editedItem.fat = item.fat;

    // Abre el diálogo de edición
    dialog.value = true;
    // Encuentra el índice del elemento editado en la lista desserts
    editedIndex.value = desserts.value.findIndex((d) => d.id === item.id);

    const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${editedIndex}`, {
        method: 'PUT',
        body: JSON.stringify({
            id: editedIndex,
            title: editedItem.calories,
            body: editItem.fat,
            userId: editedItem.id,
        }),
        headers: {
            'Content-type': 'application/json; charset=UTF-8',
        },
    });
};

const save = async () => {
    const newItem = {
        title: editedItem.calories.toString(), // Utiliza el campo 'name' para el título del nuevo elemento
        body: editedItem.fat.toString(), // Utiliza el campo 'calories' como el cuerpo del nuevo elemento (conversión a cadena)
        userId: editedItem.id, // Puedes establecer el 'userId' según tus necesidades
    };

    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        body: JSON.stringify(newItem),
        headers: {
            'Content-type': 'application/json; charset=UTF-8',
        },
    });

    if (response.ok) {
        const jsonResponse = await response.json();

        // Agrega el nuevo elemento a la lista 'desserts'
        desserts.value.push({
            id: jsonResponse.id, // Utiliza el ID devuelto por el servidor
            name: editedItem.name,
            calories: editedItem.calories,
            fat: editedItem.fat,
        });

        // Cierra el diálogo y restablece el formulario
        close();
    } else {
        console.error('Error al agregar el elemento');
    }
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

const initialize = async () => {
    await obtenerInfo();
    await obtenerUsuario();
    await mergedData();
}

async function mergedData() {
  const mergedDesserts = [];
  for (const post of posts.value) {
    const user = users.value.find((u) => u.id === post.userId);
    if (user) {
      mergedDesserts.push({
        id: post.id,
        name: user.name,
        calories: post.title,
        fat: post.body,
      });
    }
  }
  desserts.value = mergedDesserts;
}

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

onMounted(() => {
    initialize();
});

watch(dialog, (val) => {
    val || close();
});

watch(dialogDelete, (val) => {
    val || closeDelete();
});

</script>