<script setup>
import { ref, computed, onMounted } from 'vue'
import cocktailsData from '@/data/cocktails.json'
import ingredientsData from '@/data/ingredients.json'
import { mdiPlus, mdiCircleOutline, mdiCircle, mdiMagnify, mdiEmoticonSad, mdiClose } from '@mdi/js'

// État réactif pour les cocktails, ingrédients et filtres
const cocktails = ref([])
const ingredients = ref([])
const selectedIngredients = ref([])
const sournessRange = ref([0, 5])
const bitternessRange = ref([0, 5])
const sweetnessRange = ref([0, 5])
const onlyMocktails = ref(false)

// État pour le dialogue d'ajout
const showAddDialog = ref(false)
const newCocktail = ref({
  name: '',
  emoji: '🍻',
  ingredients: [{ ingredient: '', amount_ml: 0 }],
  sourness: 0,
  bitterness: 0,
  sweetness: 0,
  mocktail: false
})

// Charger les cocktails depuis localStorage ou utiliser les données par défaut
const loadCocktails = () => {
  const savedCocktails = localStorage.getItem('gocktail-cocktails')
  if (savedCocktails) {
    cocktails.value = JSON.parse(savedCocktails)
  } else {
    cocktails.value = [...cocktailsData]
    saveCocktails()
  }
}

// Charger les ingrédients depuis localStorage ou utiliser les données par défaut
const loadIngredients = () => {
  const savedIngredients = localStorage.getItem('gocktail-ingredients')
  if (savedIngredients) {
    ingredients.value = JSON.parse(savedIngredients)
  } else {
    ingredients.value = [...ingredientsData]
    saveIngredients()
  }
}

// Sauvegarder les cocktails dans localStorage
const saveCocktails = () => {
  localStorage.setItem('gocktail-cocktails', JSON.stringify(cocktails.value))
}

// Sauvegarder les ingrédients dans localStorage
const saveIngredients = () => {
  localStorage.setItem('gocktail-ingredients', JSON.stringify(ingredients.value))
}

// Ajouter de nouveaux ingrédients à la liste si ils n'existent pas
const addNewIngredients = (cocktailIngredients) => {
  let hasNewIngredients = false
  
  cocktailIngredients.forEach(cocktailIngredient => {
    const ingredientName = cocktailIngredient.ingredient.toLowerCase().trim()
    const exists = ingredients.value.some(ingredient => 
      ingredient.toLowerCase() === ingredientName
    )
    
    if (!exists && ingredientName) {
      ingredients.value.push(cocktailIngredient.ingredient.trim())
      hasNewIngredients = true
    }
  })
  
  if (hasNewIngredients) {
    // Trier les ingrédients par ordre alphabétique
    ingredients.value.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()))
    saveIngredients()
  }
}

// Ajouter un nouvel ingrédient à la recette
const addIngredient = () => {
  newCocktail.value.ingredients.push({ ingredient: '', amount_ml: 0 })
}

// Supprimer un ingrédient de la recette
const removeIngredient = (index) => {
  if (newCocktail.value.ingredients.length > 1) {
    newCocktail.value.ingredients.splice(index, 1)
  }
}

// Sauvegarder le nouveau cocktail
const saveCocktail = () => {
  // Valider que tous les champs sont remplis
  if (!newCocktail.value.name.trim()) return

  const validIngredients = newCocktail.value.ingredients.filter(
    ing => ing.ingredient.trim() && ing.amount_ml > 0
  )

  if (validIngredients.length === 0) return

  // Ajouter les nouveaux ingrédients à la liste globale
  addNewIngredients(validIngredients)

  // Créer le nouveau cocktail
  const cocktailToAdd = {
    ...newCocktail.value,
    name: newCocktail.value.name.trim(),
    ingredients: validIngredients
  }

  // Ajouter à la liste
  cocktails.value.push(cocktailToAdd)
  saveCocktails()

  // Réinitialiser le formulaire
  resetForm()
  showAddDialog.value = false
}

// Réinitialiser le formulaire
const resetForm = () => {
  newCocktail.value = {
    name: '',
    emoji: '🍻',
    ingredients: [{ ingredient: '', amount_ml: 0 }],
    sourness: 0,
    bitterness: 0,
    sweetness: 0,
    mocktail: false
  }
}

// Initialiser au montage du composant
onMounted(() => {
  loadCocktails()
  loadIngredients()
})

