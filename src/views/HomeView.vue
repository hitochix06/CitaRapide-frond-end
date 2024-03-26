<template>
  <div
    class="min-h-screen p-10 flex flex-col items-center justify-center space-y-12"
  >
    <div class="flex justify-center items-center">
      <h1
        class="text-4xl md:text-5xl lg:text-5xl font-bold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-teal-400 to-blue-500"
      >
        Générateur de
        <span class="text-teal-500 animate-pulse">citations inspirantes</span>
      </h1>
    </div>
    <!-- Bar with selection for themes -->
    <div class="w-full md:w-1/2 lg:w-1/3 xl:w-1/4">
      <div class="flex flex-col space-y-4">
        <transition name="slide-fade">
          <div
            v-if="showToast"
            class="fixed top-5 right-5 bg-gradient-to-r from-green-400 to-blue-500 text-white py-2 px-4 rounded"
          >
            {{ toastMessage }}
          </div>
        </transition>

        <input
          v-model="search"
          class="form-input block w-full text-gray-700 py-3 px-4 border border-gray-300 bg-white rounded-md shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
          placeholder="Rechercher un thème"
        />
        <div
          v-for="(item, index) in filteredData"
          :key="index"
          @click="handleClick(item)"
          class="cursor-pointer p-2 rounded hover:bg-[#39C3B4] hover:text-white text-center"
        >
          {{ item.theme }}
        </div>
        <!-- Le bouton Générer apparaît seulement si un thème est sélectionné -->
        <button
          v-if="selectedTheme && search.trim()"
          @click="handleGenerate"
          class="py-3 px-4 font-bold rounded-md shadow focus:outline-none bg-gradient-to-r from-green-400 to-blue-500 text-white hover:from-green-500 hover:to-blue-600 focus:ring-4 focus:ring-blue-300"
        >
          Générer
        </button>
        <!-- Ajouter un nouveau bouton pour afficher toutes les citations -->
      </div>
    </div>
  </div>
</template>

<style scoped>
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter,
.slide-fade-leave-to {
  opacity: 0;
  transform: translateX(50px);
}
</style>

<script setup>
import { ref, computed } from "vue";
import axios from "axios";
import { selectData } from "../components/SelectData";

const toastMessage = ref("");
const showToast = ref(false);

const search = ref("");
const selectedTheme = ref(null);
const errorMessage = ref("");
const quotes = ref([]); // Assurez-vous d'initialiser quotes si vous souhaitez stocker les citations générées

const filteredData = computed(() => {
  return search.value
    ? selectData.filter((item) =>
        item.theme.toLowerCase().startsWith(search.value.toLowerCase())
      )
    : [];
});

const handleClick = (item) => {
  selectedTheme.value = item.theme; // Stocker le thème sélectionné
  search.value = item.theme; // Mettre à jour la barre de recherche avec le nom du thème sélectionné
  errorMessage.value = ""; // Réinitialiser le message d'erreur lorsqu'un thème est sélectionné
};

const generateQuoteFromAPI = async () => {
  try {
    const response = await axios.get(
      `https://cita-rapide-backend.vercel.app/v1/quotes/${selectedTheme.value}`
    );
    if (response.data && response.data.length > 0) {
      const quote = response.data[0];
      const saveResponse = await axios.post(
        "https://cita-rapide-backend.vercel.app/citations",
        {
          proverbe: quote.quote,
          auteur: quote.author,
          theme: selectedTheme.value,
        }
      );
      if (saveResponse.data && saveResponse.data.result) {
        quotes.value.push(saveResponse.data.citation);
        displayToast("Citation bien rajoutée");
        errorMessage.value = "";
      } else {
        errorMessage.value =
          "Impossible de sauvegarder la citation dans la base de données.";
      }
    }
  } catch (error) {
    console.error(error);
    errorMessage.value =
      "Erreur lors de la connexion à l'API externe ou à l'API backend.";
  }
};

const handleGenerate = async () => {
  await generateQuoteFromAPI();
  search.value = ""; // Réinitialiser la recherche
  selectedTheme.value = null; // Réinitialiser le thème sélectionné
  displayToast("Citation bien rajoutée");
};

const displayToast = (message) => {
  toastMessage.value = message;
  showToast.value = true;
  setTimeout(() => {
    showToast.value = false;
  }, 3000);
};
</script>
