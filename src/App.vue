<template>
  <div id="app">
    <header class="topbar">
      <div class="container topbar__inner">
        <div class="brand">
          <div class="brand__mark" aria-hidden="true"></div>
          <div>
            <div class="brand__name">API STORE</div>
            <div class="brand__tag">Catálogo de productos con Vue y API pública</div>
          </div>
        </div>

        <div class="topbar__actions">
          <button class="btn btn--ghost" @click="toggleTheme">
            <span v-if="theme === 'dark'">☾ Dark</span>
            <span v-else>☼ Light</span>
          </button>

          <button class="btn btn--accent" @click="cartOpen = true">
            Cart
            <span class="pill" v-if="cartCount > 0">{{ cartCount }}</span>
            <span class="pill" v-else>0</span>
          </button>
        </div>
      </div>
    </header>

    <section class="hero">
      <div class="container hero__inner">
        <div class="hero__copy">
          <h1>Catálogo interactivo consumiendo una API pública.</h1>
          <p>
            Aplicación desarrollada en <b>Vue.js</b> que consulta productos en tiempo real
            desde una <b>API pública</b> y los muestra en una interfaz moderna y responsive.
          </p>
          <div class="hero__cta">
            <button class="btn btn--accent" @click="scrollToProducts">Ver productos</button>
            <button class="btn btn--ghost" @click="cycleAccent">Cambiar acento</button>
          </div>
          <div class="chips">
            <span class="chip">API pública</span>
            <span class="chip">Diseño responsive</span>
            <span class="chip">Carrito interactivo</span>
          </div>
        </div>

        <div class="hero__panel">
          <div class="stat">
            <div class="stat__label">Selección</div>
            <div class="stat__value">{{ filteredProducts.length }} productos</div>
          </div>
          <div class="stat">
            <div class="stat__label">En carrito</div>
            <div class="stat__value">{{ cartCount }} items</div>
          </div>
          <div class="stat">
            <div class="stat__label">Total</div>
            <div class="stat__value">{{ formatMoney(cartTotal) }}</div>
          </div>
        </div>
      </div>
    </section>

    <section class="controls" ref="productsSection">
      <div class="container controls__inner">
        <div class="search">
          <input
            class="input"
            type="text"
            placeholder="Buscar productos..."
            v-model="search"
          />
        </div>

        <div class="filters">
          <button class="seg" :class="{ 'seg--active': category === 'all' }" @click="category = 'all'">Todo</button>
          <button class="seg" :class="{ 'seg--active': category === 'mens-shirts' }" @click="category = 'mens-shirts'">Camisas</button>
          <button class="seg" :class="{ 'seg--active': category === 'mens-shoes' }" @click="category = 'mens-shoes'">Zapatos</button>
          <button class="seg" :class="{ 'seg--active': category === 'womens-bags' }" @click="category = 'womens-bags'">Bolsos</button>

          <label class="check">
            <input type="checkbox" v-model="onlyInStock" />
            <span>Solo stock</span>
          </label>

          <select class="select" v-model="sortBy">
            <option value="featured">Destacados</option>
            <option value="priceAsc">Precio: menor a mayor</option>
            <option value="priceDesc">Precio: mayor a menor</option>
            <option value="nameAsc">Nombre A–Z</option>
          </select>
        </div>
      </div>

      <div class="container results-bar">
        <p class="muted">Mostrando {{ filteredProducts.length }} producto(s)</p>
      </div>
    </section>

    <main class="products">
      <div class="container">
        <div v-if="loading" class="empty">
          <h3>Cargando productos...</h3>
          <p>Espera un momento mientras consultamos la API pública.</p>
        </div>

        <div v-else-if="error" class="empty">
          <h3>Error al cargar productos</h3>
          <p>{{ error }}</p>
          <button class="btn btn--ghost" @click="fetchProducts">Reintentar</button>
        </div>

        <div v-else class="grid">
          <ProductCard
            v-for="p in filteredProducts"
            :key="p.id"
            :product="p"
            :theme="theme"
            :is-favorite="favorites.has(p.id)"
            @add="addToCart"
            @toggle-fav="toggleFav"
          />
        </div>

        <div class="empty" v-if="!loading && !error && filteredProducts.length === 0">
          <h3>No hay productos con ese filtro.</h3>
          <p>Prueba cambiando categoría o buscando otra palabra.</p>
          <button class="btn btn--ghost" @click="resetFilters">Reset filtros</button>
        </div>
      </div>
    </main>

    <div class="drawer-backdrop" v-if="cartOpen" @click.self="cartOpen = false">
      <aside class="drawer">
        <div class="drawer__top">
          <h2>Tu carrito</h2>
          <button class="icon-btn" @click="cartOpen = false" aria-label="Cerrar">✕</button>
        </div>

        <div class="drawer__content">
          <div v-if="cart.length === 0" class="cart-empty">
            <p>Tu carrito está vacío.</p>
            <button class="btn btn--accent" @click="cartOpen = false">Explorar productos</button>
          </div>

          <div v-else class="cart-list">
            <div class="cart-item" v-for="item in cart" :key="item.id + '-' + item.size">
              <div class="cart-item__thumb">
                <img v-if="item.image" class="cart-item__img" :src="item.image" :alt="item.name" loading="lazy" />
                <span v-else aria-hidden="true">{{ item.name.charAt(0).toUpperCase() }}</span>
              </div>
              <div class="cart-item__info">
                <div class="cart-item__name">
                  {{ item.name }}
                  <span class="mini" v-if="item.size">• Talla {{ item.size }}</span>
                </div>
                <div class="cart-item__meta">
                  <span class="price">{{ formatMoney(item.price) }}</span>
                  <span class="dot">•</span>
                  <span class="muted">x{{ item.qty }}</span>
                </div>
                <div class="qty">
                  <button class="qty__btn" @click="decQty(item.id)">−</button>
                  <span class="qty__num">{{ item.qty }}</span>
                  <button class="qty__btn" @click="incQty(item.id)">+</button>
                  <button class="link danger" @click="removeFromCart(item.id)">Eliminar</button>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="drawer__bottom">
          <div class="summary">
            <div class="row">
              <span class="muted">Subtotal</span>
              <b>{{ formatMoney(cartTotal) }}</b>
            </div>
            <div class="row">
              <span class="muted">Envío</span>
              <b>{{ formatMoney(shippingCost) }}</b>
            </div>
            <div class="row total">
              <span>Total</span>
              <b>{{ formatMoney(cartTotal + shippingCost) }}</b>
            </div>
          </div>

          <button class="btn btn--accent btn--full" :disabled="cart.length === 0" @click="checkout">
            Comprar ahora
          </button>
          <button class="btn btn--ghost btn--full" @click="clearCart" :disabled="cart.length === 0">
            Vaciar carrito
          </button>
        </div>
      </aside>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import ProductCard from './components/ProductCard.vue'

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

