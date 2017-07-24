# Capítulo 9. Conceptos avanzados

Con lo aprendido hasta ahora sobre `vue-router`, podríamos cubrir gran parte de la funcionalidad necesaria para un buen número de aplicaciones.

Sin embargo, cuando nos enfrentamos a aplicaciones más grandes, tener en cuenta otras posibilidades nos puede ayudar en términos de reutilización y buenas prácticas.

El posts de hoy está dedicado a estudiar todos aquellos conceptos avanzados de vue-router que pueden ayudarnos a mejorar y a darnos mayor versatilidad cuando desarrollamos en un proyecto con vue.

Para conseguir esto, nos centraremos en el anidamiento de rutas, el paso de propiedades a un ComponentView, la inclusión de meta información y el cambio de comportamiento del scroll de nuestra web. Vayamos con ello:


## Anidar rutas

Durante esta serie hemos hecho ejemplos con rutas bastantes simples. Los sistemas de navegación de los ejemplos siempre han sido de una ruta a otra pero, ¿qué ocurre cuando dentro de nuestro sistema existe una navegación entre subrutas?

Imaginemos por un momento, que tenemos que desarrollar una aplicación bancaria y que estamos implementando la funcionalidad de transferencias entre cuentas. Nuestra aplicación podría contar, por ejemplo, con un proceso dividido en 3 pantallas:

* Una pantalla para configurar los datos de la transferencia: En esta pantalla, el usuario indica el importe y el destinatario al que realizar la transferencia.
* Una pantalla para mostrar el detalle de la transferencia: Nos puede ser muy útil para mostrar el estado actual de la cuenta, el día en que se hará la transferencia y el cálculo de las comisiones que conlleva la operación.
* Y una pantalla de confirmación de la transferencia realizada: Se le muestra al usuario cuando toda la operación de transferencia fue correctamente.

Para realizar esto, podríamos diseñar 3 vistas que se corresponderían con estas rutas:

```
/transfer/config
/transfer/detail
/transfer/confirm
```

Como vemos, la propia funcionalidad, me lleva a tener un subenrutado o anidamiento de rutas.

Estos procesos suelen conllevar una interfaz parecida en cada vista para favorecer la navegación al usuario. Por ejemplo, las 3 vistas van a compartir unas cabeceras y un componente de navegación que nos indique en qué paso de la realización de la transferencia nos encontramos.

Para desarrollar esto, podemos hacerlo de 3 maneras diferentes:

1. Creando un HTML diferente para cada una de las vistas.
2. Extrayendo todo el HTML común y creando componentes que se utilizasen en las 3 vistas.
3. Creando una 'layout' que contenga toda la parte común a las 3 vistas e ir rellenando de manera dinámica la diferencia entre vistas.

La opción 1 la descartamos porque no respeta los principios de reutilización. Si necesito hacer un cambio en cualquier momento, tengo que cambiar 3 vistas.

La opción 2 es bastante buena, pero no llega a ser lo suficientemente reutilizable; Sí, estamos reutilizando componentes comunes, pero aún hay partes que pueden cambiar en el tiempo y que me hagan trabajar más de la cuenta, manteniendo código idéntico.

La opción 3 es la mejor porque respeta los principios de reutilización ya que toda la parte común se encuentra encapsulada en un componente padre que orquesta a 3 componentes hijos.

La idea es hacer algo como esto:

![Ejemplo de las vistas con layouts](/images/router-avanzado/untitled-diagram.png)

Desarrollar un componente común que se llame ​`TransferView` que haga de componente contenedor, y 3 componentes hijos, más específicos, llamados `ConfigView`, `DetailView` y `ConfirmView`.

Aunque con vue tendríamos soporte para desarrollar esto, vue-router nos da una funcionalidad que se apoya en un sistema de rutas anidadas. Yo podría configurar mi ruta de la siguiente manera:

```javascript
const TransferView = {
    template: `
        <div>
            <h2>Cabecera func. transferencia</h2>
            <router-view></router-view>
        </div>
    ` 
}; 

const ConfigView = { 
    template: '<h3>Paso configuración</h3>' 
}; 

const DetailView = { 
    template: '<h3>Paso detalle</h3>' 
}; 

const ConfirmView = { 
    template: '<h3>Paso confirmación</h3>' 
}; 

const router = new VueRouter({ 
    routes: [ 
        { 
            path: '/transfer', 
            component: TransferView, 
            children: [ 
                { path: 'config', component: ConfigView }, 
                { path: 'detail', component: DetailView },
                { path: 'confirm', component: ConfirmView }
            ] 
        } 
    ] 
});
```
 
