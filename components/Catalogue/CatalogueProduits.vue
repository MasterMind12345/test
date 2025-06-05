<template>
  <div class="catalogue-container">
    <!-- Filtres par catégorie -->
    <div class="filters-section">
      <h2 class="section-title">Filtrer par catégorie</h2>
      <div class="categories-list">
        <div 
          v-for="category in categories" 
          :key="category.id"
          class="category-item"
        >
          <label class="category-checkbox">
            <input 
              type="checkbox" 
              :value="category.id" 
              v-model="selectedCategories"
              @change="filterProducts"
            >
            <span class="checkmark"></span>
            {{ category.Nom }}
          </label>
        </div>
      </div>
    </div>

    <!-- Liste des produits filtrés -->
    <div class="products-section">
      <div class="section-header">
        <h2 class="section-title">Catalogue des produits</h2>
        <div class="product-count">{{ filteredProducts.length }} produits</div>
      </div>
      
      <div v-if="loading" class="loading">Chargement en cours...</div>
      <div v-else-if="error" class="error">{{ error }}</div>
      
      <div v-else class="products-grid">
        <div 
          v-for="product in filteredProducts" 
          :key="product.id"
          class="product-card"
          @click="selectProduct(product)"
        >
          <div class="product-badge" v-if="product.anscienPrix">PROMO</div>
          <div class="product-image-container">
            <img
              v-if="product.image?.[0]?.url"
              :src="getStrapiImageUrl(product.image[0].url)"
              :alt="product.Nom"
              class="product-image"
              loading="lazy"
            />
            <div v-else class="image-placeholder">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
              <span>Image non disponible</span>
            </div>
            <button class="quick-view-btn">Voir plus</button>
          </div>
          <div class="product-info">
            <div class="product-category">{{ product.categorie?.Nom }}</div>
            <h3 class="product-name">{{ product.Nom }}</h3>
            
            <div class="price-container">
              <div class="current-price">{{ product.prix }} FCFA</div>
              <div class="old-price" v-if="product.anscienPrix">
                {{ product.anscienPrix }} FCFA
              </div>
            </div>
            
            <div class="product-actions">
              <button class="add-to-cart-btn" @click.stop="addToCart(product)">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                  <path d="M3 1a1 1 0 000 2h1.22l.305 1.222a.997.997 0 00.01.042l1.358 5.43-.893.892C3.74 11.846 4.632 14 6.414 14H15a1 1 0 000-2H6.414l1-1H14a1 1 0 00.894-.553l3-6A1 1 0 0017 3H6.28l-.31-1.243A1 1 0 005 1H3zM16 16.5a1.5 1.5 0 11-3 0 1.5 1.5 0 013 0zM6.5 18a1.5 1.5 0 100-3 1.5 1.5 0 000 3z" />
                </svg>
                Ajouter
              </button>
              <button class="wishlist-btn" @click.stop="toggleWishlist(product)">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const categories = ref([]);
const products = ref([]);
const selectedCategories = ref([]);
const loading = ref(true);
const error = ref(null);

const STRAPI_BASE_URL = 'https://kind-duck-a00ba31603.strapiapp.com';
const API_URL = `${STRAPI_BASE_URL}/api`;

// Fonction pour récupérer l'URL complète d'une image Strapi
const getStrapiImageUrl = (url) => {
  if (!url) return '';
  return url.startsWith('http') ? url : `${STRAPI_BASE_URL}${url}`;
};

// Computed pour filtrer les produits selon les catégories sélectionnées
const filteredProducts = computed(() => {
  if (selectedCategories.value.length === 0) return products.value;
  return products.value.filter(product => {
    const categoryId = product.category?.id;
    return categoryId && selectedCategories.value.includes(categoryId);
  });
});

// Récupération des catégories depuis Strapi
const fetchCategories = async () => {
  try {
    const res = await fetch(`${API_URL}/categories`);
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
    const json = await res.json();
    categories.value = json.data || json;
  } catch (err) {
    console.error('Erreur fetch catégories:', err);
    error.value = 'Impossible de charger les catégories';
  }
};

// Récupération des produits avec images
const fetchProductsWithImages = async () => {
  try {
    const res = await fetch(`${API_URL}/produits?populate=image`);
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
    const json = await res.json();
    return json.data || json;
  } catch (err) {
    console.error('Erreur fetch produits avec images:', err);
    throw err;
  }
};

// Récupération des produits avec catégories (ici 'category')
const fetchProductsWithCategories = async () => {
  try {
    const res = await fetch(`${API_URL}/produits?populate=category`);
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
    const json = await res.json();
    return json.data || json;
  } catch (err) {
    console.error('Erreur fetch produits avec catégories:', err);
    throw err;
  }
};

