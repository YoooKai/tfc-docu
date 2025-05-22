
# DEW

He utilizado **Vue.js** como framework principal para el desarrollo del frontend, aprovechando su estructura basada en componentes para construir una interfaz modular, clara y fácilmente mantenible. Cada funcionalidad está dividida en componentes reutilizables que encapsulan lógica, estilos y estructura, lo que facilita la escalabilidad y mejora la organización del código.

Durante el desarrollo, se contempló inicialmente la integración con **TypeScript** para aprovechar sus beneficios en cuanto a tipado estático y detección temprana de errores. No obstante, debido a problemas de compatibilidad al migrar componentes ya desarrollados en JavaScript y a restricciones de tiempo antes de la entrega, se optó por continuar con JavaScript puro para garantizar la estabilidad y funcionalidad del proyecto.

El proyecto hace un uso intensivo de **Pinia** como gestor de estado global, lo que permite centralizar y sincronizar datos críticos como la gestión del carrito de compras, el estado de autenticación y el control de carga, proporcionando una experiencia de usuario fluida y reactiva.

Entre las funcionalidades destacadas implementadas con Vue, se incluyen:

- Gestión dinámica de productos con opciones configurables y cantidades ajustables, integrando validaciones y actualización automática de precios.
- Visualización avanzada de imágenes mediante una galería modal con navegación, para mejorar la experiencia visual.
- Persistencia del estado crítico (carrito, sesión, usuario) mediante almacenamiento local (`localStorage`), lo que garantiza la continuidad de la sesión y los datos entre recargas o cierres del navegador.
- Uso de métodos y propiedades computadas para mantener la lógica de negocio desacoplada de la presentación y optimizar el rendimiento.

Como mejora futura, se plantea una migración progresiva a TypeScript, comenzando por los componentes y módulos más críticos y con mayor lógica, para reforzar la robustez del código sin comprometer la base establecida.

### Dependencias (`dependencies`)

- **vue** (`^3.2.13`)
    
    Framework progresivo para construir interfaces de usuario. Versión 3, la más reciente y con Composition API.
    
- **vue-router** (`^4.0.3`)
    
    Librería oficial para manejar rutas y navegación en aplicaciones Vue 3.
    
- **pinia** (`^3.0.2`)
    
    Alternativa moderna y más ligera a Vuex para gestión de estado en Vue 3.
    
- **axios** (`^1.8.4`)
    
    Cliente HTTP para hacer peticiones AJAX a APIs, muy popular y sencillo de usar.
    
- **aos** (`^2.3.4`)
    
    Animate On Scroll, librería para animar elementos cuando aparecen al hacer scroll en la página.
    
- **core-js** (`^3.8.3`)
    
    Biblioteca para polyfills de JavaScript moderno, asegurando compatibilidad con navegadores antiguos.
    

---

**Conexión de vue al backend de Django:**
 Se realiza la conexión del frontend (Vue) con el backend (Django REST Framework) a través de la librería `axios`. Las rutas protegidas están autenticadas con tokens JWT, y los datos de usuarios, productos y carrito son intercambiados entre cliente y servidor de forma segura y asincrónica.

### Componentes y vistas

Este proyecto cuenta con una serie de componentes y vistas:

**Vistas:** About, Cart, Category, CategoryView, Checkout, Comissions, Contact, Events, FAQ, Home, LogIn, MyAccount,Portfolio, PostDetail, PostList, Product, Search, SignUp, Store, Success.

**Componentes:** AuthModals, BaseTitle, CardPost, CartItem, CategorySidebar, Marquee, Modal, Navbar, OrderSummary, ProductBox, ProductGrid, Socials, SolidButton.

A continuación hablaré de cada uno de ellos:

### **Navbar.vue (componente)**

Este componente gestiona la barra de navegación, mostrando enlaces a las distintas vistas, el carrito y los controles de autenticación. Además, al montarse carga el token de la store y lo añade a los headers de Axios para que todas las llamadas al backend Django incluyan autenticación.

- **Composition API & SFC**: usa `<script setup>` con `ref`, `computed` y `onMounted`.
    
    ```jsx
    <script setup>
    import { ref, computed, onMounted } from 'vue'
    // …
    const showLogin = ref(false)
    const cart = computed(() => mainStore.cart)
    onMounted(() => {
      const token = mainStore.token
      axios.defaults.headers.common['Authorization'] = token
        ? 'Token ' + token
        : ''
    })
    </script>
    
    ```
    
- **Conexión a Django**: inyecta el token en Axios para llamadas al API de Django.
    
    ```jsx
    axios.defaults.headers.common['Authorization'] = 'Token ' + token
    
    ```
    
- **Pinia (composable)**: importa y utiliza la store (`useMainStore`) para estado global (auth, carrito).
    
    ```jsx
    import { useMainStore } from '../store/mainstore'
    const mainStore = useMainStore()
    
    ```
    
- **Vue Router**: usa `useRouter()` y `<router-link>` para navegación SPA.
    
    ```jsx
    <router-link class="navbar-brand" to="/">TangerineMess.</router-link>
    
    ```
    
- **Directivas**: muestra/oculta elementos según `isAuthenticated` con `v-if`, controla clics con `@click`.
    
    ```html
    <template v-if="isAuthenticated">
      <router-link …>Mi cuenta</router-link>
    </template>
    <button @click="showLogin = true">…</button>
    
    ```
    

---

**Home.vue (vista)**

Pantalla de portada con un “hero” a toda pantalla, un título y dos componentes hijos (`Marquee` y `Socials`). Define el título de la pestaña del navegador y usa SVG para una curva decorativa.

- **SFC**: todo en un `.vue` con `<template>`, `<script>` y `<style scoped>`.
- **Components**: importa y registra `Marquee` y `Socials`.
    
    ```
    <script>
    import Marquee from '@/components/Marquee.vue'
    import Socials from '@/components/Socials.vue'
    document.title = 'Home | TangerineMess'
    export default { components: { Marquee, Socials } }
    </script>
    
    ```
    
- **Estilos scoped**: aplica estilos locales al hero y al texto con sombras CSS.

---

**Marquee.vue (componente)**

Genera una cinta de texto que se desplaza indefinidamente, adaptando la duración de la animación al ancho del contenido.

- **SFC & Options API**: usa `mounted` y `beforeUnmount` para hooks de ciclo de vida, y `methods` para lógica.
- **Refs con `$refs`**: accede directamente a los nodos DOM para medir ancho y posicionar el segundo bloque.
    
    ```
    mounted() {
      this.adjustAnimation()
      window.addEventListener('resize', this.adjustAnimation)
    },
    methods: {
      adjustAnimation() {
        const width = this.$refs.marquee1.offsetWidth
        // …
        this.$refs.marquee2.style.left = `${width}px`
      }
    }
    
    ```
    
- **Animación dinámica**: inyecta un bloque `<style>` en el `<head>` con `@keyframes` que trasladan el texto según el ancho calculado.

---

### Login.vue

Este componente implementa la funcionalidad de inicio de sesión para la aplicación. Utiliza la Options API de Vue para manejar el estado local y métodos. Cuando el usuario envía el formulario, se hace una petición POST al endpoint `/api/v1/token/login/` de la API Django REST para autenticar. Si la autenticación es exitosa, se recibe un token que se almacena en `localStorage` y en un store global con Pinia (`useMainStore`). Además, el token se configura en los headers de Axios para futuras peticiones autenticadas. Se redirige al usuario a la ruta `/store`. En caso de error, se muestran los mensajes recibidos desde el backend.

