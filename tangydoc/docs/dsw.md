
# DSW

### **Introducción:**

El backend del proyecto está desarrollado con **Django** y se encarga de gestionar la lógica de negocio y los datos esenciales de la aplicación: productos, usuarios, pedidos, publicaciones de blog, eventos, imágenes y más. Mediante **Django REST Framework**, se crean las APIs que comunican esta información con el frontend. Django facilita el desarrollo gracias a sus herramientas integradas como el sistema de administración, el manejo de usuarios y su conexión con bases de datos.

### Librerías usadas en el backend

El proyecto he experimentado y explorado posibilidades, descubriendo librerías nuevas que me ayudaron a aumentar la rapidez para en la creación de apis.

- **Django** y **DRF** forman la base del backend y la API REST.
- **djoser** y **simplejwt** facilitan la autenticación de usuarios mediante tokens JWT.
- **django-cors-headers** permite la comunicación con el frontend en Vue configurando el acceso entre orígenes distintos.
- **pillow** gestiona la carga y edición de imágenes.
- **stripe** permite integrar pagos seguros en la web.
- **cryptography** y **defusedxml** refuerzan la seguridad del proyecto, protegiendo datos sensibles y evitando vulnerabilidades XML.
- **Kubi** - personalizar el panel de administración
- También se incluyen bibliotecas como **requests**, **oauthlib**, o **social-auth.**

### **APLICACIONES**

Este proyecto cuenta con varias aplicaciones. Main, Post, Event, order, personal, portfolio, product, order. A continuación hablaré resumidamente de cada una de ellas.

Cuenta con la carpeta media, donde se guardan todas las imágenes subidas al proyecto.

#### MAIN

La app `main` centraliza la configuración general del proyecto, como rutas, bases de datos, correo y middlewares. Incluye la configuración de **envío de correos mediante Gmail** para facilitar la comunicación en el formulario de contacto, y define los **endpoints principales bajo `/api/v1/`**, lo que permite escalar futuras versiones (como `v2`). También gestiona la carga de archivos multimedia y estática, y define las rutas que enlazan con otras apps como `post`, `product`, `order`, etc.

```python
from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('djoser.urls')),
    path('api/v1/', include('djoser.urls.authtoken')),
    path('api/v1/', include('post.urls')),
    path('api/v1/', include('product.urls')),
    path('api/v1/', include('order.urls')),
    path('api/v1/', include('event.urls')),
    path('api/v1/', include('portfolio.urls')),
    path('api/v1/', include('personal.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

```

Les he escrito el prefijo v1 para que no haya conflicto en caso de una nueva actualización de la api, a la que llamaría v2, ets…

**Djoser: path('api/v1/', include('djoser.urls')),**

Incluye las URLs estándar de Djoser para manejar autenticación (registro, login, logout, cambio de contraseña, etc).

**path('api/v1/', include('djoser.urls.authtoken')),**

Incluye rutas específicas para el **token de autenticación** cuando usas el sistema de tokens con Django REST Framework.

###  Aplicación **Post**

Esta aplicación gestiona publicaciones o entradas de blog, permitiendo almacenar contenido textual junto con imágenes relacionadas, organizadas en categorías definidas.

---

### Modelo `Post`

Representa una entrada o publicación en el blog. Incluye los siguientes campos:

- `title`: Título del post, máximo 256 caracteres.
- `slug`: Cadena única generada automáticamente a partir del título para URLs amigables.
- `content`: Contenido principal del post (texto libre).
- `created_at`: Fecha y hora de creación, asignada automáticamente.
- `category`: Categoría del post, con opciones predefinidas (`Artist Alley`, `Daily Life`, `Announcement`, etc.).

El método `save()` sobreescrito asegura que el campo `slug` se actualice automáticamente usando la función `slugify` con base en el título, facilitando la generación de URLs limpias.

La clase `Meta` ordena los posts por fecha de creación descendente para mostrar primero los más recientes.

```python
class Post(models.Model):
    class Category(models.TextChoices):
        ARTIST_ALLEY = 'AL', 'Artist Alley'
        DAILY_LIFE = 'DL', 'Daily Life'
        OTHER = 'OT', 'Other'
        ANNOUNCEMENT = 'AN', 'Announcement'

    title = models.CharField(max_length=256)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    category = models.CharField(max_length=2, choices=Category.choices, default=Category.OTHER)

    def save(self, *args, **kwargs):
        self.slug = slugify(self.title)
        super().save(*args, **kwargs)

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return f'/{self.slug}/'

    class Meta:
        ordering = ['-created_at']

```

