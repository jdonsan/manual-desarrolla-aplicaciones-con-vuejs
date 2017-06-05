# Capítulo 7. Introduciendo rutas en nuestra aplicación

Dejamos por un momento los componentes y nos centramos en una nueva funcionalidad del framework: el enrutamiento. Una vez que hemos creado nuestros componentes base o comunes (botones, inputs, listados, items, cartas...) y nuestros componentes de negocio (formularios, widgets, buscadores...) puede ser que necesitemos crear componentes vista o página para crear una aplicación completa (un SPA).

Dependiendo del número de vistas de las que se componga mi aplicación, la navegación entre ellas será más o menos fácil. Por ello, cuando una aplicación empieza a crecer en el número de vistas, es buena idea incluir algún mecanismo o herramienta que nos permita escalar este problema de la manera adecuada.

En el post de hoy - y en los sucesivos - veremos las formas en las que podemos incluir un sistema de navegación en nuestra aplicación VueJS de una manera escalable y poco intrusiva. Siganme, por favor:

Leer más…

¿Qué es un sistema de rutas en mi aplicación?

Es un sistema que permite configurar la navegación de nuestra aplicación. Suele componerse de una librería que se encuentra interceptando la ruta que indicamos en nuestro navegador para saber en todo momento a qué estado de la aplicación debe moverse.

Un enrutador nos permite decir, para una url determinada, que componente renderizar. Está muy basado en los sistemas de Modelo-Vista-Controlador y el diseño de API Rest. Suelen ser muy útiles para gestionar de una manera centralizada el comportamiento y el viaje que debe llevar un usuario por nuestra aplicación. Al final no deja de ser una forma de solucionar las diferentes direcciones web con las que cuenta mi aplicación en un sistema SPA.

¿Y si no necesitamos un sistema de rutas?

En muchas ocasiones, incluimos una librería compleja de gestión de la navegación sin plantearnos si quiera si lo necesitamos. Imaginad que hemos desarrollado una pequeña web donde presentamos nuestro producto, la típica web estática que no presenta más de 7 u 8 páginas diferentes.

Si por un casual, hemos decidido desarrollarla con VueJS, puede ser tentador usar su librería hermana vue-router. Sin embargo, quizá añadamos complejidad sin necesidad.

Y si no añadimos un enrutador ¿Cómo lo hacemos? Podemos preparar nosotros una pequeña solución que permita este dinamismo. Por ejemplo, con algo parecido a esto:

 

 
Lo que hacemos, en este caso, es apoyarnos de la funcionalidad de propiedades computadas que nos ofrece el framework  para conseguir dinamismo. Lo que conseguimos es que cada vez que la variable currentRoute cambie, se ejecute la función ViewComponent que devuelve el componente que hayamos configurado en nuestro array routes. Si la ruta puesta en el navegador no es correcta, renderizamos el componente NotFound.

La implementación es bastante sencilla y nos va a permitir la navegación por nuestra aplicación sin hacer mucho más. Si nuestra aplicación empieza a crecer, tenemos que tener en cuenta que una solución como esta es limitada y que deberemos ir pensando en incluir algo más elaborado.

Y si lo necesitamos ¿Cómo empezamos?

Si necesitamos algo más elaborado, puede ser una buena idea que diseñemos nuestra propia librería como hace Juanma en este post, o que usemos una de las librerías con las que ya contamos en la comunidad como es el caso de Director.

Tenemos también la opción de extender VueJS con una nueva librería llamada vue-router, esta es la opción que vamos a explicar. Es la opción que mejor se adapta al propio framework y que nos va a proporcionar todo lo necesario.

Para usarla tenemos varias formas: si es un proyecto que ya tenemos empezado, por medio de la línea de comandos:

$ npm install vue-router --save
O creando un proyecto desde 0 con vue-cli, indicando en la configuración del proyecto, que queremos una plantilla con vue-router, como ya explicamos en el post anterior.

Una vez que tenemos esto, lo siguiente será configurar nuestras rutas e indicar a VueJS que incluya esta configuración en su contexto. Para hacer esto, creamos una carpeta en la raíz del proyecto de esta manera:

Captura de pantalla de 2017-05-17 11-42-53.png

Dentro del fichero index.js vamos a ir incluyendo toda la configuración de rutas de nuestra aplicación. Dentro de este fichero incluimos las siguientes lineas.

import Vue from 'vue';
import Router from 'vue-router';

Vue.use(Router);

export default new Router({});
Lo que estamos haciendo es importar tanto la librería de vue como la de vue-router. Lo siguiente es extender VueJS por medio de Vue.use(Router). De esta manera, extendemos la funcionalidad con un plugin. Por último, devolvemos la instancia del router que vamos a configurar.
 
Para terminar de integrar vue-router totalmente, solo nos falta inyectar esta instancia en todos los componentes. Esto lo conseguimos yendo al fichero main.js, donde se encuentra el 'setup' inicial de mi aplicación vue. Dentro ponemos lo siguiente:
 
import Vue from 'vue';
import router from './router';
import App from './components/app/app.vue';

new Vue({
    el: '#app',
    router,
    render: h => h(App)
});
Lo único que hacemos es inyectar en la instancia principal de nuestra aplicación vue, nuestro router para que sea accesible a todo el árbol de componentes.
 
Si ahora queremos que estos componentes se pinten, vue-router cuenta con un componente específico donde se irá incluyendo el componente que la ruta nos indique. En nuestro componente  app, hay que añadir el componente  <router-view>. Lo único que hace este componente es sustituirse por nuestra vista.
 
Ya está. No necesitamos más fontanería. Ya podemos empezar a configurar rutas.
¿Cómo configuramos rutas?

Dentro de una instancia del router contamos con un parámetro llamado routes que permite configurar nuestras rutas. Por ejemplo, podríamos hacer lo siguiente:

// router/index.js
import HomeView from '@/components/views/home-view.vue';

export default new Router({
    routes: [
        { path: '/home', component: HomeView }
    ]
});
De esta forma, cuando un usuario ponga en el navegador la ruta www.mi-spa.com/#/home, vue renderizará mi componente HomeView.

Dentro de este objeto podemos incluir otro nuevo parámetro llamado name. Este parámetro esta muy bien para dar un nombre a nuestra ruta. De esta forma los desarrolladores desacoplan la url física del estado al que nos tenemos que dirigir y hace que podamos renombrar rutas muy largas. Por ejemplo:

// router/index.js
import HomeView from '@/components/views/home-view.vue';

export default new Router({
    routes: [
        { 
            path: 'my/shop/detail/go/www/43544352/app/home', 
            name: 'home',
            component: HomeView 
        }
    ]
});
Ahora podemos usar el nombre para referenciarla sin tener que usar toda la ruta. Es recomendable siempre ponerlo y usar ese nombre dentro de nuestra app.

Cómo decíamos, al final nuestro sistema de rutas está muy pensado para cargar estados o recursos en nuestros componentes de página y por ello, además de rutas estáticas, podemos usar rutas dinámicas. Esto significa que yo puedo indicar que se renderice siempre un componente para una serie de rutas que tienen patrones en común. Por ejemplo:

// router/index.js
import ProductDetailView from '@/components/views/product-view.vue';

export default new Router({
    routes: [
        { 
            path: 'products/:productId', 
            name: 'product-detail',
            component: ProductDetailView 
        }
    ]
});
Lo que hemos conseguido con esto es que tanto /products/1234 como products/3452 nos renderice el mismo componente. Siempre que queramos incluir una parte dinámica a nuestra ruta, tenemos que indicarlo con dos puntos :.

Este dinamismo nos puede ser muy útil para obtener productos por un id determinado de servidor pues, la parte dinámica, es inyectada dentro de nuestros componentes en el campo $route.params.

Con este comportamiento se puede hacer cualquier cosa que se nos ocurra ya que vue-router usa la librería path-to-regexp para relacionar rutas por medio de expresiones regulares. Si necesitas algo mucho más específico sería bueno que le echases una ojeada.