// Cocktails filtrés
const filteredCocktails = computed(() => {
  return cocktails.value.filter(cocktail => {
    // Filtre ingrédients
    if (selectedIngredients.value.length > 0) {
      const hasSelectedIngredients = selectedIngredients.value.every(ingredient =>
        cocktail.ingredients.some(cocktailIngredient =>
          cocktailIngredient.ingredient.toLowerCase().includes(ingredient.toLowerCase())
        )
      )
      if (!hasSelectedIngredients) return false
    }

    // Filtre sourness
    if (cocktail.sourness < sournessRange.value[0] || cocktail.sourness > sournessRange.value[1]) {
      return false
    }

    // Filtre bitterness
    if (cocktail.bitterness < bitternessRange.value[0] || cocktail.bitterness > bitternessRange.value[1]) {
      return false
    }

    // Filtre sweetness
    if (cocktail.sweetness < sweetnessRange.value[0] || cocktail.sweetness > sweetnessRange.value[1]) {
      return false
    }

    // Filtre mocktail
    if (onlyMocktails.value && !cocktail.mocktail) {
      return false
    }

    return true
  })
})

// Fonction pour obtenir l'emoji approprié pour chaque cocktail
const getCocktailEmoji = (cocktail) => {
  return cocktail.emoji || '🍻'
}

// Fonction pour obtenir la couleur basée sur les caractéristiques
const getFlavorColor = (sourness, bitterness, sweetness) => {
  if (sweetness >= 4) return '#FF6B9D'
  if (bitterness >= 4) return '#8B4513'
  if (sourness >= 4) return '#FFD700'
  return '#4FC3F7'
}
</script>

