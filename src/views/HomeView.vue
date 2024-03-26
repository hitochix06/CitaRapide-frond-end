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
        <!-- rajouter  citation -->
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

        <button
          @click="handleGenerate"
          class="py-3 px-4 bg-gradient-to-r from-green-400 to-blue-500 text-white font-bold rounded-md shadow hover:from-green-500 hover:to-blue-600 focus:ring-4 focus:ring-blue-300 focus:outline-none"
        >
          Générer
        </button>
        <!-- Ajouter un nouveau bouton pour afficher toutes les citations -->
      </div>
    </div>
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
import { ref, computed, onMounted } from "vue";
import axios from "axios";
import { selectData } from "../components/SelectData"; // Importer selectData

const toastMessage = ref("");
const showToast = ref(false);

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
    const response = await axios.get("http://localhost:3000/citations");
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

const generateQuoteFromAPI = async () => {
  if (!selectedTheme.value) {
    errorMessage.value =
      "Veuillez sélectionner un thème pour générer une citation.";
    return;
  }
  isLoading.value = true; // Début du chargement
  try {
    // Modifier l'URL et la structure de la requête pour correspondre à la nouvelle API
    const response = await axios.get(
      `https://cita-rapide-backend.vercel.app/v1/quotes/${selectedTheme.value}`
    );
    if (response.data && response.data.length > 0) {
      const quote = response.data[0]; // Supposer que l'API renvoie un tableau de citations
      // Envoyer la citation à votre API backend pour stockage, si nécessaire
      const saveResponse = await axios.post(
        "https://cita-rapide-backend.vercel.app/citations",
        {
          proverbe: quote.quote, // Assurez-vous que ces champs correspondent à ceux attendus par votre API backend
          auteur: quote.author,
          theme: selectedTheme.value,
        }
      );
      if (saveResponse.data && saveResponse.data.result) {
        // Citation enregistrée avec succès, ajouter à l'état local si nécessaire
        quotes.value.push(saveResponse.data.citation);
        errorMessage.value = "";
      } else {
        errorMessage.value =
          "Impossible de sauvegarder la citation dans la base de données.";
      }
    } else {
      errorMessage.value = "Aucune citation trouvée pour ce thème.";
    }
  } catch (error) {
    console.error(error);
    errorMessage.value =
      "Erreur lors de la connexion à l'API externe ou à l'API backend.";
  }
  isLoading.value = false; // Fin du chargement
};

// Modifier la méthode handleGenerate pour utiliser generateQuoteFromAPI
const handleGenerate = () => {
  generateQuoteFromAPI();
  search.value = ""; // Réinitialiser la valeur de search
  isGenerated.value = true; // Mettre à jour la valeur de isGenerated
  displayToast("Citation bien rajoutée"); // Afficher un toast pour confirmer l'ajout
};

const displayToast = (message) => {
  toastMessage.value = message;
  showToast.value = true;
  setTimeout(() => {
    showToast.value = false;
  }, 3000); // Le toast disparaît après 3 secondes
};

onMounted(fetchQuotes);
</script>