### Modelo `PostImage`

Permite asociar múltiples imágenes a un post, con los siguientes campos:

- `post`: Relación Many-to-One con el modelo `Post`.
- `image`: Archivo de imagen que se guarda en la carpeta `uploads/blog_posts/`.
- `caption`: Texto descriptivo o pie de foto, opcional.

```python
class PostImage(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='images')
    image = models.ImageField(upload_to='uploads/blog_posts/')
    caption = models.CharField(max_length=256, blank=True)

    def __str__(self):
        return f"Image for: {self.post.title}"

```

---

## Vistas

Se utilizan vistas basadas en clases (APIView) para ofrecer acceso a las publicaciones a través de una API RESTful.

- **`PostList`**: Devuelve una lista de posts, con opción a filtrar por categoría a través de un parámetro GET `category`.
- **`PostDetail`**: Devuelve los detalles completos de un post identificado por su `slug`.

Ambas usan serializers para estructurar la respuesta JSON.

```python
class PostList(APIView):
    def get(self, request, format=None):
        category = request.GET.get('category', None)
        posts = Post.objects.all()
        if category:
            posts = posts.filter(category=category)
        serializer = PostListSerializer(posts, many=True)
        return Response(serializer.data)

class PostDetail(APIView):
    def get_object(self, slug):
        try:
            return Post.objects.get(slug=slug)
        except Post.DoesNotExist:
            raise Http404

    def get(self, request, slug, format=None):
        post = self.get_object(slug)
        serializer = PostDetailSerializer(post)
        return Response(serializer.data)

```

---

## Serializadores

Transforman los modelos en datos JSON para la API.

- `PostImageSerializer`: Serializa imágenes con su ruta y caption.
- `PostListSerializer`: Resume los posts con título, slug, fecha, categoría legible, resumen del contenido y lista de imágenes.
- `PostDetailSerializer`: Muestra todos los detalles del post, incluyendo contenido completo e imágenes.

```python
class PostImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = PostImage
        fields = ['image', 'caption']

class PostListSerializer(serializers.ModelSerializer):
    summary = serializers.SerializerMethodField()
    category_display = serializers.CharField(source='get_category_display', read_only=True)
    images = PostImageSerializer(many=True, read_only=True)

    class Meta:
        model = Post
        fields = ['title', 'slug', 'created_at', 'category', 'category_display', 'summary', 'images']

    def get_summary(self, obj):
        return ' '.join(obj.content.split()[:25]) + '...'

class PostDetailSerializer(serializers.ModelSerializer):
    images = PostImageSerializer(many=True, read_only=True)
    category_display = serializers.CharField(source='get_category_display', read_only=True)

    class Meta:
        model = Post
        fields = ['title', 'slug', 'content', 'created_at', 'category', 'category_display', 'images']

```

---

## Tests

Se valida el comportamiento de la API para obtener la lista de posts y el detalle de un post específico, incluyendo la gestión de errores para slugs no existentes.

- Se crea un post de prueba en `setUp`.
- Se verifica que la lista de posts incluya dicho post.
- Se consulta el detalle por slug y se confirma la respuesta.
- Se prueba que un slug inexistente devuelve un error 404.

```python
class PostAPITest(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.post = Post.objects.create(
            title="Test Post",
            content="Contenido de prueba para el post.",
            category=Post.Category.ANNOUNCEMENT
        )

    def test_post_list(self):
        response = self.client.get('/api/v1/posts/')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(any(p['title'] == "Test Post" for p in response.data))

    def test_post_detail(self):
        response = self.client.get(f'/api/v1/posts/{self.post.slug}/')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['title'], "Test Post")

    def test_post_not_found(self):
        response = self.client.get('/api/v1/posts/slug-inexistente/')
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)

```

---

## URLs

Se definen rutas RESTful para la API de posts:

```python
urlpatterns = [
    path('posts/', PostList.as_view(), name='post-list'),
    path('posts/<slug:slug>/', PostDetail.as_view(), name='post-detail'),
]

```

---

## **Aplicación Event**

Esta aplicación contiene el modelo UpcomingEvent representa eventos próximos con detalles básicos para su presentación.