interface CartItem extends Product {
  qty: number
  size: string
}

export default defineComponent({
  name: 'App',
  components: { ProductCard },

  data() {
    return {
      theme: 'dark' as string,
      accentIndex: 0 as number,
      accents: ['#ff5a1f', '#ff7a00', '#ff3d2e'] as string[],
      search: '' as string,
      category: 'all' as string,
      onlyInStock: false as boolean,
      sortBy: 'featured' as string,
      cartOpen: false as boolean,
      favorites: new Set<number>(),
      products: [] as Product[],
      cart: [] as CartItem[],
      loading: false as boolean,
      error: '' as string,
    }
  },

  async mounted() {
    document.documentElement.setAttribute('data-theme', this.theme)
    document.documentElement.style.setProperty('--accent', this.accent)
    await this.fetchProducts()
  },

  watch: {
    theme(newTheme: string) {
      document.documentElement.setAttribute('data-theme', newTheme)
    },
    accent(newAccent: string) {
      document.documentElement.style.setProperty('--accent', newAccent)
    },
  },

  computed: {
    accent(): string {
      return this.accents[this.accentIndex % this.accents.length] ?? '#ff5a1f'
    },
    cartCount(): number {
      return this.cart.reduce((sum, it) => sum + it.qty, 0)
    },
    cartTotal(): number {
      return this.cart.reduce((sum, it) => sum + it.price * it.qty, 0)
    },
    shippingCost(): number {
      return this.cartTotal >= 250000 || this.cartTotal === 0 ? 0 : 12000
    },
    filteredProducts(): Product[] {
      const q = this.search.trim().toLowerCase()
      const list = this.products.filter((p) => {
        const matchCategory = this.category === 'all' || p.category === this.category
        const matchStock = !this.onlyInStock || p.stock > 0
        const matchSearch =
          q === '' ||
          p.name.toLowerCase().includes(q) ||
          p.tags.some((t) => t.toLowerCase().includes(q)) ||
          p.description.toLowerCase().includes(q)

        return matchCategory && matchStock && matchSearch
      })

      switch (this.sortBy) {
        case 'priceAsc':
          list.sort((a, b) => a.price - b.price)
          break
        case 'priceDesc':
          list.sort((a, b) => b.price - a.price)
          break
        case 'nameAsc':
          list.sort((a, b) => a.name.localeCompare(b.name))
          break
        default:
          list.sort((a, b) => a.featured - b.featured)
      }

      return list
    },
  },

  methods: {
    async fetchProducts() {
      this.loading = true
      this.error = ''

      try {
        const res = await fetch('https://dummyjson.com/products/category/mens-shirts')
        const res2 = await fetch('https://dummyjson.com/products/category/mens-shoes')
        const res3 = await fetch('https://dummyjson.com/products/category/womens-bags')

        if (!res.ok || !res2.ok || !res3.ok) {
          throw new Error('No se pudieron cargar los productos')
        }

        const data1 = await res.json()
        const data2 = await res2.json()
        const data3 = await res3.json()

        const combined = [...data1.products, ...data2.products, ...data3.products]

        this.products = combined.map((item: any, index: number) => ({
          id: item.id,
          name: item.title,
          category: item.category,
          price: Math.round(item.price * 4000),
          featured: index + 1,
          stock: item.stock,
          tags: item.tags ?? [],
          badge: item.stock === 0 ? 'SOLD' : index < 3 ? 'NEW' : '',
          image: item.thumbnail,
          description: item.description ?? '',
        }))
      } catch (err) {
        this.error = 'Hubo un problema cargando la API pública.'
        console.error(err)
      } finally {
        this.loading = false
      }
    },
    formatMoney(value: number): string {
      return value.toLocaleString('es-CO', { style: 'currency', currency: 'COP' })
    },
    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark'
    },
    cycleAccent() {
      this.accentIndex++
    },
    resetFilters() {
      this.search = ''
      this.category = 'all'
      this.onlyInStock = false
      this.sortBy = 'featured'
    },
    scrollToProducts() {
      const el = this.$refs.productsSection as HTMLElement
      if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' })
    },
    toggleFav(productId: number) {
      if (this.favorites.has(productId)) this.favorites.delete(productId)
      else this.favorites.add(productId)
      this.favorites = new Set(this.favorites)
    },
    addToCart(product: Product & { size: string }) {
      if (product.stock <= 0) return
      const found = this.cart.find((it) => it.id === product.id && it.size === product.size)
      if (found) {
        if (found.qty < product.stock) found.qty++
      } else {
        this.cart.push({ ...product, qty: 1, size: product.size || 'M' })
      }
    },
    incQty(productId: number) {
      const item = this.cart.find((it) => it.id === productId)
      const product = this.products.find((p) => p.id === productId)
      if (!item || !product) return
      if (item.qty < product.stock) item.qty++
    },
    decQty(productId: number) {
      const item = this.cart.find((it) => it.id === productId)
      if (!item) return
      item.qty--
      if (item.qty <= 0) this.removeFromCart(productId)
    },
    removeFromCart(productId: number) {
      this.cart = this.cart.filter((it) => it.id !== productId)
    },
    clearCart() {
      this.cart = []
    },
    checkout() {
      alert(`Compra simulada ✅\nTotal: ${this.formatMoney(this.cartTotal + this.shippingCost)}`)
      this.clearCart()
      this.cartOpen = false
    },
  },
})
</script>
