<template>
  <main>
    <div>
    <div>
      <label for="categorie">Choisir une catégorie :</label>
      <select id="categorie" v-model="data.categorieSelectionnee" @change="chargerProduitsParCategorie">
        <option disabled value="">-- Sélectionnez une catégorie --</option>
        <option v-for="categorie in data.listeCategories" :key="categorie.code" :value="categorie.code">
          {{ categorie.libelle }}
        </option>
      </select>
    </div>
      <table>
        <caption>Liste des produits par catégorie</caption>
        <thead>
          <tr>
            <th>Nom</th>
            <th>Prix</th>
            <th>Stock</th>
            <th>Commandés</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="data.listeProduits.length === 0">
            <td colspan="4">Veuillez patienter, chargement des produits...</td>
          </tr>
          <tr v-for="produit in data.listeProduits" :key="produit.nom">
            <td>{{ produit.nom }}</td>
            <td>{{ produit.prixUnitaire }}</td>
            <td>{{ produit.unitesEnStock }}</td>
            <td>{{ produit.unitesCommandees }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </main>
</template>


<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest } from "@/api";

const produitVide = {
  libelle: "",
  description: ""
};

// Pour réinitialiser le formulaire
const categorieVide = {
    libelle: "",
    description: ""
};

const data = reactive({
  formulaireProduit: { ...produitVide },
  listeProduits: [],
  listeCategories: [],
  categorieSelectionnee: "",
  pagination: {
    current: null,
    first: null,
    prev: null,
    next: null,
    last: null,
    pageNumber: 0,
    totalPages: 1
  }
});

function showError(error) {
  console.error("Erreur :", error.status);
  console.error(error.body);
  alert(error.message);
}

function chargerProduits() {
// Appel à l'API pour avoir la liste des catégories
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest("/api/produits?")
        .then((json) => {
            data.listeProduits = json._embedded.produits;
        })
        .catch(showError);
}

function chargerProduitsParCategorie() {
// Appel à l'API pour avoir la liste des catégories
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest(`/api/categories/${data.categorieSelectionnee}/produits`)
        .then((json) => {
            data.listeProduits = json._embedded.produits;
        })
        .catch(showError);
}

function chargerPage(url) {
  if (!url) return;

  doAjaxRequest(url)
    .then((json) => {
      data.listeProduits = json._embedded.produits;

      data.pagination.current = json._links.self?.href || null;
      data.pagination.first = json._links.first?.href || null;
      data.pagination.prev = json._links.prev?.href || null;
      data.pagination.next = json._links.next?.href || null;
      data.pagination.last = json._links.last?.href || null;

      data.pagination.pageNumber = json.page.number;
      data.pagination.totalPages = json.page.totalPages;
    })
    .catch(showError);
}

function chargerCategories() {
    // Appel à l'API pour avoir la liste des catégories
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest("/api/categories?sort=code,desc")
        .then((json) => {
            data.listeCategories = json._embedded.categories;
        })
        .catch(showError);
}

onMounted(() => {
    chargerCategories();
    chargerProduits();
});
</script>



<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}

th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}
</style>