- **Campos principales:**
    - `title` (CharField): Título del evento.
    - `image` (ImageField): Imagen asociada al evento, guardada en la carpeta `uploads/events/`.
    - `start_date` (DateField): Fecha de inicio del evento.
    - `end_date` (DateField, opcional): Fecha de finalización, permite eventos de varios días.
    - `location` (CharField): Lugar donde se realizará el evento.
    - `link` (URLField, opcional): Enlace externo relacionado con el evento (por ejemplo, página oficial).
    - `location_link` (URLField, opcional): Enlace para la ubicación (como un mapa o dirección web).
- **Comportamiento:**
    - Ordena los eventos automáticamente por fecha de inicio (`start_date`).
    - Método `is_multi_day()` que devuelve `True` si el evento dura más de un día.
- **Representación:**
    - El método `__str__` retorna el título para facilitar su identificación en interfaces administrativas.
    

Tiene una vista basada en APIView que crea un endpoint para obtener la lista completa de eventos próximos. Al recibir una petición GET, consulta todos los registros del modelo `UpcomingEvent`, los serializa con `UpcomingEventSerializer` y devuelve los datos en formato JSON.

La url para esta es: 

```python
path('events/', UpcomingEventList.as_view(), name='upcoming-events'),
```

También tiene un test que crea un evento de prueba, hace una petición GET al endpoint `/events/, v`erifica que la respuesta es 200 OK y comprueba que el evento creado aparece en la respuesta.

### Aplicación Portfolio

Representa una entrada del portafolio.

**Modelo** 

- `title` (CharField): Título de la entrada del portafolio, con un máximo de 255 caracteres.
- `image` (ImageField): Imagen asociada a la entrada, almacenada en la carpeta `uploads/portfolio/`.
- `category` (CharField): Categoría de la entrada, con opciones predefinidas: Concept Art, Animación, Ilustración, y Otros (por defecto).
- `created_at` (DateTimeField): Fecha y hora en que se creó la entrada, se asigna automáticamente al crear el registro.

Las entradas se ordenan por fecha de creación descendente (`ordering = ['-created_at']`), mostrando primero las más recientes.

Como en el caso anterior, es una aplicación simple con una vista basada en APIView que obtiene una lista de registros que son serializados. Y una URL para acceder a los datos.

```python
urlpatterns = [
    path('portfolio/', PortfolioEntryList.as_view(), name='portfolio-list'),
]
```

### Aplicación **Personal**

Contiene modelos que gestionan la información personal, comisiones, preguntas frecuentes y contactos de la web.

### Modelo **About**

- `title` (CharField, opcional): Título descriptivo, puede quedar vacío.
- `content` (TextField): Texto con información sobre la persona o proyecto.
- `image` (ImageField, opcional): Imagen relacionada, almacenada en `media/about/`.

Representa la sección “Acerca de mí” o información general.

---

### Modelo **Commissions**

- `title` (CharField, opcional): Título del servicio o comisión ofrecida.
- `description` (TextField): Descripción detallada de la comisión.
- `price` (DecimalField): Precio de la comisión, con hasta 10 dígitos y 2 decimales.
- `slots_left` (PositiveIntegerField): Número de plazas disponibles para la comisión.
- `image` (ImageField, opcional): Imagen ilustrativa guardada en `media/comissions/`.

Gestiona las ofertas y disponibilidad de comisiones personalizadas.

---

### Modelo **FAQ**

- `question` (CharField): Pregunta frecuente.
- `answer` (TextField): Respuesta correspondiente a la pregunta.

Modelo para mostrar las preguntas frecuentes y sus respuestas.

---

### Modelo **Contact**

- `name` (CharField): Nombre del usuario que contacta.
- `email` (EmailField): Correo electrónico del usuario.
- `message` (TextField): Mensaje enviado por el usuario.
- `created_at` (DateTimeField): Fecha y hora en que se creó el contacto (registro automático).

Permite registrar los mensajes y consultas recibidas desde la sección de contacto.

### Vistas (APIView)

- **ContactView (POST)**
    
    Recibe nombre, email y mensaje, valida campos, guarda contacto y envía email notificando el mensaje.
    
- **AboutView (GET)**
    
    Devuelve la primera instancia de About serializada.
    
- **CommissionsList (GET)**
    
    Devuelve la lista completa de comisiones serializadas.
    
- **FAQList (GET)**
    
    Devuelve todas las preguntas frecuentes serializadas.
    

### Serializadores

Se basan en `ModelSerializer` y exponen todos los campos de sus modelos:

`AboutSerializer, CommissionsSerializer, FAQSerializer`

### Formulario

**ContactForm, un sencillo formulario para recibir mensajes desde el front.**