Se usan datos reactivos para el formulario (`username`, `password`) y para almacenar errores que se presentan en la UI. La interacción con el store global permite centralizar el estado del usuario.

Fragmento clave del método que envía el formulario:

```jsx
async submitForm() {
  this.errors = []
  axios.defaults.headers.common["Authorization"] = ""
  localStorage.removeItem("token")

  const formData = { username: this.username, password: this.password }
  try {
    const response = await axios.post("/api/v1/token/login/", formData)
    const token = response.data.auth_token

    localStorage.setItem("username", this.username)
    localStorage.setItem("token", token)
    this.mainStore.setUsername(this.username)
    this.mainStore.setToken(token)
    axios.defaults.headers.common["Authorization"] = "Token " + token
    this.mainStore.initializeStore()

    this.$router.push('/store')
  } catch (error) {
    if (error.response) {
      for (const property in error.response.data) {
        this.errors.push(`${property}: ${error.response.data[property]}`)
      }
    } else {
      this.errors.push('Something went wrong. Please try again')
    }
  }
}

```

---

### SignUp.vue

Este componente permite registrar un nuevo usuario. Incluye un formulario con campos para usuario, email, contraseña y confirmación de contraseña. Antes de enviar, se valida en el cliente que no falten campos y que las contraseñas coincidan, mostrando errores si es necesario.

Al enviar el formulario, se realiza una petición POST al endpoint `/api/v1/users/` para crear el usuario en el backend. Si la petición es exitosa, se emite un evento hacia el componente padre para indicar que el registro fue exitoso y cambiar la vista a login. En caso de errores de validación o de servidor, los mensajes se muestran en pantalla.

Fragmento del método que valida y envía los datos:

```jsx
async submitForm() {
  this.errors = []

  if (!this.username) this.errors.push('El nombre de usuario es obligatorio')
  if (!this.email) this.errors.push('El email es obligatorio')
  if (!this.password) this.errors.push('La contraseña es obligatoria')
  if (this.password !== this.password2) this.errors.push('Las contraseñas no coinciden')

  if (this.errors.length) return

  try {
    await axios.post('/api/v1/users/', {
      username: this.username,
      email: this.email,
      password: this.password
    })
    this.$emit('signup-success')
  } catch (error) {
    if (error.response) {
      for (const property in error.response.data) {
        this.errors.push(`${property}: ${error.response.data[property]}`)
      }
    } else {
      this.errors.push('Algo salió mal. Inténtalo de nuevo.')
    }
  }
}

```

---

### Modal.vue

Un componente modal reutilizable que sirve como contenedor para mostrar contenido sobre una capa oscura en la pantalla. Recibe una propiedad booleana `show` que controla la visibilidad. Cuando el usuario hace click fuera del contenido (overlay) o en el botón de cerrar, se emite un evento `close` para que el componente padre controle el estado y oculte el modal.

Usa un slot para permitir que cualquier contenido dinámico se inserte dentro del modal, por ejemplo, los formularios de login y signup. El modal está estilizado con bordes redondeados, sombra y animación de aparición.

Código clave:

```jsx
<template>
  <div v-if="show" class="modal-overlay" @click.self="close">
    <div class="modal-content cartoon-card animate-pop">
      <button class="modal-close" @click="close"><i class="fas fa-times"></i></button>
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Modal',
  props: {
    show: { type: Boolean, required: true }
  },
  emits: ['close'],
  methods: {
    close() {
      this.$emit('close')
    }
  }
}
</script>

```

---

### AuthModals.vue

Este componente es el encargado de controlar cuál modal se muestra: el de login o el de registro (signup). Mantiene un estado local `showSignUp` que indica qué modal debe estar visible. Importa y usa los componentes `Modal`, `LogIn` y `SignUp`.

Cuando se quiere cambiar entre login y signup, se invocan métodos que actualizan `showSignUp`. Además, maneja el evento de registro exitoso para cerrar el modal de signup, abrir el de login y mostrar un toast con Bootstrap para notificar al usuario que la cuenta fue creada.

El componente también incluye el contenedor para los toasts de Bootstrap, y genera dinámicamente los toasts para los mensajes.

Fragmento que controla el estado y muestra los modales:

```jsx
data() {
  return {
    showSignUp: false
  }
},
methods: {
  switchToSignUp() {
    this.showSignUp = true
  },
  switchToLogin() {
    this.showSignUp = false
  },
  handleSignupSuccess() {
    this.showSignUp = false
    this.showBootstrapToast('¡Cuenta creada! Ahora inicia sesión.', 'success')
  },
  showBootstrapToast(message, type = 'success') {
    const toastContainer = document.getElementById('toast-container')
    if (!toastContainer) return

    const toastEl = document.createElement('div')
    toastEl.className = `toast align-items-center text-white bg-primary border-0`
    toastEl.style.minWidth = '300px'
    toastEl.style.fontSize = '1.1rem'

    toastEl.setAttribute('role', 'alert')
    toastEl.setAttribute('aria-live', 'assertive')
    toastEl.setAttribute('aria-atomic', 'true')
    toastEl.innerHTML = `
      <div class="d-flex">
        <div class="toast-body fs-5">${message}</div>
        <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
      </div>
    `
    toastContainer.appendChild(toastEl)
    const bsToast = new bootstrap.Toast(toastEl, { delay: 3000 })
    bsToast.show()
    toastEl.addEventListener('hidden.bs.toast', () => { toastEl.remove() })
  }
}

```

---

### Store.vue

Este componente es la página principal de la tienda. Su estructura HTML está dividida en un sidebar con las categorías y un área principal con la búsqueda y el listado de productos.

- Usa un componente `CategoriesSidebar` para mostrar la lista de categorías disponibles, destacando la categoría activa.
- Tiene un formulario de búsqueda que redirige a la ruta `/search` enviando el término en query string.
- Los productos más recientes se cargan desde la API (`/api/v1/latest-products/`) al montar el componente, almacenándose en `latestProducts`.
- Usa el componente `ProductsGrid` para renderizar la grilla de productos.
- Además, usa un store global `useMainStore` para manejar el estado de carga (`isLoading`) mientras se obtiene la información.

Fragmento clave de la llamada a la API para obtener productos recientes:

```jsx
async fetchLatestProducts() {
  const store = useMainStore()
  store.setIsLoading(true)

  try {
    const res = await axios.get("/api/v1/latest-products/")
    this.latestProducts = res.data
  } catch (error) {
    console.error(error)
  }

  store.setIsLoading(false)
}

```

El CSS se encarga de estilos responsivos para el formulario de búsqueda y la disposición general, con colores personalizados.

---

### CategoriesSidebar.vue

Este componente es una barra lateral que lista las categorías de productos disponibles para filtrar.

- Recibe por prop `activeCategorySlug` para saber qué categoría está activa y aplicar estilos especiales.
- Usa enlaces `router-link` para navegar entre categorías, cada uno con una ruta distinta.
- Tiene estilos personalizados para los enlaces, con efecto hover y resaltado para la categoría activa.
- Emplea el componente `BaseTitle` para mostrar un título estilizado "Categorías".
- El método `isActive(slug)` devuelve true si el slug coincide con la categoría activa, para aplicar la clase CSS `active`.

Fragmento del método que determina si una categoría está activa:

```jsx
methods: {
  isActive(slug) {
    return this.activeCategorySlug === slug;
  }
}

```

Estilísticamente destaca la usabilidad con colores, transición y tipografía custom.

---

### Search.vue

Este componente muestra los resultados de búsqueda de productos basados en el término que el usuario ingresó.

