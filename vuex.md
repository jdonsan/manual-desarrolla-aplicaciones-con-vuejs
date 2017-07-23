# Capítulo 10. Introducción

Cuando decidimos apostar por una arquitectura de componentes, uno de los primeros problemas con los que nos enfrentamos cuando nuestra aplicación empieza a crecer es la dificultad que solemos tener para comunicar componentes que se encuentran a distintos niveles de nuestro árbol.

Una vez que hemos tratado el problema de diseñar buenos componentes, que hemos hablado de cómo dividir nuestra aplicación en diferentes vistas de las que podemos navegar sin problema, llega el turno de cómo gestionar el estado interno de una aplicación de página única.

A lo largo de los próximos posts, hablaremos de cómo atajar este problema dentro del ecosistema de vue. Seguimos dando un paso más en nuestro camino progresivo hacia la construcción de aplicaciones del mundo real con VueJS. Continuemos ;):

## Antes de empezar...

### ¿Qué es el estado de una aplicación?

Cuando hablamos de estados en una aplicación, hablamos del conjunto completo de variables y constantes que configuran nuestra aplicación. Los estados son todas las formas posibles en las que se puede encontrar mi sistema en un momento determinado.

La forma en que gestionemos y estructuremos el estado de nuestra aplicación puede ser la clave para evitar bugs innecesarios. Saber cómo, cuándo, dónde y por qué muta un estado en particular y de la manera más rápida posible, nos ayudará en nuestro día a día.

### ¿Qué son las acciones de una aplicación?

Son todos aquellos métodos, funciones, o procedimientos que se encargan de mutar los estados de nuestra aplicación. Se encargan de que dado x este pueda crear un nuevo estado y. La definición matemática de una función es este:

```
f(x) = y
```

Dado un estado x que es pasado a una función f es obtenido el nuevo estado y. Los paradigmas de programación se articulan como propuestas para gestionar y mutar los estados por medios de diferentes tipos de cómputos.

### ¿Cómo puedo comunicarme entre componentes?

Al trabajar con arquitecturas de componentes jerarquizadas en forma de árbol, uno de nuestros trabajos consiste en conseguir comunicar estados de unos componentes a otros. Cuando empezamos a desarrollar nuestro árbol, nuestras arquitecturas son sencillas y el comunicar componentes padres con componentes hijos es relativamente fácil.

El problema viene cuando nuestro árbol empieza a crecer y tenemos que comunicar componentes hermanos - componentes que comparten el mismo componente que los instanció - o componentes que no tienen ningún parentesco dentro de nuestro árbol.

Cuando ocurre esto, tenemos que empezar a pensar en alguna estrategia que nos sea útil y fácilmente de mantener. Una de estas estrategias nos la proporciona la propia librería de vue. En vue podemos comunicar diferentes componentes creando un bus de datos interno.

Este bus de datos se consigue creando una nueva instancia de la clase Vue. Yo por ejemplo, podría hacer esto para comunicar dos componentes sin parentesco:

 
```javascript
const bus = new Vue();

const componentA = {
  methods: {
      doAction() {
        bus.$emit('increment', 1);
      }
  }
};

const componentB = {
  data() {
    return {
      count: 0
    }
  },
  created() {
    bus.$on('increment', (num) => {
      this.count += num;
    });
  }
};
```
 
Lo que hago es crear una instancia de Vue que cuenta con un método $emit para emitir eventos y un método $on para registrarme a eventos. Con esto, lo que hago es, en el componente B, registrarme al evento increment cuando el componente ya ha sido creado y esperar a que el componente A emita nuevos cambios al ejecutar su método doAction. Es muy parecido a la comunicación que tiene un hijo con un padre, pero esta vez sin parentesco.

Este sistema nos puede funcionar puntualmente para aplicaciones pequeñas o en casos aislados. Cuando el sistema empieza a crecer, empieza a hacerse inmantenible. Nos dificulta las labores de reutilizar acciones y de compartir estados comunes. 

Por tanto, tenemos que buscar otras alternativas.

