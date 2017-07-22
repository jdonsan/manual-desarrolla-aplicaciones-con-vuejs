# Capítulo 1. The Progressive JavaScript Framework

Hoy comienza una nueva serie en El Abismo. Hoy empezamos un nuevo viaje hacia nuevas tierras tecnológicas. Hemos hablado largo y tendido en el blog sobre buenas prácticas, patrones, paradigmas y pequeños trucos que nos han ayudado a hacer mejor JavaScript. Creo que es hora de que aprovechemos todo ese conocimiento que hemos ido adquiriendo, y que lo empleemos en conocer mejor nuestras herramientas de trabajo.

Como desarrolladores, tenemos muchas herramientas de trabajo, pero nada suele ser tan importante como el conjunto de librerías o frameworks a los que nos atamos en los proyectos. Incluir estas dependencias en nuestro proyectos va a determinar la forma en la que trabajamos y el estilo que tendremos que usar. De ahí la importancia de elegir aquellas herramientas que realmente necesitamos y que nos harán sentirnos cómodos.

Como antes de poder incluir cualquier herramienta en producción, hay que probarla y aprenderla, creo que es un buen momento para que nos detengamos unas semanas y prestemos atención a una de las librerías JavaScript que más está llamando la atención en los últimos meses: VueJS.

No sé cuánto durará esta serie, así que lo mejor es que nos pongamos al lío y dediquemos nuestro tiempo a entenderla y a saber cómo se usa. ¿Te apetece unirte? ¿Estás con ganas de aprender la enésima librería de JavaScript? ¿Sí? Pues sigamos:

## ¿Qué es VueJS?

