---
theme: gaia
_class: lead
paginate: false
backgroundColor: #374348
color: #fff
marp: true
---

![bg w:480](./assets/VueLogo.svg)

# Tweaks útiles en VueJS

## Network Status Based UI / UX, Image Optimization, Rendering Performance y Animations

---

<!-- _class: lead -->

# Introducción

---

<!-- _class: lead -->

-   Vue destaca por: Simplicidad y pocas restricciones + gran comunidad. (grande en cuánto a calidad que no a tamaño)
-   Buena base con todo lo necesario (para montar un proyecto bastante completo) y muy ampliable con libertad.
-   Cantidad de tweaks tanto de performance cómo visuales o de ahorro de recursos.

---

<!-- _class: lead -->

# Algunos tweaks útiles (hay muchos más)

-   1.  Network Status Based UI
-   2.  Image Optimization with Nuxt Image
-   3.  Rendering performance
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
-   4. One Line Animations

---

<!-- _class: lead -->

Índice

-   **_1. Network Status Based UI_**
-   2.  Image Optimization with Nuxt Image
-   3.  Rendering performance
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
-   4. One Line Animations

---

<!-- _class: lead -->

# 1. Network Status Based UI / UX

UI/UX basada en el estado de la red

---

**Network Status Based UI / UX**

-   Ideales en situaciones en las que tenemos una conexión a internet lenta / inestable (mercados emergentes).
-   Útil para informar a los usuarios del estado de la conexión
-   Muy utilizado en sitios de streaming (regulación del bitrate), o de edición de documentos onLine.
-   Nota: Las funciones (useNetwork, useOnline) de **VueUse** ya implementan esto.
-   Veremos cómo hacerlo VanillaJS cómo excepción

---

![bg w:1200](./assets/1.png)

---

![bg w:400](./assets/2.png)

---

**Ejemplo: Mostrar info conexión**

-   Nos apoyamos en window.navigator:

```javascript
const navigator = window.navigator; // Info ab. user-agent

const isOnline = ref(navigator.onLine);

// Listen for events of the navigator interface to detect NW changes
window.addEventListener("online", () => {
    isOnline.value = true;
});
window.addEventListener("offline", () => {
    isOnline.value = false;
});
```

-   Utilizamos isOnline en nuestra UI para adaptarla

---

**Ejemplo: Enviar versión lightweight de recursos de nuestra app**

-   Nos apoyaremos en "Network Information API" (experimental), aporta información adicional sobre la red del usuario.

-   Ejemplo de propiedades de la instancia:

    -   NetworkInformation.downlink
    -   NetworkInformation.rtt
    -   NetworkInformation.saveData
    -   NetworkInformation.effectiveType
    -   etc

-   Referencia: [MDN: NetworkInformation](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation)

---

-   NetworkInformation.downlink -> Estimación del ancho de banda en mbps

![bg w:1200](./assets/4.png)

---

-   NetworkInformation.rtt -> Round Trip Time (tiempo en ms que un paquete de datos tarda en volver a su emisor habiendo pasado por su destino)

![bg w:1200](./assets/5.png)

---

-   NetworkInformation.saveData -> Si el cliente tiene activada esta opción

![bg w:1200](./assets/6.png)

-   La opción saveData se activa a veces autom. al hacer wifi tether.

---

-   NetworkInformation.effectiveType -> Tipo de red del cliente (estimado según otros valores)

![bg w:1200](./assets/7.png)

---

-   Basándose en estos datos, se lleva a cabo el "Adaptive Loading", en el que no sólo se tiene en cuenta el tamaño de pantalla para decidir qué servir, sino que también se tiene en cuenta la red y Hardware del dispositivo del usuario.

-   Se tienen en cuenta también preferencias del usuario, cómo por ejemplo el modo ahorro de datos, para servir assets más o menos grandes.

---

![bg w:1200](./assets/8.png)

---

-   Accediendo a esta información:

```javascript
const navigator = window.navigator; // Info ab. user-agent
const connection = (navigator as any). connection

// Set default to 4g
const effectiveType = ref('4g')

updateNetworkInfo() {
    effectiveType.value = connection.effectiveType ?? 'unknown'
}

if (connection) { //the API is not available on every browser
    // Add eventListener for when connection changes
    connection.addEventListener('change', updateNetworkInfo)
    updateNetworkInfo() //Make a call to update the netw. info on the
}
```

---

-   Utilizando la información para mostrar una imagen a completa resolución o una versión blurry de un thumbnail.