<template>
  <div class="cocktail-app">
    <!-- Header avec titre -->
    <div class="app-header">
      <h1 class="app-title">
        🍸 Gocktail 🍹
      </h1>
      <p class="app-subtitle">Découvrez votre cocktail parfait</p>
    </div>

    <v-container fluid class="pa-4">
      <!-- Panneau de filtres avec style glassmorphism -->
      <v-card class="glass-card mb-6" elevation="0">
        <v-card-title class="text-h5 text-center pb-2">
          <v-icon :icon="mdiMagnify" class="mr-2"></v-icon>
          Filtres de recherche
        </v-card-title>

        <v-card-text>
          <v-row>
            <!-- Sélection d'ingrédients -->
            <v-col cols="12" md="6">
              <v-select hide-details v-model="selectedIngredients" :items="ingredients" label="🧪 Ingrédients" multiple
                chips closable-chips variant="outlined" class="glass-input">
                <template v-slot:prepend-item>
                  <v-list-item>
                    <v-list-item-title class="text-caption">
                      Sélectionnez les ingrédients que vous avez
                    </v-list-item-title>
                  </v-list-item>
                  <v-divider></v-divider>
                </template>
              </v-select>
            </v-col>

            <!-- Switch mocktail -->
            <v-col cols="12" md="6" class="d-flex align-center">
              <v-switch v-model="onlyMocktails" label="🚫 Seulement les mocktails" color="primary"
                hide-details></v-switch>
            </v-col>
          </v-row>

          <v-row class="mt-2">
            <!-- Slider Acidité -->
            <v-col cols="12" md="4">
              <div class="slider-container">
                🍋
                <span class="slider-label ml-1">Acidité</span>
              </div>
              <v-range-slider hide-details v-model="sournessRange" :min="0" :max="5" step="1" thumb-label
                color="yellow-darken-2" track-color="yellow-lighten-4" class="glass-slider">
                <template v-slot:thumb-label="{ modelValue }">
                  {{ modelValue }}
                </template>
              </v-range-slider>
            </v-col>

            <!-- Slider Amertume -->
            <v-col cols="12" md="4">
              <div class="slider-container">
                ☕
                <span class="slider-label ml-1">Amertume</span>
              </div>
              <v-range-slider hide-details v-model="bitternessRange" :min="0" :max="5" step="1" thumb-label
                color="brown-darken-2" track-color="brown-lighten-4" class="glass-slider">
                <template v-slot:thumb-label="{ modelValue }">
                  {{ modelValue }}
                </template>
              </v-range-slider>
            </v-col>

            <!-- Slider Douceur -->
            <v-col cols="12" md="4">
              <div class="slider-container">
                🍯
                <span class="slider-label ml-1">Douceur</span>
              </div>
              <v-range-slider hide-details v-model="sweetnessRange" :min="0" :max="5" step="1" thumb-label
                color="pink-darken-2" track-color="pink-lighten-4" class="glass-slider">
                <template v-slot:thumb-label="{ modelValue }">
                  {{ modelValue }}
                </template>
              </v-range-slider>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>

      <!-- Résultats -->
      <div class="results-section">
        <h2 class="text-h4 mb-4 text-center">
          🎯 {{ filteredCocktails.length }} cocktail{{ filteredCocktails.length > 1 ? 's' : '' }} trouvé{{
            filteredCocktails.length > 1 ? 's' : '' }}
        </h2>

        <v-row>
          <v-col v-for="cocktail in filteredCocktails" :key="cocktail.name" cols="12" sm="6" md="4" lg="3">
            <v-card class="glass-card cocktail-card" elevation="0"
              :style="{ borderColor: getFlavorColor(cocktail.sourness, cocktail.bitterness, cocktail.sweetness) }">
              <v-card-title class="text-center pb-2">
                <div class="cocktail-emoji">{{ getCocktailEmoji(cocktail) }}</div>
                <div class="cocktail-name">{{ cocktail.name }}</div>
                <v-chip v-if="cocktail.mocktail" color="success" size="small" class="mt-1">
                  🚫 Sans alcool
                </v-chip>
              </v-card-title>

              <v-card-text>
                <!-- Recette -->
                <div class="recipe-section mb-3">
                  <h4 class="recipe-title">📝 Recette</h4>
                  <div class="ingredients-list">
                    <div v-for="ingredient in cocktail.ingredients" :key="ingredient.ingredient"
                      class="ingredient-item">
                      <span class="ingredient-name">{{ ingredient.ingredient }}</span>
                      <span class="ingredient-amount">{{ ingredient.amount_ml }}ml</span>
                    </div>
                  </div>
                </div>

                <!-- Caractéristiques gustatives -->
                <div class="flavor-profile">
                  <div class="flavor-item">
                    <span class="flavor-icon">🍋</span>
                    <span class="flavor-label">Acidité:</span>
                    <v-rating :model-value="cocktail.sourness" readonly size="small" color="yellow-darken-2"
                      :empty-icon="mdiCircleOutline" :full-icon="mdiCircle"></v-rating>
                  </div>

                  <div class="flavor-item">
                    <span class="flavor-icon">☕</span>
                    <span class="flavor-label">Amertume:</span>
                    <v-rating :model-value="cocktail.bitterness" readonly size="small" color="brown-darken-2"
                      :empty-icon="mdiCircleOutline" :full-icon="mdiCircle"></v-rating>
                  </div>

                  <div class="flavor-item">
                    <span class="flavor-icon">🍯</span>
                    <span class="flavor-label">Douceur:</span>
                    <v-rating :model-value="cocktail.sweetness" readonly size="small" color="pink-darken-2"
                      :empty-icon="mdiCircleOutline" :full-icon="mdiCircle"></v-rating>
                  </div>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Message si aucun résultat -->
        <div v-if="filteredCocktails.length === 0" class="text-center mt-8">
          <v-icon size="64" color="grey-lighten-1" :icon="mdiEmoticonSad"></v-icon>
          <h3 class="text-h5 mt-4 text-grey-darken-1">
            Aucun cocktail ne correspond à vos critères
          </h3>
          <p class="text-body-1 text-grey-darken-1 mt-2">
            Essayez de modifier vos filtres pour découvrir de nouvelles recettes !
          </p>
        </div>
      </div>
    </v-container>

    <!-- Floating Action Button -->
    <v-fab :icon="mdiPlus" location="bottom end" size="large" color="primary" @click="showAddDialog = true"
      class="fab-button rounded-circle"></v-fab>

    <!-- Dialog pour ajouter un cocktail -->
    <v-dialog v-model="showAddDialog" max-width="600px" persistent>
      <v-card class="glass-card">
        <v-card-title class="text-h5 text-center pb-2">
          <v-icon :icon="mdiPlus" class="mr-2"></v-icon>
          Ajouter un nouveau cocktail
        </v-card-title>

        <v-card-text>
          <v-form>
            <v-row>
              <!-- Nom du cocktail -->
              <v-col cols="12" md="8">
                <v-text-field v-model="newCocktail.name" label="Nom du cocktail" variant="outlined" class="glass-input"
                  hide-details required></v-text-field>
              </v-col>

              <!-- Emoji -->
              <v-col cols="12" md="4">
                <v-text-field v-model="newCocktail.emoji" label="Emoji" variant="outlined" class="glass-input"
                  hide-details maxlength="2"></v-text-field>
              </v-col>

              <!-- Switch mocktail -->
              <v-col cols="12">
                <v-switch v-model="newCocktail.mocktail" label="🚫 Sans alcool (mocktail)" color="primary"
                  hide-details></v-switch>
              </v-col>

              <!-- Ingrédients -->
              <v-col cols="12">
                <h4 class="recipe-title mb-3">📝 Ingrédients</h4>
                <div v-for="(ingredient, index) in newCocktail.ingredients" :key="index" class="ingredient-row mb-3">
                  <v-row>
                    <v-col cols="12" md="8">
                      <v-text-field v-model="ingredient.ingredient" label="Ingrédient" variant="outlined"
                        class="glass-input" hide-details required></v-text-field>
                    </v-col>
                    <v-col cols="10" md="3">
                      <v-text-field v-model="ingredient.amount_ml" label="Quantité (ml)" type="number"
                        variant="outlined" class="glass-input" hide-details required></v-text-field>
                    </v-col>
                    <v-col cols="2" md="1" class="d-flex align-center">
                      <v-btn v-if="newCocktail.ingredients.length > 1" :icon="mdiClose" size="small" color="error"
                        variant="text" @click="removeIngredient(index)"></v-btn>
                    </v-col>
                  </v-row>
                </div>
                <v-btn @click="addIngredient" color="primary" variant="outlined" size="small" class="mb-4">
                  <v-icon :icon="mdiPlus" class="mr-1"></v-icon>
                  Ajouter un ingrédient
                </v-btn>
              </v-col>

              <!-- Caractéristiques gustatives -->
              <v-col cols="12">
                <h4 class="recipe-title mb-3">🎯 Profil gustatif</h4>

                <!-- Acidité -->
                <div class="mb-4">
                  <div class="slider-container">
                    🍋
                    <span class="slider-label ml-1">Acidité: {{ newCocktail.sourness }}</span>
                  </div>
                  <v-slider v-model="newCocktail.sourness" :min="0" :max="5" step="1" thumb-label
                    color="yellow-darken-2" track-color="yellow-lighten-4" class="glass-slider" hide-details></v-slider>
                </div>

                <!-- Amertume -->
                <div class="mb-4">
                  <div class="slider-container">
                    ☕
                    <span class="slider-label ml-1">Amertume: {{ newCocktail.bitterness }}</span>
                  </div>
                  <v-slider v-model="newCocktail.bitterness" :min="0" :max="5" step="1" thumb-label
                    color="brown-darken-2" track-color="brown-lighten-4" class="glass-slider" hide-details></v-slider>
                </div>

                <!-- Douceur -->
                <div class="mb-4">
                  <div class="slider-container">
                    🍯
                    <span class="slider-label ml-1">Douceur: {{ newCocktail.sweetness }}</span>
                  </div>
                  <v-slider v-model="newCocktail.sweetness" :min="0" :max="5" step="1" thumb-label color="pink-darken-2"
                    track-color="pink-lighten-4" class="glass-slider" hide-details></v-slider>
                </div>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>

        <v-card-actions class="pa-4">
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="outlined" @click="showAddDialog = false; resetForm()">
            Annuler
          </v-btn>
          <v-btn color="primary" variant="elevated" @click="saveCocktail"
            :disabled="!newCocktail.name.trim() || newCocktail.ingredients.every(ing => !ing.ingredient.trim() || ing.amount_ml <= 0)">
            Ajouter
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<style scoped>
.cocktail-app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  position: relative;
}