Campos: `name`, `email`, `message` con validación básica, usado para validar datos de contacto.

### Urls

```python
urlpatterns = [
    path('contact/', ContactView.as_view(), name='contact'),
    path('about/', AboutView.as_view(), name='about'),
    path('commissions/', CommissionsList.as_view(), name='commissions'),
    path('faq/', FAQList.as_view(), name='faq'),
]

```

### **Tests**

He añadido unos test de comprobación sencillos en esta app:

- **test_get_about**: Verifica que la vista de "About" responde con éxito (200 OK) y que el contenido incluye el título esperado.
- **test_get_commissions**: Comprueba que la lista de comisiones se obtiene correctamente y que devuelve al menos un elemento.
- **test_get_faq**: Asegura que la lista de preguntas frecuentes (FAQ) se recupera correctamente y no está vacía.
- **test_post_contact_success**: Prueba que al enviar un mensaje de contacto válido, la API responde con éxito, guarda el mensaje en la base de datos y confirma el envío.

```python
def test_post_contact_success(self):
        url = reverse('contact')
        data = {
            'name': 'Sergio',
            'email': 'matraca@example.com',
            'message': 'Hola matraca, este es un mensaje de prueba.'
        }
        response = self.client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['message'], '¡Se ha enviado el mensaje con éxito!')
        self.assertTrue(Contact.objects.filter(email='matraca@example.com').exists())
```

- **test_post_contact_missing_fields**: Verifica que si faltan datos obligatorios en el formulario de contacto, la API responde con un error 400 y muestra un mensaje de error.

## Aplicación Product

### Modelo Category

Este modelo representa las categorías de productos en la tienda. Cada categoría tiene un nombre (`name`) y un `slug` único que se utiliza para construir URLs amigables. Las categorías se ordenan alfabéticamente por nombre para facilitar su visualización.

- **Campos:**
    - `name`: Nombre descriptivo de la categoría.
    - `slug`: Identificador único para URLs.
- **Métodos importantes:**
    - `get_absolute_url()`: Devuelve la URL relativa basada en el slug, facilitando la navegación hacia la categoría.
- **Meta:**
    - Ordena las categorías por nombre (`ordering = ('name',)`).

---

### Modelo Product

Este modelo define los productos que se ofrecen en la tienda, cada uno vinculado a una categoría específica mediante una relación de clave foránea (`ForeignKey`). Incluye detalles básicos como nombre, descripción, precio y fechas de creación. También permite gestionar imágenes principales y miniaturas, generando automáticamente una miniatura si no existe para optimizar la carga.

- **Campos principales:**
    - `category`: Categoría a la que pertenece el producto.
    - `name`: Nombre del producto.
    - `slug`: Identificador único para URLs.
    - `description`: Descripción detallada (opcional).
    - `price`: Precio del producto con dos decimales.
    - `image`: Imagen principal del producto.
    - `thumbnail`: Imagen en miniatura, optimizada para cargas rápidas.
    - `date_added`: Fecha en que se añadió el producto.
    - `weight`: Peso base del producto en gramos (importante para envíos y logística).
- **Métodos relevantes:**
    - `get_absolute_url()`: Construye la URL del producto usando el slug de la categoría y del producto.
    - `get_image()`: Retorna la URL completa de la imagen principal.
    - `get_thumbnail()`: Retorna la URL de la miniatura, generándola automáticamente si no existe.
    - `make_thumbnail(image, size)`: Crea una miniatura redimensionando y optimizando la imagen principal.
- **Meta:**
    - Ordena los productos por fecha de adición, mostrando primero los más recientes (`ordering = ('-date_added',)`).

```python
class Product(models.Model):
    category = models.ForeignKey(Category, related_name='products', on_delete=models.CASCADE)
    name = models.CharField(max_length=255)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True, null=True)
    price = models.DecimalField(max_digits=6, decimal_places=2)
    image = models.ImageField(upload_to='uploads/', blank=True, null=True)
    thumbnail = models.ImageField(upload_to='uploads/', blank=True, null=True)
    date_added = models.DateTimeField(auto_now_add=True)
    weight = models.PositiveIntegerField(default=0, help_text="Peso base en gramos")

    class Meta:
        ordering = ('-date_added',)

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        return f'/{self.category.slug}/{self.slug}/'

    def get_image(self):
        if self.image:
            return 'http://127.0.0.1:8000' + self.image.url
        return ''

    def get_thumbnail(self):
        if self.thumbnail:
            return 'http://127.0.0.1:8000' + self.thumbnail.url
        else:
            if self.image:
                self.thumbnail = self.make_thumbnail(self.image)
                self.save()
                return 'http://127.0.0.1:8000' + self.thumbnail.url
            else:
                return ''

    def make_thumbnail(self, image, size=(600, 400)):

        img = Image.open(image)
        img = img.convert('RGB')
        img.thumbnail(size)

        thumb_io = BytesIO()
        img.save(thumb_io, 'JPEG', quality=85)

        thumbnail = File(thumb_io, name= image.name)
        return thumbnail
    
```

