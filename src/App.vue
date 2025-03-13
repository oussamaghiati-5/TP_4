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
      cart,
    };
  },
};
</script>

<style scoped>
@import url("https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css");

body {
  font-family: "Montserrat", sans-serif;
  background-color: #f8f9fa;
}

.container {
  max-width: 1200px;
  margin: auto;
  padding: 20px;
}

header {
  background-color: #7451eb;
  color: white;
  padding: 15px;
  text-align: center;
  font-size: 24px;
  font-weight: bold;
}

.filtres {
  background: white;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
}

.fiches {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.fiche {
  width: 18rem;
  background: white;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  padding: 10px;
  text-align: center;
}

.fiche img {
  width: 100%;
  height: auto;
  border-radius: 5px;
}

.panier {
  background: white;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}
</style>

<template>
  <div class="container">
    <header>
      Les Bonnes Pièces
    </header>

    <div class="row mt-3">
      <!-- Filters Section -->
      <div class="col-md-3 filtres">
        <h3>Filtres</h3>
        <input v-model="searchQuery" class="form-control mb-2" placeholder="Rechercher..." />
        <select v-model="selectedCategory" class="form-select mb-2">
          <option value="">Toutes catégories</option>
          <option v-for="piece in piecesData" :key="piece.categorie" :value="piece.categorie">
            {{ piece.categorie }}
          </option>
        </select>
        <button @click="sortOrder = sortOrder === 'asc' ? 'desc' : 'asc'" class="btn btn-primary w-100">
          Trier par prix {{ sortOrder === "asc" ? "↑" : "↓" }}
        </button>
      </div>

      <!-- Products Section -->
      <div class="col-md-6 fiches d-flex flex-wrap justify-content-center">
        <div v-for="piece in filteredPieces" :key="piece.id" class="fiche card">
          <img :src="piece.image" :alt="piece.nom" class="card-img-top" />
          <div class="card-body">
            <h5 class="card-title">{{ piece.nom }}</h5>
            <p class="card-text">Prix: {{ piece.prix }}€</p>
            <p class="card-text" :class="{ 'text-success': piece.Disponible, 'text-danger': !piece.Disponible }">
              {{ piece.Disponible ? "En stock" : "Rupture de stock" }}
            </p>
            <button @click="addToCart(piece)" class="btn btn-success w-100">Ajouter au panier</button>
          </div>
        </div>
      </div>

      <!-- Cart Section -->
      <div class="col-md-3 panier">
        <h3>Panier</h3>
        <ul class="list-group">
          <li v-for="item in cart" :key="item.id" class="list-group-item">
            {{ item.nom }} - {{ item.prix }}€
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