- Obtiene el parámetro de búsqueda desde la URL con `useRoute` y lo almacena en una variable reactiva `query`.
- Cada vez que cambia `query` o al montar el componente, se hace una petición POST al backend (`/api/v1/products/search/`) enviando el término de búsqueda.
- Los productos encontrados se almacenan en `products`.
- Si no hay productos, se muestra un mensaje de aviso.
- Se usan componentes `ProductBox` para mostrar cada producto, `BaseTitle` para el encabezado y `SolidButton` para un botón que vuelve a la tienda completa.

Fragmento que realiza la búsqueda:

```jsx
async function performSearch() {
  try {
    const response = await axios.post('/api/v1/products/search/', { query: query.value })
    products.value = response.data
  } catch (error) {
    console.error(error)
  }
}

```

Además, hay un watcher que responde a cambios en la query para actualizar los resultados sin recargar.

---

### ProductBox.vue

Componente que representa una tarjeta individual de producto.

- Muestra la imagen principal, el nombre y el precio del producto.
- El nombre y la imagen son enlaces que llevan a la página del producto (usando `product.get_absolute_url`).
- La imagen tiene un efecto de zoom al pasar el mouse.
- La tarjeta tiene efectos visuales para mejorar la experiencia, como sombra y desplazamiento al hacer hover.
- Estilizado con colores cálidos y tipografías definidas para mantener el estilo visual del sitio.

Fragmento de la estructura principal:

```jsx
<div class="card h-100 border-0 product-card">
  <router-link :to="product.get_absolute_url" class="image-link">
    <img :src="product.get_thumbnail" alt="Product image" class="card-img-top" />
  </router-link>
  <div class="card-body d-flex flex-column px-2 py-2">
    <router-link :to="product.get_absolute_url" class="product-name text-uppercase fw-bold mb-2 mt-2">
      {{ product.name }}
    </router-link>
    <p class="product-price mt-auto mb-3">{{ product.price }} €</p>
  </div>
</div>

```

---

### ProductsGrid.vue

Este componente es un contenedor que recibe un array de productos y renderiza una grilla con tarjetas `ProductBox`.

- Recibe por prop `products` la lista de productos.
- Usa un `v-for` para iterar y renderizar cada producto.
- Facilita la reutilización y organización, permitiendo mostrar grillas de productos en cualquier parte del proyecto.

Fragmento de la plantilla:

```jsx
<div class="row g-4 container-custom">
  <ProductBox
    v-for="product in products"
    :key="product.id"
    :product="product"
  />
</div>

```

No tiene lógica extra, solo la estructura y la importación de `ProductBox`.

---

### **`Category`**

Este componente es como la ventana que muestra los productos de una categoría específica. Cuando entras, hace una llamada al backend con Axios para traer la info: el nombre de la categoría y sus productos.

- Usa `useMainStore` (Pinia) para manejar un loading que le avisa al usuario que está cargando datos.
- Escucha cuando cambias de categoría en la URL y automáticamente recarga los datos.
- También actualiza el título de la pestaña del navegador con el nombre de la categoría.

Un pedacito importante:

```jsx
async getCategory() {
  const categorySlug = this.$route.params.category_slug
  store.setIsLoading(true)
  try {
    const response = await axios.get(`/api/v1/products/${categorySlug}/`)
    this.category = response.data
    document.title = this.category.name + ' | TangerinMess'
  } catch (error) {
    toast({ message: 'Algo salió mal, intenta de nuevo.', type: 'is-danger' })
  }
  store.setIsLoading(false)
}

```

En resumen: cuando abres una categoría, este componente se encarga de mostrar el título y una lista de productos con un componente `ProductBox` que recibe cada producto como prop.

---

### **`CategoryView`**

Esta vista es como la versión más completa y pulida del `Category`. Aquí tienes el sidebar con todas las categorías para navegar, un buscador para filtrar productos y el listado de productos en un grid.

- El sidebar se hace con un componente `CategoriesSidebar` que recibe todas las categorías.
- El buscador está preparado para enviar la consulta a la ruta `/search`.
- Si la categoría no tiene productos, muestra un mensaje amigable avisando que no hay productos.
- Igual que en el otro, hace la llamada con Axios para traer la categoría y productos y actualiza el título.

Un fragmento clave del template para el buscador y productos:

```jsx
<form method="get" action="/search" class="mb-4 search-form ms-auto">
  <div class="input-group">
    <input type="text" class="form-control" placeholder="¿Qué estás buscando?" name="query" />
    <button class="btn btn-custom" type="submit">
      <i class="fas fa-search"></i>
    </button>
  </div>
</form>

<div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 row-cols-xl-4 g-4">
  <ProductBox
    v-for="product in category.products"
    :key="product.id"
    :product="product"
  />
</div>

<div v-if="category.products.length === 0" class="alert alert-warning text-center mt-4">
  No hay productos disponibles en esta categoría.
</div>

```

Este componente te da la experiencia completa para navegar, buscar y visualizar productos en una categoría, con estilos responsivos y sidebar para facilitar la navegación.

---

### **`Contact`**

Aquí tienes el formulario para que los usuarios puedan enviarte mensajes o pedir colaboraciones. La magia es que todo se maneja con Vue: el formulario está ligado con `v-model` para capturar cada dato, y cuando envías, hace un POST a la API con Axios.

- Si el envío es exitoso, muestra un toast verde con el mensaje de éxito.
- Si falla, muestra un toast rojo con el error.
- También tienes enlaces a redes sociales con iconos muy visibles para que te encuentren fácilmente.
- El formulario es súper sencillo, con validaciones básicas (todos los campos son requeridos).

Este método muestra cómo se hace el envío y el manejo de los mensajes:

```jsx
async handleSubmit() {
  try {
    const response = await axios.post('http://127.0.0.1:8000/api/v1/contact/', this.formData)
    this.message = response.data.message || '¡Mensaje enviado correctamente!'
    this.messageType = 'success'
    this.resetForm()
  } catch (error) {
    this.message = 'Error al enviar. Intenta de nuevo.'
    this.messageType = 'error'
  }
  this.showToast = true
  setTimeout(() => { this.showToast = false }, 4000)
},

```

En pocas palabras: este componente da una experiencia limpia y sencilla para que cualquier persona pueda contactarte fácilmente, con mensajes claros de confirmación o error.

Claro, aquí te dejo una documentación más personal, explicando qué hace cada componente (Cart, CartItem y Checkout) con fragmentos relevantes de código para que lo entiendas bien y puedas usarlo o modificarlo con confianza.

---

# **Cart.vue**

Este es el contenedor principal del carrito de compras. Su función es mostrar todos los productos que el usuario ha agregado, mostrar el resumen del pedido con el subtotal y total, y ofrecer un botón para ir al checkout.

- Muestra una lista de productos con el componente `CartItem`.
- Calcula y muestra el total del pedido.
- Permite eliminar productos del carrito usando el método `removeFromCart` del store.
- Si el carrito está vacío, muestra un mensaje informativo.

```jsx
<CartItem
  v-for="item in cart.items"
  :key="item.product.id"
  :item="item"
  @removeFromCart="removeFromCart"
/>

<div class="summary-row">
  <span>Subtotal</span>
  <span>{{ cartTotalPrice.toFixed(2) }} €</span>
</div>

<SolidButton
  text="Ir a la caja"
  iconClass="fas fa-arrow-right"
  routeTo="/cart/checkout"
/>

```

### 

---

## **CartItem.vue**

Este componente representa cada producto dentro del carrito. Se encarga de mostrar detalles del producto, su imagen, precio, opción seleccionada, cantidad y el total de ese producto. Además, tiene controles para incrementar o decrementar la cantidad, y eliminar el producto.

