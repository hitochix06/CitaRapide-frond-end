<template>
  <div
    class="min-h-screen p-10 flex flex-col items-center justify-center space-y-12"
  >
    <div class="flex justify-center items-center">
      <h1
        class="text-4xl md:text-5xl lg:text-5xl font-bold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-teal-400 to-blue-500"
      >
        Mon
        <span class="text-teal-500 animate-pulse">citations inspirantes</span>
      </h1>
    </div>
    <!-- Bar with selection for themes -->
    <div class="w-full md:w-1/2 lg:w-1/3 xl:w-1/4">
      <div class="flex flex-col space-y-4">
        <!-- rajouter  citation -->

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

        <button
          @click="handleGenerate"
          class="py-3 px-4 bg-gradient-to-r from-green-400 to-blue-500 text-white font-bold rounded-md shadow hover:from-green-500 hover:to-blue-600 focus:ring-4 focus:ring-blue-300 focus:outline-none"
        >
          Rechercher
        </button>
        <!-- Ajouter un nouveau bouton pour afficher toutes les citations -->
        <button
          @click="showAllQuotes"
          :class="{
            'opacity-50 cursor-not-allowed': !isGenerated,
            'py-3 px-4 bg-gradient-to-r from-blue-400 to-purple-500 text-white font-bold rounded-md shadow hover:from-blue-500 hover:to-purple-600 focus:ring-4 focus:ring-purple-300 focus:outline-none':
              isGenerated,
          }"
        >
          Afficher toutes les citations
        </button>
        <div
          v-if="isLoading"
          class="w-full h-2 bg-blue-500 animate-pulse rounded"
        ></div>
      </div>
    </div>
    <!-- message -->
    <div v-if="errorMessage" class="text-red-500">{{ errorMessage }}</div>

    <!-- Cards with quotes -->
    <div class="grid gap-6 grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
      <div
        v-for="(quote, index) in quotes"
        :key="index"
        class="relative bg-white transform hover:scale-105 transition duration-500 ease-in-out rounded-lg shadow-lg overflow-hidden"
        :style="{ backgroundColor: quote.color }"
      >
        <i
          class="fas fa-times absolute top-0 right-0 cursor-pointer px-4 py-3 hover:text-black text-white"
          @click="handleDelete(quote._id)"
          ><svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-8 h-8"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0"
            />
          </svg>
        </i>
        <!-- Icône pour copier la citation -->
        <i
          class="fas fa-copy absolute top-0 left-0 cursor-pointer px-4 py-3 hover:text-black text-white"
          @click="copyQuote(quote)"
          ><svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-6 h-6"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M8.25 7.5V6.108c0-1.135.845-2.098 1.976-2.192.373-.03.748-.057 1.123-.08M15.75 18H18a2.25 2.25 0 0 0 2.25-2.25V6.108c0-1.135-.845-2.098-1.976-2.192a48.424 48.424 0 0 0-1.123-.08M15.75 18.75v-1.875a3.375 3.375 0 0 0-3.375-3.375h-1.5a1.125 1.125 0 0 1-1.125-1.125v-1.5A3.375 3.375 0 0 0 6.375 7.5H5.25m11.9-3.664A2.251 2.251 0 0 0 15 2.25h-1.5a2.251 2.251 0 0 0-2.15 1.586m5.8 0c.065.21.1.433.1.664v.75h-6V4.5c0-.231.035-.454.1-.664M6.75 7.5H4.875c-.621 0-1.125.504-1.125 1.125v12c0 .621.504 1.125 1.125 1.125h9.75c.621 0 1.125-.504 1.125-1.125V16.5a9 9 0 0 0-9-9Z"
            />
          </svg>
        </i>
        <div class="px-6 py-8">
          <div class="text-center">
            <!-- Utilisez v-if pour vérifier s'il y a des citations -->
            <p class="text-white" :class="quote.size" style="margin: 10px">
              {{ quote.emoji }}
            </p>
            <div v-if="quotes.length > 0">
              <h5 class="text-white text-base quote-proverb">
                {{ quote.proverbe }}
              </h5>
              <span class="text-white quote-author">— {{ quote.auteur }}</span>
            </div>
            <div v-else>
              <p class="text-white text-base">Aucune citation disponible.</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Ajouter un élément pour la notification de copie -->
  <div
    v-if="showCopyNotification"
    class="fixed bottom-4 right-4 bg-green-500 text-white py-2 px-4 rounded"
  >
    Citation copiée !
  </div>