```html
<img
    :src="
        effectiveType === '4g' 
        ? '/fullsize.webp'
        : '/thumbnail.webp' 
    "
    :class="
        'blur-md': effectiveType !== '4g'
    "
/>
```

-   Tener en cuenta que Safari o FFox aún no lo soportan

---

<!-- _class: lead -->

Índice

-   1. Network Status Based UI
-   **_2. Image Optimization with Nuxt Image_**
-   3.  Rendering performance
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
-   4. One Line Animations

---

<!-- _class: lead -->

# 2. Image Optimization with [Nuxt Image](https://v1.image.nuxtjs.org/)

---

**Image Optimization with Nuxt Image**

-   El performance de nuestra web impacta todo (ux, ratios conversión, usabilidad, etc).

-   Lo que más impacta al performance (+ tiempo) es la candidad de datos que el usuario ha de descargar. (junto con el renderizado, que abordaremos más adelante y añade consumo cpu y ram)

-   Hay varias maneras de reducir esta cantidad de datos, una de las más comunes siendo la optimización de imágenes.

-   Ejemplo: Twitter tiene acceso a la versión HD de nuestra imagen de perfil, pero en la card del tweet, no tiene sentido. Trae una versión reducida.

---

![bg w:1200](./assets/9.png)

---

-   El componente de Nuxt Image, nos permite interactuar con distintos servicios de compresión de imagen, especificando desde el tag en el template <nuxt-img>, el tamaño y formato que consideremos ideal para la imagen.

```html
<template>
    <nuxt-img src="/image.png" width="48" height="48" format="webp" />
</template>
```

-   La documentación es muy completa y destaca del componente, la cantidad de proveedores de compresión de imagen disponibles

---

![bg w:200](./assets/10.png)

---

-   La configuración del proveedor es muy sencilla.

![bg w:900](./assets/11.png)

---

-   Nuxt Image nos expone dos componentes

| Nuxt Image       |   Nativo    |
| ---------------- | :---------: |
| <nuxt-img />     |   <img />   |
| <nuxt-picture /> | <picture /> |

-   Podemos además, especificar para algunos proveedores distintos tamaños para adoptar carácter responsive mediante "sizes"

```html
<template>
    <nuxt-img
        src="/image.png"
        sizes="sm:100px md:300px lg:900px" <!-- Dynamic sizing depending on viewport  real state-->
        format="webp"
    />
</template>
```

---

-   Algunas otras acciones que se pueden tomar sobre las imágenes (depende del soporte del proveedor) son:
    -   Grayscale
    -   Blur
    -   Smart cropping
    -   Focal points
    -   Image Rounding
    -   Rotation
    -   etc

---

-   Estas opciones se pasan mediante la prop :modifiers

```html
<template>
    <nuxt-img
        src="/image.png"
        sizes="sm:100px md:300px lg:900px"
        format="webp"
        :modifiers="{
        filters: {
            blur: 10,
            rotate: 90
        }
    }"
    />
</template>
```

---

<!-- _class: lead -->

Índice

-   1. Network Status Based UI
-   2. Image Optimization with Nuxt Image
-   **_3. Rendering performance_**
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
-   4. One Line Animations

---

<!-- _class: lead -->

# 3. Rendering Performance

---

**Rendering Performance**

-   El performance de renderizado es una métrica muy importante para las aplicaciones web

-   Por cada segundo que nuestra app tarda en renderizar, más probable que el usuario salga de la misma

-   La aplicación ha de sentirse responsive ante el input de los usuarios

-   Vue tiene por defecto un performance bastante bueno

-   Aún así, hay maneras en las que podemos mejorar este performance

---

1. Virtual Lists
2. Eliminación de re-renderizaciónes innecesarias
    - v-once
    - v-memo

---

<!-- _class: lead -->

Índice

-   1. Network Status Based UI
-   2. Image Optimization with Nuxt Image
-   3. Rendering performance
    -   **_3.1 Rendering Performance with Virtual Lists_**
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
-   4. One Line Animations

---

**3.1 - Rendering Performance with Virtual Lists**

-   Si intentamos recorrer una lista enorme con v-for, tendremos un impacto en el performance de nuestra app. Tendremos tanto una primera carga lenta, cómo un scroll muy laggy.

-   La solución a esta situación pasa por el **_virtual scrolling_**, en concreto para listas, por el **_list virtualization_**

-   Renderizaremos divs que estén cerca del viewport actual

-   Lo más común es combinar el virtual scrolling con el infinite scrolling (paginación +1 a la llegada al final de la lista actual)

-   Visualización scroll v-for vs. virtual scroll

---

![bg w:400](./assets/12.png)
![bg w:500](./assets/13.png)

---

