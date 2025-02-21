<script lang="ts" setup>
import { ref, computed, onMounted } from "vue";
import { useFetch } from "#imports";

export interface Character {
  info: Info;
  results: Result[];
}

export interface Info {
  count: number;
  pages: number;
  next: string;
  prev: null;
}

export interface Result {
  id: number;
  name: string;
  status: string;
  species: string;
  gender: string;
  image: string;
}

const dialog = ref(false);
const editingCharacter = ref(false);
const characters = ref<Result[]>([]);
const searchQuery = ref("");

const newCharacter = ref<Result>({
  id: 0,
  name: "",
  status: "",
  species: "",
  gender: "",
  image: "",
});

const openCreateModal = () => {
  editingCharacter.value = false;
  newCharacter.value = {
    id: 0,
    name: "",
    status: "",
    species: "",
    gender: "",
    image: "",
  };
  dialog.value = true;
};

const saveCharacter = () => {
  if (!newCharacter.value.name || !newCharacter.value.status || !newCharacter.value.species) return;

  if (editingCharacter.value) {
    const index = characters.value.findIndex(c => c.id === newCharacter.value.id);
    if (index !== -1) {
      characters.value[index] = { ...newCharacter.value };
    }
  } else {
    const newId = characters.value.length ? Math.max(...characters.value.map(c => c.id)) + 1 : 1;
    characters.value.push({ ...newCharacter.value, id: newId });
  }

  dialog.value = false;
};

const { data } = await useFetch<Character>("https://rickandmortyapi.com/api/character");

onMounted(() => {
  if (data.value?.results) {
    characters.value = data.value.results;
  }
});

const exploreCharacter = (character: Result) => {
  alert(`Explorando a ${character.name}: ${character.species} - ${character.status}`);
};

const editCharacter = (character: Result) => {
  newCharacter.value = { ...character };
  editingCharacter.value = true;
  dialog.value = true;
};

const deleteCharacter = (characterId: number) => {
  characters.value = characters.value.filter(character => character.id !== characterId);
};

const filteredCharacters = computed(() => {
  if (!searchQuery.value.trim()) {
    return characters.value;
  }
  return characters.value.filter((character) =>
      character.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      character.status.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      character.species.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});
</script>

<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12" md="6">
        <v-text-field
            variant="solo"
            v-model="searchQuery"
            label="Buscar personaje..."
            prepend-inner-icon="mdi-magnify"
            clearable
            @update:modelValue="searchQuery = $event"
            @click:clear="searchQuery = ''"
        />
      </v-col>
      <v-col class="text-right">
        <v-btn size="large" prepend-icon="mdi-plus" color="#1481ff" variant="tonal" @click="openCreateModal">
          Crear
        </v-btn>
      </v-col>
    </v-row>

    <v-dialog v-model="dialog" max-width="400px">
      <v-card>
        <v-card-title>{{ editingCharacter ? "Editar Personaje" : "Crear Nuevo Personaje" }}</v-card-title>
        <v-card-text>
          <v-text-field v-model="newCharacter.name" label="Nombre"></v-text-field>
          <v-select v-model="newCharacter.status" :items="['Alive', 'Dead', 'Unknown']" label="Estado"></v-select>
          <v-text-field v-model="newCharacter.species" label="Especie"></v-text-field>
          <v-select v-model="newCharacter.gender" :items="['Male', 'Female', 'Unknown']" label="Género"></v-select>
          <v-text-field v-model="newCharacter.image" label="URL de Imagen"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn color="red" variant="tonal" @click="dialog = false">Cancelar</v-btn>
          <v-btn color="green" variant="tonal" @click="saveCharacter">{{ editingCharacter ? "Actualizar" : "Guardar" }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>

  <v-container fluid>
    <v-row>
      <v-col v-for="item in filteredCharacters" :key="item.id" cols="12" md="4">
        <v-card class="mx-auto" max-width="344">
          <v-img height="200px" :src="item.image || 'https://via.placeholder.com/200'" cover></v-img>
          <v-card-title>{{ item.name }}</v-card-title>
          <v-card-subtitle>{{ item.species }} - {{ item.status }}</v-card-subtitle>
          <v-card-actions>
            <v-btn color="orange-lighten-2" @click="exploreCharacter(item)">Explorar</v-btn>
            <v-btn color="#4ea9ff" @click="editCharacter(item)">Editar</v-btn>
            <v-btn color="red-darken-2" @click="deleteCharacter(item.id)">Eliminar</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>