### 

- Muestra información clave del producto.
- Permite cambiar la cantidad con botones + y −.
- Elimina el producto del carrito si la cantidad baja a cero o si el usuario lo elimina explícitamente.
- Actualiza el localStorage para mantener la persistencia del carrito.

### 

```jsx
<button class="qty-btn" @click="decrementQuantity(item)" :disabled="item.quantity <= 1">−</button>
<span class="quantity">{{ item.quantity }}</span>
<button class="qty-btn" @click="incrementQuantity(item)">+</button>

<span class="total">{{ getItemTotal(item).toFixed(2) }}€</span>

function incrementQuantity(item) {
  item.quantity += 1
  mainStore.updateLocalStorageCart()
}

function decrementQuantity(item) {
  if (item.quantity > 1) {
    item.quantity -= 1
    mainStore.updateLocalStorageCart()
  } else {
    removeFromCart(item)
  }
}

```

### 

Este componente es muy útil porque permite al usuario controlar exactamente qué quiere comprar, y cuántos. Además, mantiene todo sincronizado con el store y el almacenamiento local, asegurando que los datos no se pierdan.

---

## **Checkout.vue**

Esta es la página de pago, donde el usuario introduce sus datos de envío y puede revisar el pedido antes de confirmar el pago.

- Muestra un resumen detallado del pedido, con productos, cantidades, precios, peso y coste de envío.
- Calcula el peso total con el sobre y ajusta el coste de envío según ese peso.
- Recoge la información del usuario (nombre, email, dirección, etc.) para el envío.
- Valida que todos los campos estén completos.
- Integra Stripe para gestionar el pago con tarjeta.

### 

```jsx
<tbody>
  <tr v-for="item in cart.items" :key="item.product.id">
    <td>{{ item.product.name }} <span class="option-highlight"> - {{ item.product.selectedOption?.name || 'N/A' }}</span></td>
    <td class="price-text">{{ formatPrice(item.product.price) }}</td>
    <td>{{ item.quantity }}</td>
    <td class="price-text">{{ formatPrice(getItemTotal(item)) }}</td>
  </tr>
</tbody>

const totalWeightWithEnvelope = computed(() => {
  const baseEnvelopeWeight = 30
  const totalProductWeight = store.cart.items.reduce((acc, item) => {
    return acc + item.quantity * (item.product.totalWeight || 0)
  }, 0)
  return baseEnvelopeWeight + totalProductWeight
})

const shippingCost = computed(() => {
  const weight = totalWeightWithEnvelope.value
  if (weight <= 500) return 5
  else if (weight <= 1000) return 8
  return 12
})

function submitForm() {
  errors.value = []

  if (!first_name.value) errors.value.push('The first name field is missing!')
  // ... validations para otros campos

  if (!errors.value.length) {
    store.setIsLoading(true)

    stripe.value.createToken(card.value).then(result => {
      if (result.error) {
        store.setIsLoading(false)
        errors.value.push('Something went wrong with Stripe. Please try again')
      } else {
        stripeTokenHandler(result.token)
      }
    })
  }
}

```

Esta pantalla es fundamental para cerrar la compra. Combina la validación del formulario, cálculo de costes y la integración con Stripe para que todo el proceso sea seguro y sencillo para el usuario.

```jsx
function incrementQuantity(item) {
  item.quantity += 1
  mainStore.updateLocalStorageCart()
}

function decrementQuantity(item) {
  if (item.quantity > 1) {
    item.quantity -= 1
    mainStore.updateLocalStorageCart()
  } else {
    removeFromCart(item)
  }
}

```

Este código permite aumentar o disminuir la cantidad del producto y asegura que los datos se mantengan guardados para que el usuario no pierda el carrito al recargar la página.

---

## MyAccount

Este componente Vue usa SFC y mezcla Options API con Composition API para acceder a un store Pinia (`useMainStore`). Se conecta al backend mediante llamadas axios para obtener los pedidos y maneja la sesión usando localStorage y Vue Router para la navegación.

- **Conexión al backend (Django REST API):**
    
    Se realiza una llamada a la API para obtener pedidos:
    
    ```jsx
    async getMyOrders() {
      const response = await axios.get('/api/v1/orders/')
      this.orders = response.data
    }
    
    ```
    
- **Uso parcial de Composition API y SFC:**
    
    El componente es un SFC y usa setup() para Pinia:
    
    ```jsx
    setup() {
      const mainStore = useMainStore()
      return { mainStore }
    }
    
    ```
    
- **Uso de directivas Vue:**
    
    En template se usan `v-if`, `v-for` y eventos con `@click`:
    
    ```jsx
    <button @click="logout">Salir</button>
    <div v-for="order in orders" :key="order.id">...</div>
    <p v-if="username">Hola, {{ username }}</p>
    
    ```
    
- **Uso de Pinia (store) para estado global:**
    
    ```jsx
    import { useMainStore } from '../store/mainstore'
    
    ```
    
- **Uso de localStorage para mantener sesión y logout:**
    
    ```jsx
    logout() {
      localStorage.removeItem("token")
      localStorage.removeItem("username")
      localStorage.removeItem("userid")
    }
    
    ```
    
- **Uso de Vue Router para redireccionar tras logout:**
    
    ```jsx
    this.$router.push('/')
    
    ```
    

---

## OrderSummary

Este componente Vue recibe una prop `order` y muestra los detalles del pedido. Está en formato SFC y usa solo Options API con propiedades computadas para cálculos y formato.

- **Uso de props:**
    
    Recibe la orden como prop:
    
    ```jsx
    props: {
      order: Object
    }
    
    ```
    
- **Uso de computed properties:**
    
    Para formatear fecha y calcular totales:
    
    ```jsx
    computed: {
      formattedOrderDate() {
        const date = new Date(this.order.created_at)
        return date.toLocaleDateString('es-ES')
      },
      orderTotalPrice() {
        return this.order.items.reduce((total, item) => total + Number(item.price) * item.quantity, 0)
      }
    }
    
    ```
    
- **Uso de métodos para cálculos y formateo:**
    
    ```jsx
    methods: {
      getItemTotal(item) {
        return item.quantity * item.product.price
      },
      formatPrice(value) {
        return new Intl.NumberFormat('es-ES', { style: 'currency', currency: 'EUR' }).format(value)
      }
    }
    
    ```
    
- **Uso de directivas Vue:**
    
    `v-for` para listar productos:
    
    ```jsx
    <tr v-for="item in order.items" :key="item.product.id">
    
    ```
    

---

## Success

Componente simple SFC que muestra mensaje estático. Usa Options API y Vue Router para cambiar título en `mounted`. Se usa para redirigir a esta vista cuando se confirma una compra.

- **Uso de lifecycle hook para document title:**
    
    ```jsx
    mounted() {
      document.title = 'Success '
    }
    
    ```
    

---

## **BaseTitle**

Este componente es un título reutilizable y personalizable para mostrar encabezados en la aplicación.

- **Qué hace:**
    
    Renderiza un `<h2>` con texto dinámico recibido por props y permite personalizar el color, la fuente y el tamaño del texto mediante props también. Esto facilita que el mismo componente pueda usarse en distintos contextos con estilos diferentes sin modificar el componente base.
    
- **Características importantes:**
    - Recibe las props `text` (obligatoria), `color`, `font` y `size`.
    - Usa binding dinámico en el atributo `style` para aplicar los estilos recibidos.
    - Está estructurado como un Single File Component (SFC) típico en Vue.
- **Código clave:**