---

### Modelo ProductImage

Este modelo permite añadir múltiples imágenes adicionales para un producto, mejorando la presentación visual en la tienda.

- **Campos:**
    - `product`: Producto asociado (FK).
    - `image`: Imagen adicional.
    - `alt_text`: Texto alternativo para accesibilidad y SEO (opcional).

---

### Modelo ProductOption

Define opciones o variantes de un producto que pueden afectar su precio y peso, como diferentes tamaños, colores o configuraciones personalizadas.

- **Campos:**
    - `product`: Producto asociado (FK).
    - `name`: Nombre de la opción o variante.
    - `additional_price`: Precio adicional que suma esta opción al producto base.
    - `additional_weight`: Incremento o decremento de peso en gramos que añade esta opción.

Este conjunto de modelos está diseñado para gestionar la tienda en línea con productos categorizados, número de imágenes flexible y variantes personalizables, proporcionando una base robusta para manejar catálogo, presentación y cálculo de precios/pesos para envíos. Además, incluye funciones para optimizar la carga de imágenes con miniaturas generadas automáticamente.

### Vistas del módulo de productos

---

### `LatestProductsList` (APIView)

- **Descripción:**
    
    Proporciona una lista completa de todos los productos disponibles en la tienda.
    
- **Métodos:**
    - `get`: Recupera todos los objetos `Product`, los serializa y devuelve los datos en formato JSON.
- **Uso:**
    
    Endpoint para mostrar todos los productos recientes o disponibles.
    

---

### `ProductDetail` (APIView)

- **Descripción:**
    
    Muestra el detalle de un producto específico, identificándolo por el slug de la categoría y el slug del producto.
    
- **Métodos:**
    - `get_object(category_slug, product_slug)`: Busca el producto que corresponde a la categoría y slug indicados. Si no existe, lanza un error 404.
    - `get`: Usa `get_object` para obtener el producto y devuelve su información serializada.
- **Uso:**
    
    Endpoint para obtener la información completa de un producto en particular.
    

---

### `CategoryDetail` (APIView)

- **Descripción:**
    
    Proporciona la información detallada de una categoría identificada por su slug.
    
- **Métodos:**
    - `get_object(category_slug)`: Busca la categoría por slug o lanza error 404 si no existe.
    - `get`: Devuelve los datos serializados de la categoría encontrada.
- **Uso:**
    
    Endpoint para ver detalles de una categoría específica.
    

---

### `search` (Función con decorador `@api_view(['POST'])`)

- **Descripción:**
    
    Permite buscar productos mediante una consulta de texto, que compara el término con los campos `name` y `description`.
    
- **Parámetros:**
    - Recibe un JSON con el campo `query` que contiene la palabra o frase a buscar.
- **Funcionamiento:**
    - Filtra los productos que contengan el texto buscado en su nombre o descripción (case-insensitive).
    - Devuelve la lista de productos que coinciden, o una lista vacía si no hay término de búsqueda.
- **Uso:**
    
    Endpoint para implementar funcionalidades de búsqueda dinámica en la tienda.
    

```python
@api_view(['POST'])
def  search(request):
    query = request.data.get('query', '')
    if query:
        products = Product.objects.filter(Q(name__icontains=query) | Q(description__icontains=query))
        serializer = ProductSerializer(products, many=True)
        return Response(serializer.data)
    else:
        return Response({'products': []})
```

### Serializadores del módulo de productos

---

### `ProductImageSerializer`

- **Descripción:**
    
    Serializa las imágenes asociadas a un producto.
    
- **Campos:**
    - `image`: archivo de imagen.
    - `alt_text`: texto alternativo para accesibilidad.
    - `full_image_url`: URL completa absoluta generada dinámicamente para acceder a la imagen.
- **Métodos especiales:**
    - `get_full_image_url`: Construye la URL completa de la imagen usando el contexto de la petición para devolver un enlace absoluto.