Puedes preguntarte qué ocurriría si más de una ruta de las que has configurado coincide con la ruta especificada por el usuario. Las rutas son registradas en vue por orden en el array. Por tanto la prioridad será el orden en la que se especificó. En cuanto vue-router encuentra una coincidencia recorriendo el arreglo, ejecuta su renderizado. Tenlo en cuenta.

¿Cómo navegamos a nuevas rutas?

Una vez que hemos configurado nuestras rutas, podemos navegar entre ellas para que el usuario pueda realizar las acciones necesarias. Esta navegación la podemos hacer de dos maneras: de manera semántica por medio del componente <router-link> o por manera programática.

Si lo hacemos de manera semántica tendríamos que hacerlo de esta manera dentro de nuestros templates:

<router-link to="/home">Voy a home</router-link>
Esto se renderiza por lo siguiente:

<a href="#/home">Voy a home</a>
El parámetro to nos acepta un objeto como valor para conseguir ciertas cosas, por ejemplo indicar parámetros. Por ejemplo, con el caso anterior de los productos, yo puedo hacer lo siguiente:

<router-link :to="{ name: 'product-detail', params: { productId: 1234 }}">
   Voy a ver el producto 1234
</router-link>
Ese objeto se puede definir dentro del JS perfectamente.

Si queremos crear navegaciones en tiempo de lógica del componente, contamos con 3 métodos para poder hacerlo. Yo puedo hacer lo siguiente:

$router.push('home');
Donde home es la ruta a la que me quiero dirigir. Este método tiene la siguiente firma:

$router.push(location, onComplete?, onAbort?);
Puede también que queramos navegar una ruta, pero que no queramos que se guarde en el histórico de navegación. En este caso usaríamos el método replace de esta manera:

$router.replace('home');
En este caso, iremos a la vista home y reemplazará a la ruta en la que estamos actualmente en el histórico.

Por último, contamos con un método llamado go que nos permite decir el número de saltos hacia delante o hacia detrás que queremos dar en el histórico de navegación:

$router.go(-1);
Esto iría a la ruta anteriormente visitada.

Una de las cosas que me gusta de esta API de navegación es que los nombres no han sido puesto de manera caprichosa. Si nos fijamos en ellos, son un mapeo 1 a 1 de la API History. De manera nativa los navegadores cuentan con window.history.pushState, window.history.replaceState y window.history.go. Esto hace que si hemos usado la API nativa, usarla en vue nos muy intuitivo.

Conclusión

Nunca se sabe de que manera puede crecer una aplicación, por lo que suele ser difícil de antemano saber si se va a necesitar un sistema como vue-router o no. Es por ello que la forma en la que el ecosistema de VueJS nos permite ir incluyendo estas pequeñas funcionalidades, de manera progresiva, me parece todo un acierto para el aprendizaje y para la complejidad de nuestro proyecto.

Lo bueno de un sistema de enrutado como este es que, por lo general, se cuenta con una API muy sencilla, y aprender su mecanismo suele costar poco. La parte más difícil a la hora de desarrollar nuestra aplicación se encuentra en la parte de diseño. La parte dónde tenemos que decidir como va a ser el flujo y la experiencia del usuario entre pantallas. Si tenemos claro en qué estado se tiene que encontrar en cada momento nuestro usuario, el resto es pan comido.

En los próximos posts, profundizaremos en la librería y aprenderemos sobre el ciclo de vida que tiene una ruta en nuestro sistema y en cómo sacar partido a sus hooks. Hasta el momento, esto es todo.

Nos leemos :)

En anteriores post de VueJS en El Abismo:

Introducción

VueJS: The Progressive JavaScript Framework
VueJS: Trabajando con templates
VueJS: Enlazando clases y estilos
Desarrollando con componentes

VueJS: Creando componentes
VueJS: El ciclo de vida de un componente
VueJS: Definiendo componentes en un único fichero
Las rutas en nuestro SPA

VueJS: Introduciendo rutas en nuestra aplicación
VueJS: Navigation Guards
VueJS: Conceptos avanzados de vue-router