```jsx
props: {
  text: { type: String, required: true },
  color: { type: String, default: 'var(--color-secondary)' },
  font: { type: String, default: "'ObviouslyNarrow', sans-serif" },
  size: { type: String, default: '3.6rem' }
}

```

- **Uso del binding dinámico:**

```jsx
<h2 :style="{ color: color, fontFamily: font, fontSize: size }">{{ text }}</h2>

```

---

## **CardPost**

Este componente representa una tarjeta que muestra un post o artículo de blog con su título, extracto, fecha, género y una imagen.

- **Qué hace:**
    
    Muestra la información básica de un post en un formato visual atractivo, con imagen, fecha resaltada y un enlace para ir al post completo. Al pasar el cursor sobre la tarjeta, la imagen y el contenido cambian suavemente, mejorando la experiencia visual.
    
- **Características importantes:**
    - Recibe varias props para su contenido (`title`, `excerpt`, `date`, `genre`, `image`, `slug`).
    - Utiliza **computed properties** para:
        - Separar la fecha en día, mes y año (`dateParts`).
        - Mostrar una imagen por defecto si no se pasa ninguna (`imageToShow`).
    - Usa `<router-link>` para enlazar a la página del post, integrándose con Vue Router para navegación SPA.
    - Tiene efectos visuales en hover mediante CSS para mejorar la interacción.
- **Código clave:**

```jsx
props: {
  title: String,
  excerpt: String,
  date: String,
  genre: String,
  image: String,
  slug: { type: String, required: true }
},
computed: {
  dateParts() { return this.date.split(' '); },
  imageToShow() { return this.image || 'https://via.placeholder.com/300x200?text=No+Image'; }
}

```

- **Enlace dinámico con router:**

```jsx
<router-link :to="`/blog/posts/${slug}`" class="read-more">Leer más</router-link>

```

---

## **SocialLinks**

Componente simple que muestra iconos sociales con enlaces a diferentes plataformas.

- **Qué hace:**
    
    Renderiza una barra de iconos que enlazan a redes sociales específicas. Es un componente estático sin lógica compleja, ideal para colocarse en pies de página o secciones de contacto.
    
- **Características importantes:**
    - Usa etiquetas `<a>` con atributos para accesibilidad (`aria-label`) y seguridad (`rel="noopener"`).
    - Contiene iconos con clases de FontAwesome.
    - Está estructurado como un SFC básico sin props ni estados.
- **Código clave:**

```jsx
<a href="https://www.instagram.com/..." target="_blank" rel="noopener" aria-label="Instagram">
  <i class="fab fa-instagram"></i>
</a>

```

---

## **SolidButton**

Botón reutilizable con texto, icono y acción para volver o navegar a una ruta específica.

- **Qué hace:**
    
    Muestra un botón con un ícono (por defecto una flecha hacia la izquierda) y un texto. Al hacer clic, navega programáticamente a la ruta especificada (por defecto `/posts`). Es útil para acciones de navegación dentro de la app.
    
- **Características importantes:**
    - Recibe las props `text`, `iconClass` y `routeTo` para personalización.
    - Usa el método `goBack` que utiliza Vue Router (`this.$router.push`) para cambiar de ruta sin recargar la página.
    - Tiene estilos para efectos hover y desplazamiento del icono.
- **Código clave:**

```jsx
methods: {
  goBack() {
    this.$router.push(this.routeTo);
  }
}

```

- **Uso de evento en template:**

```jsx
<button @click="goBack">
  <i :class="iconClass"></i>
  <span>{{ text }}</span>
</button>

```

---

## **About**

Página que muestra información "Sobre mí" obtenida desde un backend.

- **Qué hace:**
    
    Al montarse, hace una petición HTTP usando axios para obtener contenido de la sección “About” desde una API REST. Si hay contenido, lo muestra con texto e imagen. Si no, muestra un mensaje de alerta.
    
- **Características importantes:**
    - Usa el ciclo de vida `mounted()` para hacer la llamada axios.
    - Guarda el contenido en `data` y actualiza el DOM con `v-if` para condicionar la presentación.
    - Cambia el título del documento para SEO y usabilidad.
    - Método para convertir rutas relativas de imágenes a URLs absolutas.
- **Código clave:**

```jsx
async getAboutContent() {
  try {
    const response = await axios.get('/api/v1/about/');
    this.aboutContent = response.data;
  } catch (error) {
    console.error('Error fetching about content:', error);
  }
}

```

- **Condicional en template:**

```jsx
<div v-if="aboutContent">
  <p>{{ aboutContent.content }}</p>
  <img :src="getImageUrl(aboutContent.image)" alt="About Illustration" />
</div>
<div v-else>
  <p>About content is not available.</p>
</div>

```

---

## **Commissions**

Página que muestra una lista de comisiones disponibles, obtenidas desde backend.

- **Qué hace:**
    
    Hace una petición HTTP para obtener una lista de comisiones, luego las renderiza en una lista con imagen, descripción, precio y slots disponibles. Usa diseño responsivo y alterna el orden de imagen/texto en cada elemento para variedad visual.
    
- **Características importantes:**
    - Usa `axios` para cargar datos desde API en `mounted()`.
    - Usa `v-for` para iterar dinámicamente las comisiones.
    - Utiliza `v-html` para renderizar contenido HTML seguro dentro de la descripción.
    - Cambia el orden de elementos con clases dinámicas para que la presentación sea más atractiva.
    - Método para generar URLs absolutas de imágenes.
- **Código clave:**

```jsx
async getCommissions() {
  try {
    const response = await axios.get('/api/v1/commissions/');
    this.commissions = response.data;
  } catch (error) {
    console.error('Error fetching commissions:', error);
  }
}

```

- **Iteración con v-for y clases dinámicas:**

```jsx
<div v-for="(commission, index) in commissions" :key="commission.id" :class="{ 'flex-row-reverse': index % 2 === 1 }">
  <!-- contenido -->
</div>

```

---

### 

# `EventsView.vue`

Este componente Vue muestra una lista de próximos eventos obtenidos desde un backend (API REST de Django) usando Axios para hacer la petición.

Se estructura como un **Single File Component (SFC)** con su template, script y estilos encapsulado

---

- **Carga de eventos al montar el componente:**
    
    En el ciclo de vida `mounted()`, se llama al método `getEvents()` que hace una petición GET y actualiza el array reactivo `events`:
    

```jsx
mounted() {
  this.getEvents();
  document.title = 'Upcoming Events';
},
methods: {
  async getEvents() {
    try {
      const response = await axios.get('/api/v1/events/');
      this.events = response.data;
    } catch (error) {
      console.error('Error fetching events:', error);
    }
  },
  ...
}

```

- **Renderizado dinámico con directivas:**
    
    Se usa `v-if` para mostrar una alerta cuando no hay eventos:
    

```jsx
v v-if="events.length === 0" class="alert alert-warning" role="alert">
  No hay futuros eventos actualmente.
</div>

```

Y `v-for` para listar eventos en tarjetas:

```html
or="event in events" :key="event.id" class="col-md-6 mb-5">
  <div class="card shadow border-0 overflow-hidden flex-md-row h-100">
    <img :src="getImageUrl(event.image)" :alt="event.title" class="event-img" />
    <div class="p-4 d-flex flex-column justify-content-between flex-grow-1">
      <h3 class="event-title">{{ event.title }}</h3>
      <p class="event-location mb-3">{{ event.location }}</p>
    </div>
  </div>
</div>

```

- **Métodos para formateo y construcción de URLs:**

