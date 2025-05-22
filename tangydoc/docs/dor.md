
# DOR

### Adaptabilidad

Para este proyecto he elegido una paleta de colores basada en azules y naranjas, representativos de la artista a quien está dedicada la web. Estos colores están definidos como variables CSS en un archivo separado para facilitar su uso consistente en todo el proyecto y permitir un mantenimiento sencillo:

```css
:root {
    --color-light-orange: #ffe2bb;
    --color-primary: #F59300;
    --color-secondary: #FF9100;
    --color-secondary-hover: #cc6300;
    --color-bg: #377AB9;
    --color-dark-blue: #0056b3;
    --color-darker-blue: #034184;
    --color-darkest-blue: #1d1d35;
    --color-warm-cream: #fffbf8;
    --color-warmer-cream:#fdfdeb;
    --color-yellow: #fdeaa5;
    --color-light-blue: #81aedb;
    --color-lighter-blue: #cbe5ff;
    --color-grey: #838897;
    --color-light-grey: #b8bfd3;
  }

```

El uso de colores con buen contraste garantiza que los elementos sean fácilmente localizables y legibles, mejorando la experiencia visual y funcional del usuario. Se han seleccionado tonos que combinan bien para destacar botones, textos y otros componentes clave.

En cuanto a la fuente, he utilizado principalmente dos familias, Degular, y ObviouslyNarrow. La primera para textos descriptivos, y la segunda para titulares, por su carácter alargado. Ambas son letras redondeadas. Además, modifico esta según ayude a la legibilidad, como por ejemplo, añadiendo la propiedad de letter-spacing en números o menús de navegación para hacer a todo el mundo entender bien lo que está leyendo. En general, uso tamaños de fuente grandes para permitir a los visitantes leer sin problema.

```python
@font-face {
    font-family: 'Degular';
    src: url('../fonts/DegularDisplay-Regular.otf') format('opentype');
    font-weight: normal;
    font-style: normal;
  }
    @font-face {
    font-family: 'ObviouslyNarrow';
    src: url('../fonts/ObviouslyNarrow-Regular.otf') format('opentype');
    font-weight: normal;
    font-style: normal;
  }
  @font-face {
    font-family: 'ObviouslyNarrowBold';
    src: url('../fonts/ObviouslyNarrow-Bold.otf') format('opentype');
    font-weight: bold;
    font-style: bold;
  }
  
```

### Accesibilidad y Usabilidad

Se ha prestado especial atención a la accesibilidad para asegurar que la web sea usable para la mayor cantidad de personas posible:

- Se emplean etiquetas `<label>` correctamente asociadas con todos los campos de formulario, facilitando el uso con lectores de pantalla y mejorando la navegación por teclado.
- Todas las imágenes cuentan con atributos `alt` descriptivos, incluyendo las imágenes de productos, para ofrecer contexto a usuarios con discapacidades visuales.
- Los indicadores visuales para la interacción, como cambios de cursor, efectos hover en enlaces y botones, o animaciones sutiles en tarjetas, ofrecen señales claras sobre la interactividad de los elementos.
- La navegación es clara, con botones y enlaces intuitivos, con tooltips informativos en iconos y elementos que requieren explicación adicional.
- Se han implementado elementos como contadores visibles en el carrito y botones de fácil acceso para acciones comunes (volver arriba, navegación atrás), que mejoran la experiencia del usuario.
- La interfaz es simple y consistente, reduciendo la curva de aprendizaje y haciendo que la web sea accesible para usuarios con diferentes niveles de habilidad digital.
- Utilizo toast o mensajes de información al usuario tras haber realizado acciones como añadir al carrito, registrarse o comprar.

### Diseño Responsive

El diseño es completamente responsive, adaptándose fluidamente a distintos tamaños y dispositivos:

- Se utiliza una combinación de grid flexible (`flexbox`, CSS grid) y el sistema de columnas de Bootstrap (`col-12`, `col-md-6`, etc.) para que los componentes y las tarjetas se reorganizen y escalen correctamente en móviles, tablets y escritorios.
- Los menús de navegación se transforman en menús desplegables o hamburguesa en dispositivos móviles para optimizar el espacio y facilitar el acceso.
- Los tamaños de fuentes y botones escalan de forma proporcional según el dispositivo para mantener una legibilidad óptima y facilitar la interacción táctil.
- Las imágenes y demás contenidos gráficos se ajustan para evitar desbordamientos o distorsiones, manteniendo la coherencia visual.
- Se han incorporado media queries para modificar estilos específicos en función del ancho de pantalla, logrando un comportamiento adaptado sin sacrificar la estética en dispositivos grandes.

![image.png](image%202.png)

![image.png](image%203.png)

![image.png](image%204.png)

## **Home**

En este caso, tengo un texto que se desplaza a lo largo de la pantalla, y he agregado un evento para evitar que se sobreponga en pantallas más pequeñas, editando su ancho.

![image.png](image%205.png)

![image.png](image%206.png)

### Navbar

Este se convierte en menú hamburguesa en dispositivos pequeños.

![image.png](image%207.png)

### Galería:

Tiene etiquetas para las categorías que filtran, y estos botones activos están en azúl.

![image.png](image%208.png)

**Pantalla pequeña**

![image.png](image%209.png)

![image.png](image%2010.png)

Se puede navegar entre elementos por flechas, y cerrar al darle click a la x, o cualquier parte fuera de la imagen.

Al hacer hover, se muestra el título de la imagen.

![image.png](06bb9aa6-e906-4166-9b32-995923620b3a.png)

**INFO - Sobre mí**

![image.png](image%2011.png)

![image.png](image%2012.png)

![image.png](image%2013.png)

### INFO - Comisiones

Se adapta a las pantallas.

![image.png](image%2014.png)

![image.png](image%2015.png)

![image.png](image%2016.png)

**Info - Eventos**

![image.png](image%2017.png)

![image.png](image%2018.png)

**Info - Preguntas y Respuestas**

Las preguntas desplegadas cambian de color e icono.

![image.png](image%2019.png)

![image.png](image%2020.png)

**Contacto**

![image.png](image%2021.png)

![image.png](image%2022.png)

![image.png](f3526e3a-e11d-4edd-8419-18fd1515c721.png)

Tiene tooltip para indicar el significado del icono.

**Blog**

![image.png](image%2023.png)

![image.png](image%2024.png)

![image.png](aa7d539d-3b6e-4a97-a9a4-052c547ce0c3.png)

![image.png](image%2025.png)

**Blog/post**

![image.png](image%2026.png)

![image.png](image%2027.png)

**Login Modal**

Fácil de abrir y cerrar, opaca el fondo de la página. Contiene texto en los inputs para especificar qué debe escribir el usuario.

![image.png](image%2028.png)

![image.png](image%2029.png)

Móvil:

![image.png](image%2030.png)

![image.png](image%2031.png)

**Mi cuenta**

![image.png](image%2032.png)

![image.png](image%2033.png)

**Carrito:**

Mensajes explican el estado del carrito si no hay añadidos

![image.png](image%2034.png)

![image.png](image%2035.png)

![image.png](image%2036.png)

### Caja. Desglose de pedido

![image.png](image%2037.png)

![image.png](image%2038.png)

**La tienda**

![image.png](image%2039.png)

Animación en la que se amplia el tamaño de la tarjeta y animación de línea bajo el título.

Permíte hacer búsquedas claras.

![image.png](5ce54250-8e7b-4dd9-a13c-b58b9e2cad93.png)

![image.png](image%2040.png)

![image.png](9b269f43-57de-47a9-9c71-914904a5bcf5.png)

**Vista móvil:**

![image.png](image%2041.png)

![image.png](image%2042.png)
