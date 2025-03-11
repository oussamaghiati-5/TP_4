<script>
import { ref, computed, onMounted } from "vue";

export default {
  setup() {
    const piecesData = ref([]);
    const searchQuery = ref("");
    const selectedCategory = ref("");
    const sortOrder = ref("asc");
    const cart = ref([]);

    onMounted(() => {
      fetch("/pieces-autos.json")
        .then((response) => response.json())
        .then((data) => {
          piecesData.value = data;
        })
        .catch((error) => console.error("Error fetching JSON:", error));
    });

    const filteredPieces = computed(() => {
      let filtered = piecesData.value;
      if (searchQuery.value) {
        filtered = filtered.filter((piece) =>
          piece.nom.toLowerCase().includes(searchQuery.value.toLowerCase())
        );
      }
      if (selectedCategory.value) {
        filtered = filtered.filter(
          (piece) => piece.categorie === selectedCategory.value
        );
      }
      return filtered.sort((a, b) =>
        sortOrder.value === "asc" ? a.prix - b.prix : b.prix - a.prix
      );
    });

    const addToCart = (piece) => {
      cart.value.push(piece);
    };

    return {
      piecesData,
      searchQuery,
      selectedCategory,
      sortOrder,
      filteredPieces,
      addToCart,
    };
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Montserrat&display=swap");

body {
  max-width: 1200px;
  margin: auto;
  padding: 16px;
  font-family: "Montserrat", sans-serif;
}

header {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-bottom: 16px;
  padding: 8px;
  background-color: #7451eb;
  text-align: center;
  color: white;
}

header img {
  height: 60px;
  margin-left: 1rem;
}

header h1 {
  flex-grow: 1;
}

main {
  display: flex;
  flex-direction: row;
}

.filtres {
  border-radius: 4px;
  box-shadow: 0px 0px 4px gray;
  margin: 8px;
  padding: 8px;
  min-width: 300px;
  min-height: 400px;
}

.filtres h3 {
  text-align: center;
}

.fiches {
  flex: 1;
  margin: 8px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.fiche {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border: 1px solid #ddd;
  padding: 10px;
  width: 100%;
  box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
  background: white;
}

.fiche img {
  max-width: 100px;
  margin-right: 10px;
}

.fiche p {
  margin-top: 4px;
  margin-bottom: 4px;
}
</style>

<template>
  <body>
    <header>
      <h1>Les Bonnes Pièces</h1>
    </header>
    <main>
      <section class="filtres">
        <h3>Filtres</h3>
        <input v-model="searchQuery" placeholder="Rechercher..." />
        <select v-model="selectedCategory">
          <option value="">Toutes catégories</option>
          <option v-for="piece in piecesData" :key="piece.categorie" :value="piece.categorie">
            {{ piece.categorie }}
          </option>
        </select>
        <button @click="sortOrder = sortOrder === 'asc' ? 'desc' : 'asc'">
          Trier par prix {{ sortOrder === "asc" ? "↑" : "↓" }}
        </button>
      </section>

      <section class="fiches">
        <div v-for="piece in filteredPieces" :key="piece.id" class="fiche">
          <img :src="piece.image" :alt="piece.nom" />
          <div>
            <p>{{ piece.nom }}</p>
            <p>Prix: {{ piece.prix }}€</p>
            <p v-if="piece.Disponible">En stock</p>
            <p v-else>Rupture de stock</p>
            <button @click="addToCart(piece)">Ajouter au panier</button>
          </div>
        </div>
      </section>
    </main>
  </body>
</template>