```jsx
methods: {
  getImageUrl(imagePath) {
    return `http://localhost:8000${imagePath}`;
  },
  formatShortDayMonth(dateStr) {
    const date = new Date(dateStr);
    return date.toLocaleDateString('es-ES', { day: '2-digit', month: 'short' });
  },
  getYear(dateStr) {
    return new Date(dateStr).getFullYear();
  },
  ...
}

```

## `FAQView.vue`

Este componente muestra una lista de preguntas frecuentes (FAQ) obtenidas desde un backend Django mediante Axios.

Se estructura como un **Single File Component (SFC)** y utiliza las funcionalidades básicas de Vue para mostrar/ocultar respuestas con interacción del usuario.

---

### Funcionalidades principales

- **Carga de FAQs al montar el componente:**
    
    En `mounted()`, se obtiene la lista de preguntas frecuentes desde la API y se inicializa cada item con la propiedad reactiva `isOpen` para controlar la visibilidad de la respuesta.
    

```jsx
async getFAQ() {
  try {
    const response = await axios.get('/api/v1/faq/');
    this.faq = response.data.map(item => ({ ...item, isOpen: false }));
  } catch (error) {
    console.error('Error fetching FAQs:', error);
  }
},
mounted() {
  this.getFAQ();
  document.title = 'Preguntas Frecuentes';
},

```

- **Toggle para mostrar/ocultar respuestas:**
    
    Al hacer clic en una pregunta, se cambia el estado `isOpen` del item correspondiente para mostrar u ocultar la respuesta.
    

```
toggleAnswer(id) {
  const selectedItem = this.faq.find(item => item.id === id);
  if (selectedItem) {
    selectedItem.isOpen = !selectedItem.isOpen;
  }
}

```

- **Uso de directivas y bindings:**
    - `v-for` para listar las preguntas
    - `v-if` para mostrar la respuesta solo si está abierta
    - `@click` para manejar la interacción
    - `:class` para cambiar el estilo según el estado

```html
<div
  v-for="item in faq"
  :key="item.id"
  class="faq-item p-3 mb-3 rounded"
  @click="toggleAnswer(item.id)"
  :class="{ 'open': item.isOpen }"
>
  <div class="d-flex justify-content-between align-items-center faq-question">
    <h4 class="mb-0">{{ item.question }}</h4>
    <i :class="['fas', item.isOpen ? 'fa-chevron-up' : 'fa-chevron-down']" class="faq-icon"></i>
  </div>
  <div v-if="item.isOpen" class="faq-answer mt-2">
    {{ item.answer }}
  </div>
</div>

```

---

### `PostsView.vue`

Este componente muestra una lista filtrable de posts obtenidos desde un backend Django usando Axios.

Se utiliza un dropdown para seleccionar la categoría y filtrar los posts mostrados. Los posts se visualizan en tarjetas mediante el componente hijo `CardPost`.

---

### Funcionalidades principales

- **Carga inicial de posts:**
    
    Al montar el componente, se cargan todos los posts disponibles desde la API.
    

```jsx
async getPosts() {
  try {
    const response = await axios.get('/api/v1/posts/');
    this.posts = response.data;
    this.filteredPosts = this.posts;
  } catch (error) {
    console.error('Error fetching posts:', error);
  }
},
mounted() {
  this.getPosts();
  document.title = 'Posts | TangerineMess';
},

```

- **Filtrado por categoría:**
    
    Al seleccionar una categoría en el dropdown, se consulta la API con un parámetro para filtrar posts y actualizar la lista visible.
    

```jsx
async filterByCategory() {
  try {
    const response = await axios.get('/api/v1/posts/', {
      params: { category: this.selectedCategory },
    });
    this.filteredPosts = response.data;
  } catch (error) {
    console.error('Error fetching filtered posts:', error);
  }
},

selectCategory(code) {
  if (this.selectedCategory !== code) {
    this.selectedCategory = code;
    this.filterByCategory();
  }
},

```

- **Renderizado con `v-for` y bindings:**
    
    Se muestran tarjetas para cada post filtrado, pasando props con información relevante.
    

```html
<div class="row g-4 mt-5">
  <div class="col-12 col-md-6 col-lg-4" v-for="post in filteredPosts" :key="post.id">
    <CardPost
      :title="post.title"
      :excerpt="post.summary"
      :date="formatDate(post.created_at)"
      :genre="post.category_display"
      :image="getImageUrl(post.images?.[0]?.image)"
      :slug="post.slug"
    />
  </div>
</div>

```

- **Transformación y formato de datos:**
    - Método para formatear fechas a un estilo corto y claro.
    - Método para construir URLs completas de imágenes o usar un placeholder.

```
formatDate(dateStr) {
  const date = new Date(dateStr);
  const day = String(date.getDate()).padStart(2, '0');
  const month = date.toLocaleString('default', { month: 'short' }).toUpperCase();
  const year = date.getFullYear();
  return `${day} ${month} ${year}`;
},

getImageUrl(imagePath) {
  return imagePath ? `http://localhost:8000${imagePath}` : 'https://via.placeholder.com/300x200';
},

```

---

## `PostDetail.vue`

Este componente muestra los detalles completos de un post específico, recuperando los datos del backend mediante el parámetro `slug` de la URL.

---

### Funcionalidades principales

- **Carga de datos según slug de ruta:**
    
    En `created()`, se obtiene el post correspondiente con el slug extraído del router.
    

```
fetchPost() {
  const slug = this.$route.params.slug;
  axios
    .get(`/api/v1/posts/${slug}/`)
    .then((response) => {
      this.post = response.data;
    })
    .catch((error) => {
      console.error('Error al obtener los detalles del post:', error);
    });
},
created() {
  this.fetchPost();
},

```

- **Renderizado condicional:**
    
    Mientras no se haya cargado el post, se muestra un texto de "Cargando...". Cuando está listo, se muestra el contenido completo.
    

```html
<div v-if="post" class="container-wide py-5" id="container">
  <div class="text-center mb-4">
    <h1 class="text-primary">{{ post.title }}</h1>
    <p class="text-muted date-large">{{ formatDate(post.created_at) }}</p>
  </div>
  ...
</div>
<div v-else>
  <p class="text-center">Cargando...</p>
</div>

```

- **Visualización de imágenes:**
    
    Se muestra una imagen principal fija y, si hay más imágenes, se muestran como miniaturas debajo.
    

```html
<div v-if="post.images && post.images.length > 0" class="text-center mb-4">
  <img
    :src="getImageUrl(post.images[0].image)"
    alt="Imagen del post"
    class="img-post-main rounded-3"
    draggable="false"
  />
</div>

<div v-if="post.images && post.images.length > 1" class="d-flex justify-content-center flex-wrap mb-4 gap-3">
  <div v-for="(image, index) in post.images.slice(1)" :key="index" class="col-md-4 mb-3">
    <img :src="getImageUrl(image.image)" alt="Imagen adicional" class="img-fluid rounded-3 img-post" />
  </div>
</div>

```

- **Formato personalizado de fecha:**
    
    Para mostrar la fecha en formato local y con horas.
    

```jsx
formatDate(dateString) {
  const options = {
    day: '2-digit',
    month: 'short',
    hour: '2-digit',
    minute: '2-digit',
    hour12: false,
  };
  const date = new Date(dateString);
  return new Intl.DateTimeFormat('es-ES', options).format(date).replace(',', '');
},

```

- **Botón para volver a la lista de posts:**
    
    Usa un componente `SolidButton` con routing para facilitar la navegación.
    

```html
<SolidButton
  text="Volver a la lista de posts"
  iconClass="fas fa-arrow-left"
  routeTo="/posts"