// Fusionner les données des produits avec images et catégories
const mergeProductsData = (productsWithImages, productsWithCategories) => {
  return productsWithImages.map(product => {
    const productWithCategory = productsWithCategories.find(p => p.id === product.id);

    return {
      id: product.id,
      Nom: product.Nom,
      prix: product.prix,
      anscienPrix: product.anscienPrix,
      image: product.image,
      category: productWithCategory?.category || null
    };
  });
};

// Chargement complet des données au montage du composant
const loadAllData = async () => {
  try {
    loading.value = true;

    // Chargement parallèle des produits (images + catégories)
    const [productsWithImages, productsWithCategories] = await Promise.all([
      fetchProductsWithImages(),
      fetchProductsWithCategories()
    ]);

    // Fusion des données
    products.value = mergeProductsData(productsWithImages, productsWithCategories);

    // Chargement des catégories séparément
    await fetchCategories();

  } catch (err) {
    console.error('Erreur chargement données:', err);
    error.value = err.message || 'Erreur lors du chargement des données';
  } finally {
    loading.value = false;
  }
};

// Fonctions interaction (exemple)
const selectProduct = (product) => {
  console.log('Produit sélectionné:', product);
};

const addToCart = (product, event) => {
  event.stopPropagation();
  console.log('Ajout au panier:', product);
};

const toggleWishlist = (product, event) => {
  event.stopPropagation();
  console.log('Wishlist:', product);
};

onMounted(loadAllData);
</script>


<style scoped>
.catalogue-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 30px 20px;
  display: flex;
  gap: 40px;
}

.filters-section {
  flex: 0 0 280px;
  background: #fff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  height: fit-content;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

.product-count {
  color: #666;
  font-size: 0.9rem;
}

.section-title {
  font-size: 1.8rem;
  font-weight: 700;
  color: #2c3e50;
  margin-bottom: 0;
}

.categories-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 20px;
}

.category-item {
  display: flex;
  align-items: center;
}

.category-checkbox {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  font-size: 0.95rem;
  color: #444;
  transition: color 0.2s;
}

.category-checkbox:hover {
  color: #2c3e50;
}

.checkmark {
  display: inline-block;
  width: 18px;
  height: 18px;
  border: 2px solid #ddd;
  border-radius: 4px;
  position: relative;
  transition: all 0.2s;
}

input[type="checkbox"] {
  display: none;
}

input[type="checkbox"]:checked + .checkmark {
  background-color: #2c3e50;
  border-color: #2c3e50;
}

.products-section {
  flex: 1;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 25px;
}

.product-card {
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0,0,0,0.05);
  transition: all 0.3s ease;
  cursor: pointer;
  position: relative;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}

.product-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #e74c3c;
  color: white;
  padding: 4px 10px;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
  z-index: 2;
}

.product-image-container {
  height: 220px;
  background: #f9f9f9;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
}

.product-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  transition: transform 0.5s ease;
  padding: 20px;
}

.product-card:hover .product-image {
  transform: scale(1.05);
}

.image-placeholder {
  color: #aaa;
  font-size: 0.9rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.image-placeholder svg {
  width: 40px;
  height: 40px;
  color: #ddd;
}

.quick-view-btn {
  position: absolute;
  bottom: -40px;
  left: 0;
  right: 0;
  background: rgba(44, 62, 80, 0.9);
  color: white;
  border: none;
  padding: 8px;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.3s ease;
  opacity: 0;
}

.product-card:hover .quick-view-btn {
  bottom: 0;
  opacity: 1;
}

.product-info {
  padding: 18px;
}

.product-category {
  font-size: 0.75rem;
  color: #7f8c8d;
  margin-bottom: 5px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.product-name {
  font-size: 1rem;
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 12px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  min-height: 40px;
}

.price-container {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}

.current-price {
  font-weight: 700;
  color: #2c3e50;
  font-size: 1.2rem;
}

.old-price {
  font-size: 0.9rem;
  color: #e74c3c;
  text-decoration: line-through;
}

.product-actions {
  display: flex;
  gap: 8px;
}

.add-to-cart-btn {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  background: #2c3e50;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s;
}

.add-to-cart-btn:hover {
  background: #1a252f;
}

.add-to-cart-btn svg {
  width: 16px;
  height: 16px;
}

.wishlist-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f8f9fa;
  border: 1px solid #eee;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.wishlist-btn:hover {
  background: #f1f3f5;
  border-color: #ddd;
}

.wishlist-btn svg {
  width: 18px;
  height: 18px;
  color: #7f8c8d;
}

.loading, .error {
  text-align: center;
  padding: 40px;
  font-size: 1.1rem;
}

.loading {
  color: #666;
}

.error {
  color: #e74c3c;
}

@media (max-width: 992px) {
  .catalogue-container {
    flex-direction: column;
    gap: 30px;
  }
  
  .filters-section {
    flex: 0 0 auto;
  }
  
  .products-grid {
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  }
}

@media (max-width: 576px) {
  .products-grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
  }
  
  .product-image-container {
    height: 180px;
  }
  
  .section-title {
    font-size: 1.5rem;
  }
}
</style>