Vue (pronunciado como viu) es la nueva herramienta JavaScript creada por [Evan You](https://www.linkedin.com/in/evanyou/ "Currículum de Evan You"), miembro bastante conocido en la comunidad por participar en el desarrollo de [Meteor](https://www.meteor.com/ "Web oficcial de meteor") y por ser desarrollador de Google durante varios años.

Evan You define su herramienta [como un framework progresivo](http://slides.com/evanyou/progressive-javascript#/ "Slides de Evan You hablando sobre framework progresivo"). Progresivo porque el framework se encuentra dividido en diferentes librerías bien acotadas que tienen una responsabilidad específica. De esta manera, el desarrollador va incluyendo los diferentes módulos según las necesidades del contexto en el que se encuentre. No es necesario incluir toda la funcionalidad desde el principio como en el caso de frameworks como [Angular](https://github.com/angular/angular "Repositorio de Angular").

Es un sistema de modularización bastante parecido al de ReactJS. Facebook desarrolló un core para poder trabajar con vistas, pero a partir de ahí se han ido creando toda una serie de librerías (tanto por parte de Facebook como de la comunidad) que permite trabajar de una manera eficiente en un SPA. Aquí todas las piezas importantes se enmarcan dentro del proyecto de [VueJS](https://github.com/vuejs "Repositorios de VueJS") creado por Evan You.

> A lo largo de estos primeros capítulos nos centraremos en el estudio del core del framework, por ahora dejaremos de lado a [vue-router](https://github.com/vuejs/vue-router "Repositorio de vue-router") y a [vuex](https://github.com/vuejs/vuex "Repositorio de vuex"), aunque en algún momento llegaremos hasta ellos.

El core principal permite el desarrollo de componentes de UI por medio de JavaScript. La librería se enmarca dentro las arquitecturas de componentes (que tan de moda están) con una gestión interna de modelos basada en el patrón [MVVM](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel "Wikipedia del patrón MVVM"). Esto quiere decir que los componentes, internamente, tienen mecanismos de doble 'data-binding' para manipular el estado de la aplicación.

Presume de ser una librería bastante rápida que consigue renderizar en mejores tiempos que ReactJS. Estos son los tiempos que podemos encontrar en su documentación:

| Vue |	React | |
|-----|-------|-|
|23ms|	63ms |	Fastest|
|42ms |	81ms |	Median|
|51ms |	94ms |	Average|
|73ms | 164ms |	95th Perc.|
|343ms|	453ms|	Slowest|

> Como curiosidad: VueJS tiene un tamaño de 74.9 KB (la versión al hacerse el post es la v2.2.6).

## ¿Qué caracteriza a VueJS?

Pero ¿qué define a VueJS? ¿Qué lo diferencia o lo asemeja al resto de alternativas? ¿Por qué se está poniendo tan de moda? Intentemos explicar algunas de sus características para que vosotros mismos veáis si el framework tiene la potencia que nos dicen:

* **Proporciona componentes visuales de forma reactiva**. Piezas de UI bien encapsulados que exponen una API con propiedades de entrada y emisión de eventos. Los componentes reaccionan ante eventos masivos sin que el rendimiento se vea perjudicado.

* **Cuenta con conceptos de directivas, filtros y componentes bien diferenciados**. Iremos definiendo y explicando estos elementos a lo largo de la serie.

* **La API es pequeña y fácil de usar**. Nos tendremos que fiar por ahora si ellos lo dicen :)

* **Utiliza Virtual DOM**. Como las operaciones más costosas en JavaScript suelen ser las que operan con la API del DOM, y VueJS por su naturaleza reactiva se encontrará todo el rato haciendo cambios en el DOM, cuenta con una copia cacheada que se encarga de ir cambiando aquellas partes que son necesarias cambiar.

* **Externaliza [el ruteo](https://router.vuejs.org/en/ "Documentación de vue-router") y [la gestión de estado](https://vuex.vuejs.org/en/ "Documentación oficial de vuex")** en otras librerías.

* **Renderiza `templates` aunque [soporta JSX](https://vuejs.org/v2/guide/render-function.html#JSX "Documentación sobre el soporte de JSX en Vue")**. JSX es el lenguaje que usa React para renderizar la estructura de un componente. Es una especie de HTML + JS + vitaminas que nos permite, en teoría, escribir plantillas HTML con mayor potencia. VueJS da soporte a JSX, pero entiende que es mejor usar plantillas puras en HTML por su legibilidad, por su menor fricción para que maquetadores puedan trabajar con estos templates y por la posibilidad de usar herramientas de terceros que trabajen con estos templates más estándar.

* **Permite focalizar CSS para un componente específico**. Lo que nos permitirá crear contextos específicos para nuestros componentes. Todo esto sin perder potencia en cuanto a las reglas de CSS a utilizar. Podremos usar todas las reglas CSS3 con las que se cuentan.

* **Cuenta con un sistema de [efectos de transición](https://vuejs.org/v2/guide/transitions.html "Documentación de transiciones en Vue")** y animación.

* **Permite renderizar componentes para entornos nativos** (Android e iOS). Es un soporte por ahora algo inmaduro y en entornos de desarrollo, pero existe una herramienta creada por Alibaba llamada [Weex](https://weex.apache.org/ "Web oficial de Weex") que nos permitiría escribir componentes para Android o iOS con VueJS si lo necesitáramos.

* **Sigue un [flujo one-way data-binding](https://vuejs.org/images/props-events.png "Imagen del flujo de vue")** para la comunicación entre componentes.

* **Sigue un flujo doble-way data-binding** para la comunicación de modelos dentro de un componente aislado.

* **Tiene [soporte para TypeScript](https://vuejs.org/v2/guide/typescript.html "Documentación de Typecscript con vue")**. Cuenta con decoradores y tipos definidos de manera oficial y son descargados junto con la librería.

* **Tiene soporte para ES6**. Las herramientas y generadores vienen con Webpack o Browserify de serie por lo que tenemos en todo momento un Babel transpilando a ES5 si queremos escribir código ES6.

* **Tiene soporte a partir de Internet Explorer 9**. Según el proyecto en el que estemos esto puede ser una ventaja o no. Personalmente cuanto más alto el número de la versión de IE mejor porque menos pesará la librería, pero seguro que tendréis clientes que os pongan ciertas restricciones. Es mejor tenerlo en cuenta.

* **Permite [renderizar las vistas en servidor](https://vuejs.org/v2/guide/ssr.html "Documentación de renderizado en servidor de vue").** Los SPA y los sistemas de renderizado de componentes en JavaScript tienen el problema de que muchas veces son difíciles de utilizar por robots como los de Google, por lo tanto el SEO de nuestra Web o aplicación puede verse perjudicado. VueJS permite mecanismos para que los componentes puedan ser renderizados en tiempo de servidor.

* **Es extensible**. Vue [se puede extender mediante plugins.](https://vuejs.org/v2/guide/plugins.html "Documentación de creación de plugins en vue")

## ¿Cómo empezamos?

Para empezar, lo único que tendremos que hacer es incluir la dependencia de VueJS a nuestro proyecto. Dependiendo de nuestras necesidades, [podremos hacer esto de varias maneras](https://vuejs.org/v2/guide/installation.html "Diferentes instalaciones de vue"). Yo me voy a quedar con la forma habitual de añadir dependencias a un proyecto de NodeJS.

Lo primero que hacemos es ejecutar los siguientes comandos en el terminal:

```
$ mkdir example-vue
$ cd example-vue
$ npm init
```

Esto nos genera una nueva carpeta para el proyecto y nos inicia un paquete de NodeJS. Una vez que tengamos esto, añadiremos VueJS como dependencia de la siguiente manera:

```
$ npm install vue --save
```

De esta forma, ya tendremos todo lo necesario para trabajar en nuestro primer ejemplo. Lo que esta dependencia nos descarga son diferentes VueJS dependiendo del entorno que necesitemos.

Lo que hacemos ahora es añadir un fichero `index.html` en la raíz e incluimos tanto la librería de Vue, como nuestro fichero JS, donde desarrollaremos este primer ejemplo:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>VueJS Example</title>
</head>
<body>
    <script src="node_modules/vue/dist/vue.js"></script>
    <script src="app.js"></script>
</body>
</html>

```
 
Si os dais cuenta, hemos añadido la librería VueJS de desarrollo y no la minificada. Esto es así porque la librería de desarrollo nos lanzará un montón de advertencias y errores que nos ayudarán a aprender y trabajar con VueJS.

La librería tiene una gestión de errores súper descriptiva por lo que os recomiendo encarecidamente usarla en tiempo de desarrollo. Lógicamente tendremos que cambiarla por la minificada cuando nos encontremos en producción, pero bueno... para eso todavía queda. Una vez que tenemos esto, estamos preparados para trabajar en nuestro primer ejemplo.

Antes de que pasemos al código, me gustaría recomendaros un par de herramientas que nos pueden ser muy útiles para trabajar con VueJS.

### Depurando el código

Casi todos los frameworks importantes cuentan con una herramienta específica para poder depurar y analizar ciertos aspectos de nuestras aplicaciones. VueJS no iba a ser menos y [cuenta con un plugin para Firefox y Chrome](https://github.com/vuejs/vue-devtools) que se integra con el resto de herramientas de desarrollo de ambos navegadores.

He podido probar la herramienta de Chrome y me ha sorprendido lo fácil que es de usar. En la versión con la que he podido trastear, es posible inspeccionar los componentes que se encuentran en nuestro HTML. Esta inspección nos permite ver los modelos que se encuentran enlazados y la posibilidad de depurar esta información.

![Plugin de vue para el developers tools de chrome](/images/introduccion/screenshot.png)

Otro de los apartados está dedicado a la posibilidad de gestionar el estado de nuestra aplicación por medio de `vuex`. La funcionalidad nos permite ver las transiciones en las que se mueve nuestra aplicación como si de un vídeo que se pueda rebobinar se tratase.

La otra funcionalidad que incluye es la posibilidad de detectar todos los eventos que se están generando en nuestra aplicación VueJS. Como iremos viendo a lo largo de los posts, VueJS genera un importante número de eventos para comunicar los diferentes componentes de nuestra interfaz. Poder ver en tiempo real y por medio de notificaciones como se van generando, es una maravilla a la hora de saber lo que pasa.

El plugin es mantenido por la propia comunidad de VueJS, por lo que, por ahora, su actualización se da por descontado.

### Gestionando un proyecto real

La forma en la que hemos instalado la primera dependencia de VueJS es algo rudimentaria. Cuando empezamos un proyecto, suele ser buena idea empezar desde un código base, una plantilla que contenga aquella parte de flujo de trabajo, necesario para trabajar en una plataforma en concreto.

Como en el caso de Angular, la gente de VueJS nos proporciona una herramienta de terminal muy útil para nuestro día a día: [vue-cli](https://github.com/vuejs/vue-cli "Repositorio de vue-cli"). Esta herramienta nos permitirá generar plantillas de nuestro proyecto, generar componentes e incluso personalizar plantillas a nuestro gusto.

`vue-cli` no solo nos ayudará a empezar rápidamente en un proyecto, sino que además, nos ayudará a entender cómo estructurar nuestro código y a seguir una serie de buenas prácticas debemos tener en cuenta si queremos que nuestro proyecto escale.

Para instalarlo como dependencia global, solo tendremos que lanzar el siguiente comando:

```
$ npm install -g vue-cli
```

### Desarrollando más rápido

Los nuevos editores de texto permiten incluir pequeños plugins para añadir funcionalidad extra. En [Visual Studio Code](https://code.visualstudio.com/ "Web oficial de Visual Studio Code") contamos con unos cuantos plugins que nos pueden ayudar a desarrollar más rápido. Dos de los más usados son:

* **Syntax Highlight for VueJS**: plugin para remarcar todas aquella sintaxis y palabras reservadas a VueJS. Este plugin nos permite localizar elemento de una forma más rápida y cómoda.

* **Vue 2 Snippets**: plugin que contiene pequeños '[snippets](https://es.wikipedia.org/wiki/Snippet "Definición de snippet en wikipedia")' para que añadir nuestro código VueJS sea más rápido. De esta forma nos ayuda también como '[intellisense](https://es.wikipedia.org/wiki/IntelliSense "Definición de Intellisense en wikipedia")'.


##Primer ejemplo

Para mostrar un poco de código, vamos a crear un pequeño ejemplo. La idea es crear una pequeña aplicación que nos permita añadir nuevos juegos a mi listado de juegos favoritos - sí, lo siento, no deja de ser el `TODO LIST` de toda la vida :).

Para conseguir esto, lo primero que vamos a hacer es añadir un elemento HTML que haga de contenedor de nuestra aplicación VueJS:



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>VueJS Example</title>
</head>
<body>
    <div id="app"></div>

    <script src="node_modules/vue/dist/vue.js"></script>
    <script src="app.js"></script>
</body>
</html>

```

 
De esta manera, conseguimos delimitar el contexto en el que puede actuar nuestra aplicación. Es una buena forma de poder crear tantas aplicaciones VueJS necesitemos en un proyecto o incluso de poder alternar tecnologías.

Lo siguiente que hacemos es crear una instancia de nuestra aplicación VueJS en nuestro fichero `app.js`:

```javascript
const app = new Vue({
    el: '#app'
});

```
 
Lo que le decimos a VueJS es que genere una nueva instancia que tenga como referencia al elemento HTML que tenga como identificador único la palabra reservada `app`.

Lo siguiente que vamos a hacer es añadirle una pequeña plantilla con el HTML de nuestra aplicación:


```javascript
const app = new Vue({
    el: '#app',
    template: `
        <div class="view">                              
            <game-header></game-header>                  
            <game-add @new="addNewGame"></game-add>     
            <game-list v-bind:games="games"></game-list>
        </div>
    `
});

```
 
Lo que hacemos es configurar la plantilla que queremos que renderice VueJS. Bueno, parece que la plantilla tiene bastante magia y que poco tiene que ver con HTML. Tenéis razón hay muchas cosas que VueJS está haciendo aquí. Veamos qué ocurre:

* [línea 4]: Defino toda la plantilla dentro de un elemento div. Todo componente o instancia tiene que encontrarse dentro de un elemento raíz. VueJS nos devuelve un error de no ser así y renderizará mal el HTML.

* [línea 5]: Aquí he definido un componente. Con VueJS puede extender nuestro HTML con nueva semántica. En este caso un componente que va a hacer de cabecera.

* [línea 6]: Vuelvo a definir otro componente. En este caso, es un componente que cuenta con un `input` y un `button`. Lo veremos más tarde. De este elemento, lo más destacable es el atributo `@new="addNewGame`. Es un nuevo atributo que no se encuentra en el estándar de HTML ¿Qué es entonces? Estos elementos personalizados son lo que en VueJS se entiende como directivas. Son un marcado personalizado que aporta nueva funcionalidad al elemento HTML marcado. En este caso, lo que estamos haciendo es añadir un listener que escucha en el evento `new`, cada vez que el componente `game-add` emite un evento `new`, el elemento padre se encuentra escuchando y ejecuta la función `addNewGame`. No os preocupéis si no lo entendéis ahora porque lo explicaremos en un post dedicado a la comunicación entre componentes.

* [línea 7]: En esta línea hemos añadido otro componente. Este componente `game-list` se encarga de pintar por pantalla el listado de videojuegos favoritos. Como vemos, tiene una nueva directiva que no conocíamos de VueJS: `v-bind`. Esta directiva lo que hace es enlazar una propiedad interna de un componente con un modelo del elemento padre, en este caso el modelo games.

Vale, parece que la plantilla se puede llegar a entender.
 
Si nos damos cuenta hay dos elementos que hemos usado en la plantilla que no hemos definido en ninguna parte en nuestra primera instancia de VueJS: `addNewGame` y `games`. Para definirlos los hacemos de la siguiente manera:

```javascript
const app = new Vue({
    el: '#app',
    template: `
        <div class="view">
            <game-header></game-header>
            <game-add @new="addNewGame"></game-add>
            <game-list v-bind:games="games"></game-list>
        </div>
    `,
    data: {
        games: [
            { title: 'ME: Andromeda' },
            { title: 'Fifa 2017' },
            { title: 'League of Legend' }
        ]
    },
    methods: {
        addNewGame: function (game) {
            this.games.push(game);
        }
    }
});

```
 
Lo que hemos hecho es meter el método y el modelo en las zonas reservadas para ello. Todos los modelos que una instancia o un componente defina internamente, se tienen que incluir dentro de data y todos los métodos dentro de methods.

Teniendo esto, tenemos el 50% hecho de nuestra "aplicación". Lo siguiente que vamos a ver es la definición de los tres componentes visuales en los que he dividido la interfaz. Empecemos por el componente `game-header`:

```javascript
Vue.component('game-header', {
    template: '<h1>Video Games</h1>'
});

```
 
Nada del otro mundo. Lo único que estamos haciendo es registrar un componente de manera global con la etiqueta `game-header`. De esta forma ya podrá usar en las instancias de Vue. Internamente definimos un `template` sencillo con el título.

El siguiente componente tiene un poco más de chicha. Se trata del componente `game-add`, el combobox encargado de incluir nuevos juegos.

```javascript
Vue.component('game-add', {
    template: `
        <div>
            <input type="text" v-model="titleGame" />
            <button @click="emitNewGame">Añadir</button>
        </div> 
    `,
    data: function () {
        return {
            titleGame: null
        }
    },
    methods: {
        emitNewGame: function () {
            if (this.titleGame) {
                this.$emit('new', { title: this.titleGame });
                this.titleGame = null;
            }
        }
    },
});

```
 
Miremos un poco en detalle:

* **[línea 3]**: Volvemos a definir una plantilla HTML con un único elemento raíz.

* **[línea 4]**: El elemento tiene una directiva `v-model` que nos va a permitir ir obteniendo el valor del `input` e ir incluyéndolo en la variable `titleGame`.

* **[línea 5]**: El elemento tiene una directiva `@click` que lo que nos permite es registrar una función cuando se genere el evento clic sobre el botón.

* **[línea 8]**: El elemento data se inicializa, en un componente, con una función y no con un objeto. En su post veremos la razón de esto. Si ponemos un objeto, recibiremos un error de VueJS.

* **[línea 14]**: La función se encarga de ver si el input se encuentra vacío y emitir un evento hacia componentes padres con el nuevo título del juego.

Los siguientes componentes se encargan de pintar el listado de juegos. Son el componente `game-list` y el componente `game-item`:

```javascript
Vue.component('game-list', {
    props: ['games'],
    template: `
        <ol>
            <game-item v-for="item in games" :game="item" :key="item.id"></game-item>
        </ol>
    `
});

Vue.component('game-item', {
    props: ['game'],
    template: '<li>{{ game.title }}</li>'
});

```
 
El componente `game-list` recibe un modelo como propiedad. Se trata del listado de juegos a mostrar. En el `template` vemos la directiva `v-for` encargado de iterar los juegos e ir pintando diferentes componentes `game-item`.

El componente `game-item` recibe un modelo y lo pinta.
El sistema es reactivo, es decir que si yo inserto un nuevo elemento en el array de juegos, VueJS es lo suficientemente inteligente para saber que tiene que renderizar los elementos precisos.

En el siguiente ejemplo podemos ver todo junto:

```javascript
Vue.component('game-add', {
    template: `
        <div>
            <input type="text" v-model="titleGame" />
            <button @click="emitNewGame">Añadir</button>
        </div> 
    `,
    data: function () {
        return {
            titleGame: null
        }
    },
    methods: {
        emitNewGame: function () {
            if (this.titleGame) {
                this.$emit('new', { title: this.titleGame });
                this.titleGame = null;
            }
        }
    },
});

Vue.component('game-list', {
    props: ['games'],
    template: `
        <ol>
            <game-item v-for="item in games" :game="item" :key="item.id"></game-item>
        </ol>
    `
});

Vue.component('game-item', {
    props: ['game'],
    template: '<li>{{ game.title }}</li>'
});

Vue.component('game-header', {
    template: '<h1>Video Games</h1>'
});

const app = new Vue({
    el: '#app',
    template: `
        <div class="view">
            <game-header></game-header>
            <game-add @new="addNewGame"></game-add>
            <game-list v-bind:games="games"></game-list>
        </div>
    `,
    data: {
        message: 'Video Games',
        games: [
            { title: 'ME: Andromeda' },
            { title: 'Fifa 2017' },
            { title: 'League of Legend' }
        ]
    },
    methods: {
        addNewGame: function (game) {
            this.games.push(game);
        }
    }
});

```

## Conclusión

Nos queda mucho camino por recorrer, pero parece que la filosofía de VueJS tiene sentido. Hay un equipo de personas muy competentes del mundo JavaScript que han sabido extraer de las herramientas que han usado en el pasado, todas las características buenas y las han desarrollado aquí.

El ejemplo es simple pero si nos puede dar una idea de lo intuitivo y fácil que puede llegar a ser. Si vienes de utilizar frameworks y librerías orientadas a componentes no te costará el cambio. 

Si vienes de un mundo más artesano que hace uso de jQuery y/o Handlebars, el aprendizaje progresivo que propone y el sistema de plugins te pueden ayudar e incluso llegar a sonar muy parecido. Y... si eres nuevo en el mundo JavaScript... bienvenido, cualquier sitio es bueno para empezar.

Nos leemos :)