/>

```

---

## `Product.vue`

Este componente muestra la página de detalle de un producto con imágenes, opciones seleccionables, cantidad y posibilidad de añadirlo al carrito. Incluye una galería modal para ver imágenes ampliadas.

---

### Funcionalidades principales

- **Carga de producto por parámetros de URL:**
    
    En el `mounted()`, se obtiene el producto desde la API REST usando los slugs de categoría y producto de la ruta.
    

```
async getProduct() {
  const store = useMainStore();
  store.setIsLoading(true);
  const category_slug = this.$route.params.category_slug;
  const product_slug = this.$route.params.product_slug;

  try {
    const response = await axios.get(
      `/api/v1/products/${category_slug}/${product_slug}`
    );
    this.product = response.data;
    document.title = this.product.name + ' | TangerineMess';
    this.displayedPrice = parseFloat(this.product.price);

    if (this.product.options && this.product.options.length > 0) {
      this.selectedOption = this.product.options[0];
      this.updatePrice();
    }
  } catch (error) {
    console.error(error);
  } finally {
    store.setIsLoading(false);
  }
},
mounted() {
  this.getProduct();
},

```

- **Visualización de imágenes y galería modal:**
    - Imagen principal con zoom al hacer clic.
    - Miniaturas de otras imágenes que abren la galería.
    - Modal con navegación entre imágenes (anterior, siguiente) y botón de cierre.

```html
<img
  :src="product.get_image"
  alt="Product Image"
  class="img-fluid rounded"
  style="max-height: 500px; object-fit: contain; cursor: zoom-in;"
  @click="openGallery({ full_image_url: product.get_image, alt_text: 'Imagen principal', id: 'main' })"
/>

<div v-if="product.images && product.images.length > 0" class="d-flex flex-wrap gap-2 mt-3 justify-content-center">
  <div v-for="img in product.images" :key="img.id" style="width: 48%;">
    <img
      :src="img.full_image_url"
      :alt="img.alt_text || 'Imagen adicional'"
      class="img-fluid rounded"
      style="cursor: zoom-in; object-fit: cover; max-height: 180px; width: 100%;"
      @click="openGallery(img)"
    />
  </div>
</div>

<div v-if="galleryOpen" class="gallery-modal" @click.self="closeGallery">
  <button class="btn-close btn-close-white gallery-close-btn" @click="closeGallery"></button>
  <img :src="currentGalleryImage.full_image_url" :alt="currentGalleryImage.alt_text || 'Imagen galería'" class="gallery-image" />
  <button class="gallery-nav prev" @click.stop="prevImage">&#10094;</button>
  <button class="gallery-nav next" @click.stop="nextImage">&#10095;</button>
</div>

```

- **Selección de opciones del producto:**
    
    Si el producto tiene opciones (e.g., tamaño, color), se muestran en un select. Al cambiar la opción, se actualiza el precio mostrado.
    

```html
<div v-if="product.options && product.options.length > 0" class="mb-3">
  <label for="product-option-select" class="form-label">Opciones</label>
  <select
    id="product-option-select"
    class="form-select"
    v-model="selectedOption"
    @change="onOptionChange"
  >
    <option v-for="option in product.options" :key="option.id" :value="option">
      {{ option.name }} ({{ parseFloat(option.additional_price).toFixed(2) }}€)
    </option>
  </select>
</div>

```

- **Cantidad con botones + / - y validación:**
    
    El usuario puede modificar la cantidad con los botones o escribiendo directamente. No permite cantidades menores que 1.
    

```html
<div class="mb-3">
  <label class="form-label">Cantidad</label>
  <div class="input-group quantity-input">
    <button class="btn quantity-btn" @click="quantity > 1 && quantity--">
      <i class="fas fa-minus"></i>
    </button>
    <input
      type="number"
      class="form-control text-center quantity-field"
      min="1"
      v-model.number="quantity"
    />
    <button class="btn quantity-btn" @click="quantity++">
      <i class="fas fa-plus"></i>
    </button>
  </div>
</div>

```

- **Añadir al carrito y notificación (toast):**
    
    Se crea un objeto con el producto seleccionado, cantidad y opción elegida, y se añade al store global. Luego se muestra un toast de confirmación que desaparece tras 2 segundos o al cerrarlo manualmente.
    

```jsx
addToCart() {
  if (isNaN(this.quantity) || this.quantity < 1) {
    this.quantity = 1;
  }

  const item = {
    product: {
      ...this.product,
      selectedOption: this.selectedOption,
      price: this.selectedOption
        ? parseFloat(this.selectedOption.additional_price)
        : parseFloat(this.product.price),
      totalWeight: this.selectedOption
        ? parseFloat(this.product.weight || 0) +
          parseFloat(this.selectedOption.additional_weight || 0)
        : parseFloat(this.product.weight || 0),
    },
    quantity: this.quantity,
  };

  const store = useMainStore();
  store.addToCart(item);

  this.showToast = true;

  if (this.toastTimeoutId) clearTimeout(this.toastTimeoutId);
  this.toastTimeoutId = setTimeout(() => {
    this.showToast = false;
  }, 2000);
},

```

- **Navegación de la galería:**

```jsx
openGallery(img) {
  const mainImage = {
    full_image_url: this.product.get_image,
    alt_text: 'Imagen principal',
    id: 'main',
  };
  this.galleryImages = [mainImage, ...this.product.images];
  this.currentGalleryIndex = this.galleryImages.findIndex(
    i => i.id === img.id || i.full_image_url === img.full_image_url
  );
  this.currentGalleryImage = this.galleryImages[this.currentGalleryIndex];
  this.galleryOpen = true;
},

closeGallery() {
  this.galleryOpen = false;
  this.currentGalleryImage = null;
  this.galleryImages = [];
  this.currentGalleryIndex = 0;
},

nextImage() {
  this.currentGalleryIndex = (this.currentGalleryIndex + 1) % this.galleryImages.length;
  this.currentGalleryImage = this.galleryImages[this.currentGalleryIndex];
},

prevImage() {
  this.currentGalleryIndex = (this.currentGalleryIndex - 1 + this.galleryImages.length) % this.galleryImages.length;
  this.currentGalleryImage = this.galleryImages[this.currentGalleryIndex];
},

```

- **Botón para volver a la lista de productos:**
    
    Usa un componente `SolidButton` con routing hacia `/store`.
    

```html
<SolidButton
  text="Volver a la lista de productos"
  iconClass="fas fa-arrow-up"
  routeTo="/store"
/>

```

---

# `mainStore.js`

Este store es el corazón de la gestión del estado global en tu aplicación usando Pinia. Básicamente, aquí guardas y manejas cosas esenciales como:

- El carrito de compras (`cart`),
- El estado de autenticación del usuario (`isAuthenticated`, `token`, `username`),
- El estado de carga (`isLoading`).

### 1. **Estado (`state`)**

El estado inicial guarda:

- `cart`: Un objeto con un array `items` donde vas almacenando los productos añadidos al carrito.
- `isAuthenticated`: Para saber si el usuario está logueado o no.
- `token`: El token de autenticación JWT o similar.
- `username`: El nombre del usuario conectado, para manejar datos específicos, por ejemplo, el carrito por usuario.
- `isLoading`: Para mostrar indicadores de carga en la UI cuando estás esperando respuestas de APIs o procesos.

```jsx
state: () => ({
  cart: { items: [] },
  isAuthenticated: false,
  token: '',
  isLoading: false,
  username: ''
}),

