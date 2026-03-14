<template>
  <article class="card" :class="{ 'card--out': product.stock <= 0, 'is-fav': isFavorite }">
    <div class="card__top">
      <span class="badge" v-if="product.badge">{{ product.badge }}</span>
      <span v-else class="badge">{{ product.stock > 0 ? 'API' : 'SOLD' }}</span>

      <button class="icon-btn" @click="$emit('toggle-fav', product.id)" aria-label="Favorito">
        {{ isFavorite ? '♥' : '♡' }}
      </button>
    </div>

    <div class="card__media">
      <img
        v-if="product.image"
        class="card__img"
        :src="product.image"
        :alt="product.name"
        loading="lazy"
      />
      <div v-else class="monogram">
        {{ product.name.charAt(0).toUpperCase() }}
      </div>
    </div>

    <div class="card__body">
      <h3 class="card__title">{{ product.name }}</h3>

      <div class="card__meta">
        <span class="price">{{ formatMoney(product.price) }}</span>
        <span class="dot">•</span>
        <span class="muted">{{ product.category }}</span>
      </div>

      <p class="muted" style="margin:0 0 12px; line-height:1.4;">
        {{ shortDescription }}
      </p>

      <div class="card__controls">
        <div class="size">
          <label class="muted" for="size">Talla</label>
          <select class="select select--mini" v-model="selectedSize">
            <option>S</option>
            <option>M</option>
            <option>L</option>
            <option>XL</option>
          </select>
        </div>

        <div class="actions">
          <button
            class="btn btn--accent btn--mini"
            :disabled="product.stock <= 0"
            @click="handleAdd"
          >
            {{ product.stock > 0 ? 'Agregar' : 'Sin stock' }}
          </button>
        </div>
      </div>
    </div>
  </article>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

interface Product {
  id: number
  name: string
  category: string
  price: number
  featured: number
  stock: number
  tags: string[]
  badge: string
  image: string
  description: string
}

export default defineComponent({
  name: 'ProductCard',
  props: {
    product: {
      type: Object as () => Product,
      required: true,
    },
    theme: {
      type: String,
      required: true,
    },
    isFavorite: {
      type: Boolean,
      required: true,
    },
  },
  emits: ['add', 'toggle-fav'],
  data() {
    return {
      selectedSize: 'M',
    }
  },
  computed: {
    shortDescription(): string {
      if (!this.product.description) return 'Producto obtenido desde API pública.'
      return this.product.description.length > 90
        ? this.product.description.slice(0, 90) + '...'
        : this.product.description
    },
  },
  methods: {
    formatMoney(value: number): string {
      return value.toLocaleString('es-CO', { style: 'currency', currency: 'COP' })
    },
    handleAdd() {
      this.$emit('add', {
        ...this.product,
        size: this.selectedSize,
      })
    },
  },
})
</script>