-   Hay varias maneras de lograr el efecto deseado, pero en este caso vamos a ver la opción con el composable **_useVirtualList_** de VueUse

-   Lo combinaríamos con **_useInfiniteScroll_** para lograr un efecto similar al que hacer twitter.

-   El uso del composable **_useVirtualList_** nos ahorra realizar muchos cálculos y tiempo a la hora de pensar cómo hacer para que la zona a renderizar se corresponda correctamente con el scrollbar (vs el total de elementos de nuestra lista)

-   Afortunadamente, este tipo de soluciones tan testadas y recomendadas nos permiten obtener ganancias de rendimiento de manera sencilla. Tendremos que instalar [VueUse](https://vueuse.org/)

---

```javascript
<script setup>
import { ref } from 'vue'
import { useVirtualList } from '@vueuse/core'

// Data -> [50] of loreipsum
// We can use it as a ref or not. It's the same
const data = ref(Array.from(Array(50).keys(), () => 'Lorem Ipsum'))

// Properties returned by useVirtualList composable
// list -> items that should actually be shown
// containerProps -> ref for container element, inline styles, onScroll event
// wrapperProps -> styles for our wrapper (like height + margin)

const {list, containerProps, wrapperProps} = useVirtualList(data, {
    itemHeight: 96 // Used for calculations
    overscan: 5 // Numb of items to be rendered outside the viewport (up n down). Smoother scroll.
})
</script>
```

---

```javascript
<template>
    <div v-bind="containerProps"> <!-- containerProps ok! -->
        <div v-bind="wrapperProps"> <!-- wrapperProps ok! -->
            <div
                v-for="{index, data} in list"
                :key="index"
                class="item-card-class" <!-- Height adds up to itemHeight! -->
            >
                <h2 class="h2-class">Item #{{index}}</h2>
                <p class="p-class">{{data}}</p>
            </div>
        </div>
    </div>
</template>
```

---

![bg w:1000](./assets/14.png)

---

-   Default overscan is 5

![bg w:300](./assets/15.png)
![bg w:300](./assets/16.png)

---

![bg w:1000](./assets/18.gif)

---

<!-- _class: lead -->

Índice

-   1. Network Status Based UI
-   2. Image Optimization with Nuxt Image
-   3. Rendering performance
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
        -   **_3.2.1 - v-once_**
        -   3.2.1 - v-memo
-   4. One Line Animations

---

**3.2.1 - Rendering Performance with v-once**

-   Renderiza el elemento o componente una sola vez. Después de ese render inicial, ese elemento/componente y todos sus hijos, serán tratados cómo estáticos

![w:600](./assets/19.png)

---

-   Para lograrlo, simplemente:

```html
<!-- v-once in a single html element -->
<p v-once>{{myStaticMessage}}</p>

<!-- v-once in an element with children -->
<div v-once>
    <p>{{message}}</p>
</div>

<!-- v-once in a component -->
<my-component v-once />

<!-- v-once in a v-for directive -->
<ul>
    <li v-for="item in list" :key="item" v-once>{{item}}</li>
</ul>
```

---

-   Consideraciones de v-once

    -   Tratar con cuidado esta directiva. Aunque cambien los datos reactivos, el componente no se re-renderizará
    -   La combinación de v-once con otras directivas puede tener efectos no-deseados, por la prioridad de este.

    ```html
    <!-- v-once with conditional components -->
    <p v-if="show" v-once>{{myStaticMessage}}</p>
    ```

    -   En el ejemplo, aunque cambie el show, el componente no reaccionará

-   Utilizaremos v-once sólo cuándo sepamos que un componente no se ha de re-renderizar aunque los datos se actualizen.

---

<!-- _class: lead -->

Índice

-   1. Network Status Based UI
-   2. Image Optimization with Nuxt Image
-   3. Rendering performance
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
        -   3.2.1 - v-once
        -   **_3.2.1 - v-memo_**
-   4. One Line Animations

---

**3.2.2 - Rendering Performance with v-memo**

-   Si queremos limitar cuándo un elemento se ha de re-renderizar, pero sin limitarlo sólo a la primera vez, utilizaremos v-memo

-   v-memo memoriza un sub-arbol de hijos de un componente y almacena renderizaciones previas para mejorar el performance

-   Lo podemos utilizar en cualquier elemento y acepta un array de dependencias. Limitará la re-renderización a cuándo alguna de esas dependencias cambien. Sin aarray lo hace "smart reactive"

-   Importante: v-memo no funciona dentro de v-for, pero si al mismo nivel (en el mismo elemento que v-for si, pero no dentro)

---

-   El <p> se re-renderizará reactivamente ante el cambio de msg o de count

```html
<!-- v-memo and some dependencies -->
<p v-memo="[msg, count]">{{msg}}, {{count}}</p>
```

-   Si utilizamos v-memo pero pasándole un array de dependencias vacío, obtenemos el mismo resultado que utilizando v-once, ya que no hay nada que haga trigger del re-render. Si no pasamos array, lo hace de manera inteligente

![w:850](./assets/20.png)

---

-   La documentación oficial considera que v-memo debería de ser utilizado sólo en micro-optimizaciones en escenarios de performance crítico (no utilizarlo by-default)

-   El uso más popular es a la hora de renderizar listas con muchos registros (length > 1000)

---

<!-- _class: lead -->

Índice

-   1. Network Status Based UI
-   2. Image Optimization with Nuxt Image
-   3.  Rendering performance
    -   3.1 Rendering Performance with Virtual Lists
    -   3.2 Rendering Performance with directives (avoid unnecessary re-render)
-   **_4. One Line Animations_**

---

<!-- _class: lead -->

# 4. One-Line Animations [Auto Animate](https://auto-animate.formkit.com/)

---

**One Line Animations**

-   Otra mejora que podemos llevar a cabo en nuestra aplicación es la del aspecto estético (con animaciones por ejemplo)

-   Para ser óptimos, tanto con las animaciones (performance) cómo con el tiempo que nos lleva implementarlas, nos podemos servir de herramientas.

-   Echaremos un ojo a [Auto Animate](https://auto-animate.formkit.com/)

-   Destaca por ser **zero** config y muy óptimo en cuánto a rendimiento. Además, funciona con cualquier app JavaScript (no está limitado a VueJS)

-   Podemos importarla globar o localmente. Expondrá directiva.

---

Global

![bg w:1050](./assets/21.png)

---

Local

![bg w:950](./assets/22.png)

---

-   Esencialmente, autoAnimate es una función que acepta un elemento, para luego animar ese elemento padre y sus hijos inmediatos cuándo:
    -   Elemento hijo se añade al DOM
    -   Elemento hijo se elimina del DOM
    -   Elemento hijo se mueve en el DOM

![w:550](./assets/23.png)

---

-   Ejemplo en que contamos con un div "header", el cual al ser clickado nos "abre" / "muestra" un <p>. La diferencia entre nos "muestra" y nos "abre", nos la aporta auto animate, utilizando un par de líneas.

---

```javascript
<script setup>
import { vAutoAnimate } from '@formkit/auto-animate' <!--import-->
import { ref } from 'vue'

const isOpen = ref(false)
</script>

<template>
    <div v-auto-animate class="p8 rounded-lg bg-neutral-900"> <!--apply-->
        <h2
            @click="isOpen = !isOpen" class="text-xl font-bold">
            Card Header
            <p v-if="isOpen" class="py-8">Card content</p>
        </h2>
    </div>
</template>
```

---

-   Nota: Además de poder utilizar como directiva, contamos con un **composable**

```javascript
<script setup>
import { vAutoAnimate } from '@formkit/auto-animate'
import { ref } from 'vue'
const isOpen = ref(false)
const [wrapperEl] = useAutoAnimate()                                     //Composable
</script>

<template>
    <div ref="wrapperEl" class="p8 rounded-lg bg-neutral-900">
        <h2
            @click="isOpen = !isOpen" class="text-xl font-bold">
            Card Header
            <p v-if="isOpen" class="py-8">Card content</p>
        </h2>
    </div>
</template>
```

---

-   Aunque simplemente funciona out-of-the-box, también podremos especificar opciones o incluso utilizar plugins.

```javascript
<template>
    <div v-auto-animate="{duration: 1000, easing:"linear"}">
    </div>
</template>
```

-   Para más información, consultar la [documentación](https://auto-animate.formkit.com/)

---

<!-- _class: lead -->

# Conclusión

---

**Conclusión**

-   Vue nos da una base muy sólida para desarrollar proyectos bastante optimizados en cuánto a performance, a la vez que nos aporta la libertad necesaria para hacer lo que queramos

-   Está muy bien contar con opciones de mejoras de rendimiento tanto nativas de JS, nativas de vue o de terceros.

-   Es importante priorizar el performance de nuestra webapp para aportar una experiencia de usuario responsive y agradable.

-   Estas son sólo algunos ejemplos, hay todo un mundo dedicado a este tipo de mejoras. Destaca el canal de youtube: [LearnVue](https://www.youtube.com/@LearnVue/)

---

<!-- _class: lead -->

# Fin.