Podría ser buena idea, para aplicaciones medias, hacer uso de un lugar centralizado donde compartir estos estados y métodos. Una pequeña librería que usen los componentes. Podríamos pensar en algo como esto:

```javascript
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    this.debug && console.log('setMessageAction triggered with', newValue)
    this.state.message = newValue
  },
  clearMessageAction () {
    this.debug && console.log('clearMessageAction triggered')
    this.state.message = ''
  }
};

var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
});

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
});
```
 
Lo que hacemos es crear un objeto que contiene el estado a compartir y unos métodos que se encargan de mutar este estado. De esta manera centralizamos los estados y las acciones.

Ahora, podemos crear instancias de componentes que compartan parte del estado. Si uno de los componentes quieren mutar un estado compartido, hacen uso de uno de los métodos del store. Cómo el objeto se encuentra referenciado en todos los componentes que deseamos, el cambio se realiza en todos.

Con esto, conseguiríamos una arquitectura muy parecida a la del siguiente dibujo:

![Flujo de un store casero](/images/vuex/state.png)

La solución no me convence del todo, porque no dejamos de tener un estado global con vía libre para realizar cambios a cualquier componente. No existe ningún control y puede ser difícil para trazar qué componente es el responsable en el cambio de un estado. No hay encapsulamiento, y si no somos cuidadosos, podemos liarla parda. Entre tener esto y nada, lo mismo.

Como decimos, este sistema nos puede funcionar, pero cuando contamos con aplicaciones más grandes aún, donde modularizar también este store será necesario, necesitaremos una librería más elaborada, más robusta y que no de tantas posibilidades para manipular externamente el estado. Es aquí donde entra en juego alternativas como vuex.

## ¿Qué es vuex?

Cuando el ejemplo anterior se nos queda demasiado corto, es hora de incluir a nuestra aplicación de vue una alternativa llamada vuex. Vuex es la implementación que ha hecho la comunidad del patrón de diseño creado por Facebook llamado flux.

Cuando llega ese momento en el que tienes que compartir demasiado estados comunes entre componentes, que tienes que hacer virguerías para comunicarte en tu propia comunidad, que tienen métodos o acciones muy parecidas en muchos componentes que te gustaría refactorizar o que empiezas a tener problemas para seguir la trazabilidad por la que pasa un estado en particular es momento de plantearse usar este tipo de arquitecturas.

Flux es una arquitectura que gestiona el estado en un objeto singleton global donde su labor es crear mecanismos para evitar que otros componentes puedan cambiar el estado de una aplicación sin su control.

Dentro de la comunidad se han desarrollado muchas implementaciones de flux, pero una de las más conocidas es redux. Redux es una librería, de estilo funcional, muy utilizada por ser agnóstica al framework.