```python
class ProductImageSerializer(serializers.ModelSerializer):
    full_image_url = serializers.SerializerMethodField()

    class Meta:
        model = ProductImage
        fields = ['image', 'alt_text', 'full_image_url']

    def get_full_image_url(self, obj):
        request = self.context.get('request')
        if request:
            return request.build_absolute_uri(obj.image.url)
        return obj.image.url

```

---

### `ProductOptionSerializer`

- **Descripción:**
    
    Serializa las opciones adicionales de un producto, como variantes con precio y peso extra.
    
- **Campos:**
    - `name`: nombre de la opción.
    - `additional_price`: incremento en el precio para esta opción.
    - `additional_weight`: incremento en el peso para esta opción.

---

### `ProductSerializer`

- **Descripción:**
    
    Serializa un producto, incluyendo sus datos básicos, imágenes y opciones relacionadas.
    
- **Campos:**
    - `id`, `name`, `price`, `weight`: información básica del producto.
    - `get_absolute_url`: URL relativa al detalle del producto.
    - `description`: descripción del producto.
    - `get_image`, `get_thumbnail`: métodos que devuelven las URLs de imagen principal y miniatura.
    - `images`: lista de imágenes adicionales, serializadas con `ProductImageSerializer`.
    - `options`: lista de opciones del producto, serializadas con `ProductOptionSerializer`.

---

### `CategorySerializer`

- **Descripción:**
    
    Serializa una categoría, incluyendo la lista de productos que pertenecen a ella.
    
- **Campos:**
    - `id`, `name`: datos básicos de la categoría.
    - `get_absolute_url`: URL relativa para acceder a la categoría.
    - `products`: lista de productos relacionados, serializados con `ProductSerializer`.

### URLs de la aplicación Product

```python
from django.urls import path
from product import views

urlpatterns = [
    path('latest-products/', views.LatestProductsList.as_view(), name='latest-products'),
    path('products/search/', views.search, name='product-search'),
    path('products/<slug:category_slug>/<slug:product_slug>/', views.ProductDetail.as_view(), name='product-detail'),
    path('products/<slug:category_slug>/', views.CategoryDetail.as_view(), name='category-detail'),
]

```

---

## Tests

Utilizo `APITestCase` de Django REST Framework para validar las principales funcionalidades de la API de productos, asegurando que las vistas respondan correctamente y que los datos devueltos sean los esperados.

```python
from rest_framework.test import APITestCase
from rest_framework import status
from django.urls import reverse
from .models import Category, Product

class ProductAPITest(APITestCase):
    def setUp(self):
        # Crear una categoría y un producto para usar en los tests
        self.category = Category.objects.create(name="Test Category", slug="test-category")
        self.product = Product.objects.create(
            category=self.category,
            name="Test Product",
            slug="test-product",
            price=9.99,
            weight=100
        )

```

### test_latest_products_list

Este test verifica que la vista de productos más recientes (`latest-products`) responda con éxito (código 200) y que el producto creado en `setUp` aparezca en la lista.

```python
def test_latest_products_list(self):
    url = reverse('latest-products')  
    # Construcción dinámica de la URL usando el nombre de la ruta
    response = self.client.get('/api/v1/latest-products/')  
    # Se puede usar directamente la URL
    self.assertEqual(response.status_code, status.HTTP_200_OK) 
     # Comprobamos respuesta exitosa
    # Confirmamos que "Test Product" esté en los datos recibidos
    self.assertTrue(any(p['name'] == "Test Product" for p in response.data))

```

---

### test_product_detail

Comprueba que la vista de detalle de producto funcione correctamente, devolviendo el producto esperado cuando se consulta por categoría y slug.

```python
def test_product_detail(self):
    url = f'/api/v1/products/{self.category.slug}/{self.product.slug}/'
    response = self.client.get(url)
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertEqual(response.data['name'], "Test Product")  
    # Validamos que el producto sea el correcto

```

---

### test_category_detail

Valida que la vista de detalle de categoría responde correctamente e incluye la lista de productos asociados.

```python
def test_category_detail(self):
    url = f'/api/v1/products/{self.category.slug}/'
    response = self.client.get(url)
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertEqual(response.data['name'], "Test Category")  
    # Confirmamos la categoría
    self.assertTrue('products' in response.data) 
     # Verificamos que se devuelvan los productos relacionados

```

---

### test_search_products

Test para la funcionalidad de búsqueda. Envía una consulta y verifica que el producto que contiene el término se incluya en la respuesta.