</template>

<style scoped>
.quote-proverb {
  font-family: "Georgia", serif;
  font-size: 1.2em;
  font-style: italic;
}

.quote-author {
  font-family: "Arial", sans-serif;
  font-size: 1em;
}
</style>

<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";
import { selectData } from "../components/SelectData"; // Importer selectData

// Définir la valeur de quotes
const quotes = ref([]);

// Définir la valeur de isGenerated
const isGenerated = ref(false);

// Définir la valeur de isLoading
const isLoading = ref(false);

// Définir la valeur de search
const search = ref("");
const selectedTheme = ref(""); // Nouvelle variable pour le thème sélectionné
const errorMessage = ref(""); // Nouvelle variable pour le message d'erreur

// Ajouter une nouvelle variable réactive pour la notification
const showCopyNotification = ref(false);

// Ajouter une nouvelle fonction pour afficher toutes les citations
const showAllQuotes = () => {
  selectedTheme.value = ""; // Réinitialiser la valeur de selectedTheme
  fetchQuotes();
  isGenerated.value = false; // Réinitialiser la valeur de isGenerated
};

// Filtrer les données en fonction de la valeur de search
const filteredData = computed(() => {
  if (!search.value) return [];
  return selectData.filter((item) =>
    item.theme.toLowerCase().startsWith(search.value.toLowerCase())
  );
});

const handleClick = (item) => {
  // Mettre à jour selectedTheme au lieu de search
  search.value = item.theme;
  selectedTheme.value = item.theme;
};

// rechercher les citations
const fetchQuotes = async () => {
  isLoading.value = true; // Début du chargement
  try {
    const response = await axios.get(
      "https://cita-rapide-backend.vercel.app/citations"
    );
    let data = response.data;
    // Si un thème est sélectionné, filtrer les citations en fonction du thème
    if (selectedTheme.value) {
      data = data.filter((quote) => quote.theme === selectedTheme.value);
    }
    // Vérifier si des citations ont été trouvées pour le thème sélectionné
    if (data.length === 0) {
      errorMessage.value = "Aucune citation trouvée pour ce thème.";
      return;
    }
    // Associer chaque citation à son thème correspondant dans selectData
    setTimeout(() => {
      quotes.value = data.map((quote) => {
        const themeData = selectData.find((item) => item.theme === quote.theme);
        return { ...quote, ...themeData };
      });
      errorMessage.value = ""; // Réinitialiser le message d'erreur
      isLoading.value = false; // Fin du chargement
    }, 2000); // Ajouter un délai de 5 secondes
  } catch (error) {
    console.error(error);
    errorMessage.value = "Impossible de se connecter à la base de données.";
    isLoading.value = false; // Fin d
  }
};

// Générer des citations
const handleGenerate = () => {
  fetchQuotes();
  search.value = ""; // Réinitialiser la valeur de search
  isGenerated.value = true; // Mettre à jour la valeur de isGenerated
};

// Supprimer une citation par son id de thème
const handleDelete = async (_id) => {
  try {
    const response = await axios.delete(
      `https://cita-rapide-backend.vercel.app/citations/${_id}`
    );
    if (response.status === 200) {
      const index = quotes.value.findIndex((quote) => quote._id === _id);
      if (index !== -1) {
        quotes.value.splice(index, 1);
      }
    } else {
      console.error(`Échec de la suppression de la citation avec l'id ${_id}`);
    }
  } catch (error) {
    console.error(error);
    errorMessage.value =
      "Une erreur s'est produite lors de la suppression de la citation.";
  }
};

// Fonction pour copier la citation et afficher la notification
const copyQuote = async (quote) => {
  try {
    await navigator.clipboard.writeText(`${quote.proverbe} — ${quote.auteur}`);
    showCopyNotification.value = true; // Afficher la notification
    setTimeout(() => {
      showCopyNotification.value = false; // Cacher la notification après 2 secondes
    }, 2000);
  } catch (error) {
    console.error("Erreur lors de la copie de la citation", error);
  }
};

onMounted(fetchQuotes);
</script>