```

### 2. **Getters**

Son como propiedades calculadas que te permiten obtener datos derivados del estado de forma sencilla y reactiva.

- `cartTotalLength`: Suma total de unidades que hay en el carrito.
- `cartTotalPrice`: Precio total acumulado de todos los productos en el carrito.

```jsx
getters: {
  cartTotalLength(state) {
    return state.cart.items.reduce((acc, item) => acc + item.quantity, 0)
  },
  cartTotalPrice(state) {
    return state.cart.items.reduce(
      (acc, item) => acc + item.quantity * item.product.price,
      0
    )
  }
}

```

Así, por ejemplo, cuando quieras mostrar el total de productos o el total a pagar, solo llamas a estas propiedades sin tener que recalcular a mano.

### 3. **Actions**

Aquí están todas las funciones que modifican el estado, que también pueden hacer operaciones asíncronas o lógicas complejas.

- **initializeStore:**
    
    Al iniciar la app, esta función carga el estado guardado en `localStorage`.
    
    Por ejemplo, recupera el token, el usuario y el carrito asociado a ese usuario.
    

```jsx
initializeStore() {
  const token = localStorage.getItem('token')
  if (token) {
    this.token = token
    this.isAuthenticated = true
  } else {
    this.token = ''
    this.isAuthenticated = false
  }

  const storedUsername = localStorage.getItem('username')
  if (storedUsername) {
    this.username = storedUsername
  }

  const cartKey = this.username ? `cart_${this.username}` : 'cart'
  const storedCart = localStorage.getItem(cartKey)
  this.cart = storedCart ? JSON.parse(storedCart) : { items: [] }
},

```

- **addToCart:**
    
    Añade un producto al carrito. Si el producto ya está (considerando la opción seleccionada), solo aumenta la cantidad. Luego actualiza el `localStorage`.
    

```jsx
addToCart(item) {
  const existing = this.cart.items.find(i =>
    i.product.id === item.product.id &&
    JSON.stringify(i.product.selectedOption) === JSON.stringify(item.product.selectedOption)
  )

  if (existing) {
    existing.quantity += item.quantity
  } else {
    this.cart.items.push(item)
  }

  const cartKey = this.username ? `cart_${this.username}` : 'cart'
  localStorage.setItem(cartKey, JSON.stringify(this.cart))
},

```

- **removeFromCart:**
    
    Elimina un producto del carrito (también compara opciones para distinguir productos iguales con diferentes opciones).
    
- **clearCart:**
    
    Vacía el carrito y sincroniza `localStorage`.
    
- **setToken, removeToken, setUsername, setIsLoading:**
    
    Métodos para actualizar estados relacionados con la sesión y la carga.
    
- **updateLocalStorageCart:**
    
    Solo para sincronizar el carrito en `localStorage` cuando se quiera sin modificar el estado.
    

---

### Uso típico

En la aplicación, cuando el usuario añade algo al carrito, se llama a:

```jsx
store.addToCart({ product: productoSeleccionado, quantity: cantidad })

```

Y cuando la aplicación arranca, cargas el estado guardado con:

```jsx
store.initializeStore()

```

---

## **Router:**

Para la navegación dentro de mi aplicación, utilizo **Vue Router 4** configurado con el modo historial (`createWebHistory`) para que las URLs se vean limpias, sin el típico `#`.

He definido todas las rutas necesarias para las distintas vistas, como la página principal (`Home`), la tienda (`Store`), los productos, el blog, el carrito, y muchas otras. Cada ruta está asociada a un componente Vue en formato **Single File Component (SFC)**.

Además, algunas rutas requieren que el usuario esté autenticado para acceder (por ejemplo, “Mi cuenta” y el proceso de “Checkout”). Para esto, utilizo el campo `meta: { requireLogin: true }` dentro de las rutas que deben estar protegidas.

Para controlar el acceso, implemento un **guard global** con `router.beforeEach`, que antes de cada cambio de ruta revisa si la ruta requiere login y si el usuario está autenticado. Esto lo verifico consultando el estado de autenticación desde la store principal (Pinia). Si no está autenticado, redirijo al usuario a la página de login y le paso la ruta original en la query para que, una vez inicie sesión, pueda regresar a donde quería ir.

Esto me asegura que solo usuarios con sesión activa puedan acceder a partes privadas de la app, manteniendo una buena experiencia de usuario y seguridad.

En resumen, con esta configuración de Vue Router logro:

- Navegación fluida y organizada entre las distintas vistas.
- URLs amigables y fáciles de leer.
- Control de acceso basado en autenticación, integrado con la store global.
- Uso de rutas dinámicas para mostrar productos y categorías según la URL.

Parte del Código comentado de router:

```python
import { createRouter, createWebHistory } from 'vue-router'
import { useMainStore } from '../store/mainstore'

// Importación de las vistas (componentes) para cada ruta
import Home from '@/views/Home.vue'
import Product from '../views/Product.vue'
// ... otras importaciones omitidas para brevedad

// Definición de rutas
const routes = [
  {
    path: '/',
    name: 'home',
    component: Home
  },
  {
    path: '/store',
    name: 'Store',
    component: Store
  },
  {
    path: '/:category_slug/:product_slug/',
    name: 'Product',
    component: Product
  },
  // Otras rutas...
  {
    path: '/my-account',
    name: 'MyAccount',
    component: MyAccount,
    meta: { requireLogin: true }  // Ruta protegida, requiere login
  },
  {
    path: '/cart/checkout',
    name: 'Checkout',
    component: Checkout,
    meta: { requireLogin: true }  // Ruta protegida, requiere login
  },
]

// Creación del router con historial HTML5 (sin # en URL)
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

// Guard global antes de cada navegación para proteger rutas con requireLogin
router.beforeEach((to, from, next) => {
  const store = useMainStore()    // Accede a la store Pinia para estado global
  store.initializeStore()         // Inicializa la store (ej. carga token/localStorage)

  if (to.matched.some(record => record.meta.requireLogin) && !store.isAuthenticated) {
    // Si la ruta requiere login y el usuario NO está autenticado, redirige a LogIn
    next({ name: 'LogIn', query: { to: to.path } })
  } else {
    // Si está autenticado o la ruta no requiere login, continúa normalmente
    next()
  }
})

export default router

```

# **Tests**

Para asegurarme de que las partes más importantes de la aplicación funcionen correctamente, he implementado algunos tests unitarios con **Jest** y **@vue/test-utils**. Estos tests me ayudan a validar el comportamiento esperado de los componentes clave como el login, el formulario de contacto y el checkout.

---

### Login

El test del login se encarga de comprobar que el formulario se renderiza correctamente y que, al hacer submit, se llama al método de autenticación del store. También incluye un caso en el que simulo un error de login para asegurarme de que se muestra un mensaje apropiado si las credenciales son incorrectas.

---

### Contacto

Aquí testeo el formulario de contacto que se encuentra en la sección "¡Hablemos!". El objetivo principal es asegurarme de que:

- Se puede enviar el formulario con datos válidos.
- Se muestra un mensaje de éxito si la petición se completa correctamente.
- En caso de error (por ejemplo, si falla la conexión con la API), se notifica adecuadamente al usuario.
- Además, el formulario se limpia después de enviarlo con éxito.

---

### Checkout

En el checkout, me enfoqué en testear el flujo más crítico: que los campos obligatorios se validen correctamente antes de continuar. El test comprueba que:

- Si falta algún campo importante (nombre, email, dirección...), se muestran errores y **no** se continúa con la petición a Stripe.
- Stripe **no** se ejecuta si hay errores de validación.
- (Más adelante podría incluir un test para simular una compra exitosa con redirección al "success").

---