```python
def test_search_products(self):
    url = '/api/v1/products/search/'
    response = self.client.post(url, {'query': 'Test'}, format='json')
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertTrue(any(p['name'] == "Test Product" for p in response.data))

```

---

### test_search_no_query

Asegura que si no se envía ningún término de búsqueda, la respuesta sea exitosa y se retorne una lista vacía.

```python
def test_search_no_query(self):
    url = '/api/v1/products/search/'
    response = self.client.post(url, {}, format='json')
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertEqual(response.data['products'], [])

```

## Aplicación Order

## Modelo

Representa una orden de compra realizada por un usuario en la tienda. Contiene información del cliente, detalles del pago y estado del envío.

### Campos principales:

- `user`: Relación con el modelo `User` de Django. Indica el cliente que realizó el pedido.
- `first_name`, `last_name`, `email`, `address`, `zipcode`, `place`, `phone`: Datos personales y de contacto del comprador para la entrega.
- `created_at`: Fecha y hora en que se creó el pedido. Se asigna automáticamente.
- `paid_amount`: Monto total pagado por el pedido. Puede estar vacío si el pago no se ha procesado aún.
- `stripe_token`: Token asociado a Stripe para el pago, usado para validación y seguimiento del pago.
- `shipping_cost`: Costo del envío, que puede variar según el destino o método.
- `status`: Estado actual del pedido. Puede ser:
    - `"processing"` (En proceso)
    - `"shipped"` (Enviado)
    - `"delivered"` (Entregado)
- `tracking_number`: Número de seguimiento para el envío, opcional y se asigna cuando está disponible.

### Comportamiento:

- Los pedidos se ordenan por fecha de creación, mostrando primero las más recientes (`ordering = ['-created_at']`).
- La representación en texto (`__str__`) devuelve el nombre del comprador, para identificación rápida en el admin o logs.

---

![image.png](image.png)

## Modelo `OrderItem`

Representa un ítem individual dentro de un pedido, es decir, un producto comprado.

### Campos principales:

- `order`: Relación con `Order`. Un ítem pertenece a un pedido específico.
- `product`: Relación con `Product`. El producto que fue comprado.
- `price`: Precio unitario del producto en el momento de la compra. Esto garantiza que cambios futuros en precios no afecten el histórico de órdenes.
- `quantity`: Cantidad comprada de ese producto.
- `selected_option`: Opcional, describe alguna variante o opción elegida para el producto (por ejemplo, talla, color).

### Comportamiento:

- La representación en texto devuelve el `id` del ítem, para referencias sencillas.

---

Esta aplicación permite gestionar pedidos complejos, donde cada orden puede contener múltiples productos con opciones específicas. Se almacena información del cliente y del pago para facilitar la gestión y seguimiento, así como el estado de envío.

## Vistas

### `checkout` (Función vista)

**Descripción:**

Permite a un usuario autenticado realizar el pago y crear una nueva orden.

**Método:**

`POST`

**Autenticación y permisos:**

- Solo usuarios autenticados mediante token pueden acceder.

**Flujo principal:**

1. Recibe los datos de la orden desde el cliente.
2. Valida los datos con `OrderSerializer`.
3. Calcula el total a cobrar sumando el costo de los productos y el envío.
4. Realiza el cobro mediante la API de Stripe usando el token recibido.
5. Si el pago es exitoso, guarda la orden asociándola al usuario autenticado.
6. Devuelve la orden creada con estado HTTP 201.

**Errores:**

- Si falla la validación o el cobro con Stripe, devuelve error 400 con detalles.

```python

@api_view(['POST'])
@authentication_classes([authentication.TokenAuthentication])
@permission_classes([permissions.IsAuthenticated])
def checkout(request):
    serializer = OrderSerializer(data=request.data)

    if serializer.is_valid():
        stripe.api_key = settings.STRIPE_SECRET_KEY
        
        items = serializer.validated_data['items']
        shipping_cost = serializer.validated_data.get('shipping_cost', 0)

        paid_amount = sum(item.get('quantity') * item.get('product').price for item in items)
        total_to_charge = paid_amount + shipping_cost

        try:
            charge = stripe.Charge.create(
                amount=int(total_to_charge * 100),
                currency='EUR',
                description='Charge from Charge from TangerineMessStore',
                source=serializer.validated_data['stripe_token']
            )

            serializer.save(user=request.user, paid_amount=total_to_charge)

            return Response(serializer.data, status=status.HTTP_201_CREATED)
        except Exception:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

---

### `OrdersList` (Clase basada en vista - APIView)

**Descripción:**

Lista todas las órdenes realizadas por el usuario autenticado.

**Método:**

`GET`

**Autenticación y permisos:**

- Solo usuarios autenticados mediante token pueden acceder.

**Flujo principal:**

1. Filtra las órdenes por el usuario autenticado.
2. Serializa la lista con `MyOrderSerializer`.
3. Devuelve la lista de órdenes en formato JSON.

```python
class OrdersList(APIView):
    authentication_classes = [authentication.TokenAuthentication]
    permission_classes = [permissions.IsAuthenticated]

    def get(self, request, format=None):
        orders = Order.objects.filter(user=request.user)
        serializer = MyOrderSerializer(orders, many=True)
        return Response(serializer.data)
