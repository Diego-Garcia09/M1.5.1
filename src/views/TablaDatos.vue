<template>
    <v-data-table :headers="headers" :items="publicaciones" :sort-by="[{ key: 'id', order: 'asc' }]" class="elevation-1">
        <template v-slot:top>
            <v-toolbar flat>
                <v-toolbar-title>Mi CRUD</v-toolbar-title>
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
                                        <v-text-field v-model="editedItem.name" label="Autor"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.title" label="Title"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.body" label="Body"></v-text-field>
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
        title: 'Autor',
        align: 'start',
        sortable: false,
        key: 'name',
    },
    { title: 'Title', key: 'title' },
    { title: 'Body', key: 'body' },
    { title: 'Actions', key: 'actions', sortable: false },
];
const publicaciones = ref([]);
const editedIndex = ref(-1);
const editedItem = reactive({
    id: 0,
    name: '',
    title: 0,
    body: 0,
});
const defaultItem = {
    id: 0,
    name: '',
    title: 0,
    body: 0,
};

const editItem = async (item) => {
    editedItem.id = item.id;
    editedItem.name = item.name;
    editedItem.title = item.title;
    editedItem.body = item.body;
    dialog.value = true;
    editedIndex.value = publicaciones.value.findIndex((d) => d.id === item.id);
};

const save = async () => {
    if (editedIndex.value === -1) {
        const newItem = {
            title: editedItem.title.toString(),
            body: editedItem.body.toString(),
            userId: editedItem.id,
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

            publicaciones.value.push({
                id: jsonResponse.id,
                name: editedItem.name,
                title: editedItem.title,
                body: editedItem.body,
            });

            close();
        } else {
            console.error('Error al agregar el elemento');
        }
    } else {
        const editedDessert = {
            id: editedIndex.value,
            title: editedItem.title.toString(),
            body: editedItem.body.toString(),
            userId: editedItem.id,
        };
        
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${parseInt(editedIndex.value)+1}`, {
            method: 'PUT',
            body: JSON.stringify({
                userId: editedDessert.userId,
                id: editedDessert.id,
                title: editedDessert.title,
                body: editedDessert.body,
            }),
            headers: {
                'Content-type': 'application/json; charset=UTF-8',
            },
        });

        if (response.ok) {
            const jsonResponse = await response.json();

            const index = editedIndex.value;
            publicaciones.value[index] = {
                id: jsonResponse.id,
                name: editedItem.name,
                title: editedItem.title,
                body: editedItem.body,
            };

            close();
        } else {
            console.error('Error al editar el elemento');
        }
    }
};

const deleteItem = async (item) => {
        try {
            const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${item.id}`, {
                method: 'DELETE',
            });

            if (response.ok) {
                const index = publicaciones.value.findIndex((d) => d.id === item.id);
                if (index !== -1) {
                    publicaciones.value.splice(index, 1);
                }

                closeDelete();
            } else {
                console.error('Error al eliminar el elemento');
            }
        } catch (error) {
            console.error('Error al eliminar el elemento', error);
        }
};

const formTitle = computed(() => {
    return editedIndex.value === -1 ? 'New Item' : 'Edit Item';
});

const close = () => {
    dialog.value = false;
    Object.assign(editedItem, defaultItem);
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
  const mergedPublicaciones = [];
  for (const post of posts.value) {
    const user = users.value.find((u) => u.id === post.userId);
    if (user) {
      mergedPublicaciones.push({
        id: post.id,
        name: user.name,
        title: post.title,
        body: post.body,
      });
    }
  }
  publicaciones.value = mergedPublicaciones;
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