Lo que hacemos es incluir un nuevo parámetro en la ruta llamado children. En este campo, podemos configurar todas las rutas anidadas que necesitemos. En nuestro caso 3.

Lo último que hemos hecho es añadir el componente `router-view` en nuestro componente padre `TransferView`. De esta manera vue-router sabe en qué parte tiene que renderizar el componente hijo.

Este anidamiento puede ser todo lo profundo que queramos, simplemente tenemos que incluir el parámetro `children`, configurar las rutas con su componente hijo que queramos e indicar en el componente padre dónde pintarlo por medio de `router-view`.

El anidamiento suele utilizarse también para crear la layout principal de nuestra aplicación. Como todas las vistas contarán con el header, el menú y el footer, lo que se hace es crear un componente `AppView` que contiene estos elementos y un componente `router-view` donde se renderizará la parte de la vista más específica.

## Pasar propiedades a un ComponentView

Aunque un `ComponentView` es un componente muy específico dentro de nuestra aplicación, y es bastante improbable que pueda ser reutilizado por estar tan acoplado con el negocio, es buena idea usar las mismas buenas prácticas que usamos en componentes más específicos.

Cuando un componente de vista hace uso de parámetros para su correcto funcionamiento, es necesario que no lo acoplemos ,dentro de lo posible, a `vue-router` y que usemos la funcionalidad de propiedades de entrada especificada por vue.

Por tanto, intentemos evitar este tipo de accesos:

```javascript
const ProductDetailView = { 
  template: '<label>ProductId {{ $route.params.productId }}</label>' 
}; 

const router = new VueRouter({ 
  routes: [{ 
    path: '/products/:id', 
    component: ProductDetailView 
  }] 
});
```
 
Por el siguiente:

```javascript
const ProductDetailView = { 
  props: ['productId'],
  template: '<label>ProductId {{ productId }}</label>' 
}; 

const router = new VueRouter({ 
  routes: [{ 
    path: '/products/:id', 
    component: ProductDetailView, 
    props: true 
  }] 
});
```

Lo que hemos hecho es decir a la ruta que active el paso de parámetros por medio de propiedades al componente. De esta forma el componente está más desacoplado y nos ayuda a su reutilización y a ser probado de manera aislada.

El atributo props nos permite varios valores según nuestras necesidades. Por ejemplo, puedo pasar un objeto para 'mapear' manualmente las propiedades con los parámetros. O incluso, puede que la ruta no cuente con parámetros, pero sí con propiedades que hay que iniciar. Este caso nos puede ser útil.

Un caso puede ser este.

```javascript
const router = new VueRouter({ 
  routes: [ 
    { 
      path: '/promotion/from-newsletter', 
      component: PromotionView, 
      props: { newsletterPopup: false } 
    } 
  ] 
}); 
```
 
Hacemos que el popup del componente no se muestre al principio al navegar a esta ruta. No recibimos parámetros de la ruta, pero si pasamos propiedades al componente.

Puedo pasar también una función. Nos puede ayudar a tomar decisiones sobre el valor que quiero incluir en la propiedad del componente dependiendo de la ruta. Por ejemplo:

```javascript
const router = new VueRouter({ 
    routes: [ 
        { 
            path: '/search', 
            component: SearchView, 
            props: (route) => ({ query: route.query.q }) 
        } 
    ] 
});
```
 
En este caso usamos la función para incluir la query realizada en la url como propiedad del componente `SearchView`.

## Incluir meta-información de una ruta

Puede darse el caso que necesitemos incluir información específica para una ruta. Puede sernos útil marcar ciertas rutas para influir en su comportamiento. Esto se puede hacer incluyendo nuevos campos en el atributo `meta` del objeto `route` de la siguiente manera:

```javascript
const router = new VueRouter({
    routes: [
        {
            path: '/login',
            component: LoginView,
            meta: {
                isPublic: true
            }
        }
    ]
});
```
 
De esta manera estamos marcando que la ruta login es pública para cualquier usuario. Hay que tener cuidado con esta meta información porque es heredada de padres a hijos por lo que tengámoslo en cuenta si pensamos que algo está yendo mal.

Esta información es inyectada a los componentes dentro de `$route.matched` y es accesible tanto dentro de los componentes como de los interceptores de navegación. Por ejemplo, podemos combinar este campo con el interceptor `beforeEach`:

```javascript
router.beforeEach((to, from, next) => { 
    if (!to.matched.some(record => record.meta.isPublic) && !auth.loggedIn()) { 
        next({ 
           path: '/login', query: { redirect: to.fullPath } 
        });
    } else {
        next();
    }
});
```
 
Si la ruta no es pública, ni el usuario tiene autorización en el sistema, se devuelve al usuario a la pantalla de login. Esto se ejecutará para todas las rutas a las que naveguemos.

## Poner nuestro sistema de rutas en modo History de HTML5

Un SPA suele contar con un sistema de rutas precedido por una almohadilla. De esta forma, el navegador sabe que no tiene que resolver la ruta contra un servidor, ni hace una recarga de la página, sino que intenta solucionar la ruta a nivel interno de navegador. De esta manera, la librería de rutas intercepta el nuevo valor y actúa según su configuración.

Los que ya hemos trabajado con otros SPAs sabemos que las rutas suelen tener esta apariencia:

```
https://my-web.com/#/app/login
```

Evitar una URL como esta puede deberse a 3 razones:

* Puede que desde negocio se quiera trabajar en un sistema de rutas usable para que el usuario pueda recordarlas o guardarlas en favoritos.

* Puede que no nos interese como desarrolladores dar pistas al usuario del tipo de herramientas que usamos para nuestro desarrollo (Cuando en una web se ve un hash de este tipo es indicativo de SPA).

* Puede que tengamos problemas de SEO al no poder acceder a ciertas rutas.

Sería bastante bueno que nuestra librería pudiese entender correctamente la siguiente ruta:

```
https://my-web.com/app/login
```

Esto es posible en vue gracias a nuestra librería de rutas `vue-router`. Yo puedo configurarlo de esta manera para que se comporte de forma nativa a como lo hace el navegador:

```javascript
const router = new VueRouter({ 
    mode: 'history', 
    routes: [...] 
});
```
 
De esta forma, tenemos el comportamiento esperado. Lo malo de esto es que ahora todas las rutas harán una petición a nuestro servidor de aplicaciones y que dependiendo de la herramienta que usemos, el error será gestionado de una u otra manera, pudiendo romper el funcionamiento correcto de nuestra aplicación.

Este mecanismo se puede configurar de diferentes formas. Por ejemplo, si nuestra aplicación es servida por un Apache, debemos registrar la siguiente regla en su configuración:

```xml
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

Esta regla nos redirige a index.html cada vez que no encuentra la ruta especificada. De esta forma, la aplicación sabrá reiniciarse correctamente.

Si usamos un NGINX, lo deberíamos hacer así:

```
location / { 
    try_files $uri $uri/ /index.html; 
}
```

Si queremos que el propio NodeJS nos gestione esto, contamos con un middleware de Express que nos permite configurar este redireccionamiento a nivel de servidor: connect-history-api-fallback.

El problema que seguimos teniendo con esto es que si la ruta no existe se nos seguirá redirigiendo a index.html no dando información al usuario de que esa ruta no existe. Para solucionar esto podemos registrar una ruta genérica en vue-router que siempre se ejecutará cuando ninguna otra regla haya conseguido relacionarse:

```javascript
const router = new VueRouter({ 
    mode: 'history', 
    routes: [{ 
        path: '*', 
        component: NotFoundView 
    }] 
});
```
 
Para cualquier ruta (wildcard *), mostramos el componente `NotFoundView`. De esta forma siempre damos una información adecuada al usuario.

## Cambiar el comportamiento del scroll

Otro 'daño colateral' de usar el modo histórico de HTML5 y no el comportamiento de URLs por defecto de un SPA, es que podemos gestionar en qué parte del scroll queremos colocar nuestra nueva vista; Cuando naveguemos podemos preferir que la nueva vista siempre se sitúe en su parte superior o que mantenga el scroll de la ruta de la que procedemos.

En el objeto router contamos con otro parámetro para gestionar esto:

```javascript
const router = new VueRouter({ 
    routes: [...], 
    scrollBehavior (to, from, savedPosition) {
        return { x: 0, y: 0 }
    }
});
```
 
Esta función `scrollBehavior` nos permite acceder a los datos de la ruta de la que vengo y de la que voy. Además, se cuenta con un parámetro opcional que solo es inyectado si el cambio de ruta es provocado por los botones de navegación del propio navegador.

En el ejemplo de arriba, hemos indicado que scroll siempre se sitúe en la parte superior e izquierda de la página. Puede darse el caso que queramos simular una navegación hasta un 'anchor' determinado. Esto lo podríamos conseguir así:

```javascript
scrollBehavior (to, from, savedPosition) { 
    if (to.hash) { 
        return { 
            selector: to.hash 
        } 
    } 
}
```
 
Esta función tiene acceso a los datos de la meta información de una ruta, por lo que el comportamiento del scroll es tan configurable como nosotros necesitemos.

> Nota: Esta funcionalidad, como decíamos, solo funciona con el modo 'history' activo.

## Todo junto

Ahora vamos a juntar todos estos conocimiento y a hacer un pequeño ejemplo. Vamos a implementar del todo la funcionalidad de transferencias de apartados anteriores.

Lo primero que hacemos es crear el servidor de NodeJS que servirá nuestra aplicación:

```javascript
const express = require('express');
const history = require('connect-history-api-fallback');
const app = express();