.cocktail-app::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><defs><pattern id="grain" width="100" height="100" patternUnits="userSpaceOnUse"><circle cx="25" cy="25" r="1" fill="rgba(255,255,255,0.1)"/><circle cx="75" cy="75" r="1" fill="rgba(255,255,255,0.1)"/><circle cx="50" cy="10" r="0.5" fill="rgba(255,255,255,0.05)"/><circle cx="20" cy="80" r="0.5" fill="rgba(255,255,255,0.05)"/></pattern></defs><rect width="100" height="100" fill="url(%23grain)"/></svg>');
  pointer-events: none;
  z-index: 1;
}

.app-header {
  text-align: center;
  padding: 2rem 0;
  position: relative;
  z-index: 2;
}

.app-title {
  font-size: 3.5rem;
  font-weight: 700;
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  margin: 0;
  letter-spacing: 0.05em;
}

.app-subtitle {
  font-size: 1.2rem;
  color: rgba(255, 255, 255, 0.9);
  margin: 0.5rem 0 0 0;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.glass-card {
  background: rgba(255, 255, 255, 0.1) !important;
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px !important;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  position: relative;
  z-index: 2;
  transition: all 0.3s ease;
}

.glass-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.cocktail-card {
  border-left: 4px solid;
  height: 100%;
}

.cocktail-emoji {
  font-size: 3rem;
  margin-bottom: 0.5rem;
}

.cocktail-name {
  font-size: 1.2rem;
  font-weight: 600;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.glass-input {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
}

.glass-input :deep(.v-field) {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 15px;
}

.slider-container {
  display: flex;
  align-items: center;
  margin-bottom: 0.5rem;
}

.slider-label {
  font-weight: 600;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.glass-slider {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 0.5rem;
  backdrop-filter: blur(10px);
}

.flavor-profile {
  margin-top: 1rem;
}

.recipe-section {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  padding: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.recipe-title {
  color: white;
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 0.75rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.ingredients-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.ingredient-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.4rem 0.6rem;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.ingredient-name {
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.85rem;
  font-weight: 500;
  flex: 1;
  text-transform: capitalize;
}

.ingredient-amount {
  color: white;
  font-size: 0.8rem;
  font-weight: 600;
  background: rgba(255, 255, 255, 0.15);
  padding: 0.2rem 0.5rem;
  border-radius: 12px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.flavor-item {
  display: flex;
  align-items: center;
  margin-bottom: 0.5rem;
  gap: 0.5rem;
}

.flavor-icon {
  font-size: 1.1rem;
  width: 24px;
}

.flavor-label {
  font-size: 0.85rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.9);
  min-width: 70px;
}

.results-section {
  position: relative;
  z-index: 2;
}

.results-section h2 {
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  font-weight: 600;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .app-title {
    font-size: 2.5rem;
  }

  .cocktail-emoji {
    font-size: 2.5rem;
  }

  .flavor-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.25rem;
  }

  .flavor-label {
    min-width: auto;
  }
}

/* Animation pour les cartes */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.cocktail-card {
  animation: fadeInUp 0.5s ease-out;
}

/* Amélioration des chips */
:deep(.v-chip) {
  background: rgba(255, 255, 255, 0.2) !important;
  color: white !important;
  border: 1px solid rgba(255, 255, 255, 0.3) !important;
  backdrop-filter: blur(10px);
}

:deep(.v-chip--variant-outlined) {
  background: rgba(255, 255, 255, 0.1) !important;
}

/* Style pour les ratings */
:deep(.v-rating .v-icon) {
  margin: 0 1px;
}

/* Amélioration du switch */
:deep(.v-switch) {
  color: white;
}

:deep(.v-switch .v-label) {
  color: white !important;
  font-weight: 500;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

/* Floating Action Button */
.fab-button {
  position: fixed !important;
  bottom: 24px;
  right: 24px;
  z-index: 1000;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3) !important;
  backdrop-filter: blur(10px);
}

.fab-button:hover {
  transform: scale(1.1);
  transition: transform 0.2s ease;
}

/* Dialog styles */
:deep(.v-dialog .v-card) {
  backdrop-filter: blur(20px) !important;
}

.ingredient-row {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  padding: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

/* Amélioration des inputs dans le dialog */
:deep(.v-dialog .glass-input .v-field) {
  background: rgba(255, 255, 255, 0.15) !important;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3) !important;
  border-radius: 15px;
}

:deep(.v-dialog .glass-input .v-field-label) {
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.v-dialog .glass-input .v-field__input) {
  color: white !important;
}

/* Amélioration des sliders dans le dialog */
:deep(.v-dialog .glass-slider) {
  background: rgba(255, 255, 255, 0.15);
  border-radius: 10px;
  padding: 0.5rem;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* Mobile adjustments for FAB */
@media (max-width: 768px) {
  .fab-button {
    bottom: 16px;
    right: 16px;
  }
}
</style>