Esto quiere decir que la librería puede ser utilizada tanto con Angular como con React sin sufrir fricciones con los diferentes planteamientos ya que se encarga de la gestión de forma que no se acopla con ninguna plataforma. Siempre necesitaremos conectores específicos para usarlo con cada una de ellas. [En vue también se cuenta con un conector para redux, por si los desarrolladores desean hacer uso de él.](https://github.com/revue/revue)

Sin embargo, como decíamos, en vue se ha optado por desarrollar una implementación específica del patrón llamada vuex y que se acopla mucho mejor a la filosofía de vue como iremos viendo a lo largo de estos posts.

La arquitectura de vuex está muy bien esquematizada en esta imagen:

![](/images/vuex/vuex1.png)

Aunque entraremos en detalle más adelante (no hoy, tranquilos :smile:) expliquemos cada elemento:

* El cuadro verde representa nuestra arquitectura de componentes, los cuales se presentan ahora como la estructura de un edificio esperando a estar habitado por los estados de la aplicación.

* El círculo morado son estos estados que los componentes usan. No se encuentran internamente dentro de los componentes, sino que ahora están gestionados por vuex y vinculados a los componentes por medio de observadores. Vuex se adecua muy bien al sistema reactivo de la plataforma y lo que hace es, que cuando un estado dentro de su sistema de almacenamiento muta, y se encuentra vinculado con un componente, se provoca la reacción de renderizar de nuevo el componente con el nuevo estado mutado.

* Por otra parte, los componentes son capaces de lanzar acciones. Las acciones son el círculo amarillo y son un buen sitio donde gestionar parte de la lógica más próxima a los datos de nuestra aplicación. Permiten gestionar asincronía, por lo que son el lugar idóneo para realizar llamadas a servidores externos (caja gris).
Cuando una acción ha terminado de realizar sus labores asíncronas (o síncronas), permite realizar confirmaciones (commits) contra el estado.

* Estas confirmaciones lo que provocan son ejecutar métodos especializados en la mutación de cambios. Esto se puede ver en el círculo rojo. Cuando se ejecutan estos métodos de mutación se desencadenan cambios en el estado que provocan renderizados en el HTML. De esta manera, cerramos el círculo.

Si nos fijamos, nos encontramos en un flujo unidireccional, lo que nos ayuda a entender en todo momento qué es lo que está ocurriendo en nuestro sistema.
Por ahora dejemos esto aquí, en el marco teórico, porque lo explicaremos con ejemplos más adelante. Ahora veamos como integrar la librería de vuex en nuestro SPA.

## ¿Cómo empezamos con vuex?

Bastante simple. Como todo en el mundo NodeJS, lo primero que hacemos es instalar e incluir la dependencia en nuestra aplicación de la siguiente manera:

```
$ npm install vuex --save
```

Como `vue-cli` no nos da soporte para vuex en su generador, lo siguiente será incluir la carpeta donde almacenaremos todo lo necesario para vuex. Lo hacemos de esta manera:

![carpeta del store en el proyecto](/images/vuex/captura-de-pantalla-de-2017-05-24-12-31-06.png)

Dentro de este `index.js` crearemos todo nuestro almacenamiento de estados. Lo primero que escribimos es lo siguiente:

```javascript
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({});
```

Donde, como siempre en vue, estamos añadiendo vuex como un plugin, como ya hacíamos con vue-router.

Lo último que hacemos para acabar con la conexión total entre vue y vuex, es incluir la instancia creada del store en la instancia de nuestra aplicación, de la siguiente manera:

```javascript
import Vue from 'vue';
import App from './components/app/app.vue';
import store from './store';


const app = new Vue({
    el: '#app',
    store,
    template: '<App/>',
    components: { App }
});
```

Ya está, ya podemos empezar con el resto de conceptos.

## Recuerda

Como ya explicamos en la introducción, vue cuenta con una buena herramienta de depuración en Chrome y que dentro de ella cuentas con una pestaña de vuex donde podrás inspeccionar en todo momento la instantánea de tu proyecto. Úsala todo lo que puedas para verificar que estás haciendo las cosas como quieres.

## Conclusión

Cuidado con vuex. Muchas veces nos dejamos llevar por la tecnología e introducimos dependencias en nuestras aplicaciones sin saber muy bien si la necesitamos. Vuex puede ser un buen caso de esto. Nuestras aplicaciones evolucionan mucho a lo largo del tiempo e incluir una librería como esta desde el principio, puede entorpecernos más que ayudarnos.

Si una cosa tiene buena vue es que permite ir incluyendo sus diferentes piezas según necesidad, así que si no tenemos muy claro si vamos a necesitar un lugar centralizado para manejar el estado, quizá sea mejor que no lo usemos, más adelante podremos hacerlo.

Hay que tener en cuenta que vuex nos ayuda a mantener y testear mejor nuestra aplicación, pero también tenemos que ser conscientes que vuex nos implica nuevos niveles de abstracción que harán que necesitemos trabajar con nuevos conceptos que harán la curva de aprendizaje menos accesible para desarrolladores juniors a nuestros proyectos. Por ello, debemos tener cuidado.

De todas las implementaciones de flux, vuex me parece la más intuitiva y coherente con las necesidades del framework, pero eso no significa que no nos vaya a suponer añadir más fontanería de la que quizá nos gustaría.

Nos leemos :)