```

- Se usa Stripe para procesar pagos, configurando la clave secreta desde la configuración del proyecto.
- La lógica de cobro es manejada dentro de la vista `checkout`, asegurando que solo se cree la orden tras confirmarse el pago.
- El uso de autenticación por token garantiza seguridad y control de acceso a las órdenes y pagos.

![image.png](image%201.png)

## Tests

Estos tests comprueban el correcto funcionamiento de los endpoints relacionados con la creación de pedidos (checkout) y la consulta de pedidos existentes para un usuario autenticado.

```python
from rest_framework.test import APITestCase
from rest_framework import status
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token
from django.urls import reverse
from .models import Order, OrderItem
from product.models import Product, Category

```

### Setup inicial

Se crea un usuario de prueba con token de autenticación, así como una categoría y producto para simular la compra.

```python
def setUp(self):
    self.user = User.objects.create_user(username='testuser', password='testpass')
    self.token = Token.objects.create(user=self.user)
    self.client.credentials(HTTP_AUTHORIZATION='Token ' + self.token.key)

    self.category = Category.objects.create(name="Test Category", slug="test-category")
    self.product = Product.objects.create(
        category=self.category,
        name="Test Product",
        slug="test-product",
        price=10.00,
        weight=100
    )

```

---

### Test: Éxito en checkout

Este test envía una solicitud POST válida a la ruta de checkout para crear un pedido. Se valida que:

- La respuesta sea HTTP 201 (creado).
- Se haya creado un pedido y un ítem relacionado.
- El monto pagado sea el correcto (precio productos + envío).

```python
def test_checkout_success(self):
    url = reverse('checkout')
    data = {
        "first_name": "Sergio",
        "last_name": "Delgado",
        "email": "matraca@example.com",
        "address": "123 Matracazo",
        "zipcode": "12345",
        "place": "City",
        "phone": "1234567890",
        "stripe_token": "tok_visa",
        "shipping_cost": "5.00",
        "items": [
            {
                "product": self.product.id,
                "quantity": 2,
                "price": "10.00"
            }
        ]
    }
    response = self.client.post(url, data, format='json')
    self.assertEqual(response.status_code, status.HTTP_201_CREATED)
    self.assertEqual(Order.objects.count(), 1)
    self.assertEqual(OrderItem.objects.count(), 1)
    self.assertEqual(Order.objects.first().paid_amount, 25.00)  # (2*10) + 5 envío

```

---

### Test: Checkout con campos faltantes

Verifica que la API rechace solicitudes incompletas o inválidas, devolviendo un error HTTP 400.

```python
def test_checkout_missing_fields(self):
    url = reverse('checkout')
    data = {
        "first_name": "Sergio",
        "stripe_token": "tok_visa",
        "items": []
    }
    response = self.client.post(url, data, format='json')
    self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

```

---

### Test: Listado de pedidos del usuario

Simula la consulta GET para obtener los pedidos creados por el usuario autenticado. Verifica que la respuesta sea correcta y que se devuelva la cantidad adecuada de pedidos.

```python
def test_orders_list(self):
    order = Order.objects.create(
        user=self.user,
        first_name="Sergio",
        last_name="Doe",
        email="matraca@example.com",
        address="123 Matracazo",
        zipcode="12345",
        place="City",
        phone="1234567890",
        stripe_token="tok_visa",
        paid_amount=15.00
    )
    response = self.client.get(reverse('orders-list'))
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertEqual(len(response.data), 1)

```

## Urls

```python
from django.urls import path
from . import views

urlpatterns = [
    path('checkout/', views.checkout, name='checkout'),
    path('orders/', views.OrdersList.as_view(), name='orders-list'),
]
```