app.use(history());
app.use(express.static('public'));

const server = app.listen(3001, "localhost", () => {
    const host = server.address().address;
    const port = server.address().port;
    console.log('Running at http://' + host + ':' + port);
});
```
 
Hemos incluido el fallback del histórico ya que nuestro ejemplo estará configurado con modo histórico de HTML5, el modo que ponía las URLs 'bonitas', recuerda.

Lo siguiente es desarrollar los componentes y la configuración de rutas:

```javascript
const TransferView = {
    template: `
        <div>
            <h2>Transferencia nacional</h2>
            <router-view></router-view>
        </div>
    `
};

const ConfigView = {
    data() {
        return {
            amount: 0,
            iban: 'ES671234567898761232'
        }
    },
    template: `
        <div>
            <h3>Configurar transferencia</h3>
            <form @submit.prevent="showDetail">
                <label for="amount">Cantidad</label>
                <input id="amount" type="text" v-model="amount"/>
                <label for="iban">IBAN</label>
                <input id="iban" type="text" v-model="iban"/>
                <button>Realizar transferencia</button>
            </form>
        </div>
    `,
    methods: {
        showDetail() {
            this.$router.push({
                name: 'detail',
                params: { amount: this.amount, iban: this.iban }
            });
        }
    }
};

const DetailView = {
    props: ['amount', 'iban'],
    template: `
        <div>
            <h3>Detalle de la transferencia que va a realizar</h3>
            <p>Cantidad: {{ amount }} €</p>
            <p>IBAN: {{ iban }}</p>
            <router-link :to="{ name: 'confirm' }">Confirmar</router-link>
        </div>
    `
};

const ConfirmView = {
    template: `
        <div>
            <h3>Transferencia realizada correctamente</h3>
            <router-link to="/">Volver</router-link>
        </div>
    `
};

const NotFoundView = {
    template: `
        <div>
            <router-link :to="{ name: 'config' }">Empezar transferencia</router-link>
        </div>
    `
};

const routes = [
    {
        path: '/transfer',
        component: TransferView,
        children: [
            { path: 'config', name: 'config', component: ConfigView },
            { path: 'detail', name: 'detail', component: DetailView, props: true },
            { path: 'confirm', name: 'confirm', component: ConfirmView },
        ]
    },
    { path: '*', name: 'not-found', component: NotFoundView }
];


const router = new VueRouter({
    mode: 'history',
    routes,
    scrollBehavior(to, from, savedPosition) {
        return { x: 0, y: 0 }
    }
});

const app = new Vue({ router }).$mount('#app');
```
 
Si vemos el ejemplo con detalle, veremos que se incluye la funcionalidad de history mode HTML5, el anidamiento de rutas, el paso de parámetros como propiedad y el control del scroll.

## Conclusión

A lo largo de estos 3 últimos posts hemos hecho un repaso a todo lo que puede aportarnos una librería como `vue-router`. No es una librería innovadora, ni lo necesita ser, simplemente es una opción que se integra perfectamente con el ecosistema de vue para la gestión de rutas.

Terminado este capítulo, entraremos en una nueva fase de la serie donde estudiaremos la gestión del estado en una aplicación. Estamos muy cerca de contar con todas las piezas necesarias para poder hacer una aplicación completa, robusta y escalable con el ecosistema de vue.

Por ahora la experiencia está mereciendo la pena y lo aprendido es coherente con lo que muchos fronts han demandado a lo largo de los años a un framework JavaScript.

Nos leemos :)
