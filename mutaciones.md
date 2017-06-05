# Capítulo 12. Las mutaciones y acciones

Con el post anterior, tenemos la mitad del flujo de vuex explicado. Ya somos capaces de obtener estados de nuestro store global y de crear reacciones en los componentes a partir de esto.

Lo siguiente que tenemos que tener en cuenta es cómo poder manipular estos estados para que se conviertan y muten en aquello que necesitamos, ya sea por la interacción del usuario, como por los eventos que se encuentran registrados en mi aplicación.

Para conseguir esto, vamos a explicar las dos funcionalidades que nos permiten esto y que, como veremos, tiene muchas similitudes a como lo hacen otras librerías como redux.

Leer más…

Las mutaciones

En vuex no puedo llegar a una variable del estado y manipularla para que cambie directamente. Si hiciese esto, los componentes no reaccionarían al cambio. Debido a que la librería quiere seguir un sistema de flujo unidireccional, donde todas las fases se encuentren en un ciclo cerrado, deberemos usar un nuevo concepto conocido como mutaciones. Las mutaciones son aquellas funciones que se encargan de cambiar el valor de nuestro estado.

Las mutaciones se comportan de igual manera que un evento. Una mutación cuenta con un tipo y un manejador que debe registrarse dentro de nuestro store. Cuando yo quiero hacer uso de esa mutación, solo tengo que invocarla.

Igual que un evento.

Para conseguir estas mutaciones, tenemos que registrarlas de la siguiente manera:

// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    mutations: {
        increment: function (state) {
            state.count++
        }
    }
};
Dentro de mutations insertamos todos aquellos manejadores que queremos que manipulen nuestros datos. increment es el tipo de mutación que queremos y la función el manejador que se disparará. Si nos damos cuenta, el estado se inyecta como parámetro para ser manipulado. Usa esta instancia para que todo funcione correctamente.

Una vez que tenemos esta mutación definida, ya podemos hacer uso de ella en nuestros componentes. Para usarlo, simplemente tendremos que ejecutar store.commit('increment'). En nuestro componente podríamos tener esto:

// ./components/componentA/componentA.js

export default {
     methods: {
         increment: {
              store.commit('increment');
         }
     }
};
Para evitar redundancias en el código, podemos hacer uso de mapMutations:

// ./components/componentA/componentA.js
import { mapMutations } from 'vuex';

export default {
     methods: mapMutations();
};
Al igual que pasaba con los getters, las mutaciones permiten payloads para manipular el estado. Por ejemplo:

// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    mutations: {
        increment: function (state, payload) {
            state.count += payload.amount;
        }
    }
};
Y para usarlo en un componente:

// ./components/componentA/componentA.js

export default {
     methods: {
         increment: {
              store.commit('increment', { amount: 200 });
         }
     }
};
Si no nos convence la firma del método commit por ser poco explicativa, podemos pasar un objeto indicando cada elemento de la siguiente forma:

// ./components/componentA/componentA.js

export default {
     methods: {
         increment: {
              store.commit({
                  type: 'increment', 
                  amount: 200
              });
         }
     }
};
Esto ya será al gusto del desarrollador o el equipo.

Para terminar con las mutaciones, vamos a aclarar un par de cosas para que el día de mañana no podamos cometer fallos innecesarios:

Cuidado con la reactividad de los objetos

El estado de un store se comporta igual que el data de un componente a nivel de 'reactividad'. Esto quiere decir, que inicies todas las variables de tu estado y que cuando vayas a mutar objetos del estado, ten en cuenta que las propiedades no provocarán reacciones al no contar con observadores ni getters internos en ellas.

Por tanto, si necesitas mutar un objeto, usa la funcionalidad Vue.set que te permitirá indicar que una propiedad ha cambiado y que el sistema debe reaccionar. En vez de hacer esto:

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    mutations: {
        changeName: function (state, payload) {
            state.user.name += payload.name;
        }
    }
};
Haz esto:

import Vue from 'vue';

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    mutations: {
        changeName: function (state, payload) {
            Vue.set(state.user, 'name', payload.name);
        }
    }
};
Si necesitas hacer un cambio de más propiedades, usa el Spread Operator de ES6:

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    mutations: {
        changeFullName: function (state, payload) {
            state.user = { ...state.user, ...payload.user };
        }
    }
};
Esto generará una nueva instancia del objeto con lo que la reacción sí está asegurada.

Poner los tipos de las mutaciones como constantes

Otro buen uso dentro de la creación de mutaciones es el llevar las cadenas a constantes. De esta forma cuando crece un proyecto, tengo localizadas todas las mutaciones que se pueden realizar y evito hardcodeados de cadenas, lo que hará que tenga intellisense en mi editor de código favorito. Por tanto, es una buena práctica hacer esto:

export const CHANGE_FULLNAME = 'CHANGE_FULLNAME';

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    mutations: {
        [CHANGE_FULLNAME](state, payload) {
            state.user = { ...state.user, ...payload.user };
        }
    }
};
De esta forma todo sigue igual. Si os dais cuenta, exporto la constante. Esto es porque ahora en los componentes también puedo hacer uso de estas constantes para realizar los commits. Si yo uso cadenas en mis componentes para referirme a mutaciones, y cambio el nombre a una de ellas, ningún editor podrá avisarme de que hay algo mal.

Sin embargo, si lo hago de esta manera, rápidamente me avisará de que algo no va bien en tiempo de escritura del código. Como digo, esto es solo una buena práctica y no es obligatorio hacer uso de ello.

No uses procesos asíncronos en tus mutaciones

Otro tema interesante es tener en cuenta es que las mutaciones deben contar con manejadores que solo gestionen lógica síncrona. Internamente, vuex incluye un proxy a cada mutación para 'logar' el estado antes y después de la mutación.

Estos hooks son usados por las herramientas de depuración para que podamos tener instantáneas en todo momento. Si incluimos operaciones asíncronas, perdemos esta visión porque las herramientas nos mentirán al no poder esperar que la mutación acabe.

Y entonces ¿Cómo gestionamos la asincronía en mis aplicaciones? ¿No voy a poder hacer llamadas a servidor o bases de datos en una aplicación de vue. Pues sí, para eso nacieron las acciones :)

Las acciones

Las acciones funcionan igual que las mutaciones. Eso sí, no mutan estado - eso lo delegan a las mutaciones - y se permiten todas las operaciones asíncronas que necesitemos. Por ejemplo, imaginemos que tenemos la mutación ADD_GISTS de esta forma:

export const ADD_GISTS = 'ADD_GISTS';

export default new Vuex.Store({
    state: {
        gists: []
    },
    mutations: {
        [ADD_GISTS](state, gists) {
            state.gists = gists;
        }
    }
};
Podríamos indicar una acción que obtenga los gists desde una API externa de la siguiente manera:

export const ADD_GISTS = 'ADD_GISTS';

export default new Vuex.Store({
    state: {
        gists: []
    },
    mutations: {
        [ADD_GISTS](state, gists) {
            state.gists = gists;
        }
    },
    actions: {
        fetchGists(context) {
             return axios.get('https//api.github.com/gists')
                 .then(response => response.data)
                 .then(gists => context.commit(ADD_GISTS, gists);
        }
};
Las acciones esperan una promesa para ser resueltas, de ahí que hagamos un return de la promesa que devuelve axios. Cuando axios nos devuelve los gists, podemos ejecutar commits con el tipo de mutación que queramos llevar a cabo. (El uso de constante tiene mucho sentido por todas las veces que vamos a usarlas durante nuestro proyecto).

Me gusta que separen ambos conceptos en la librería. Las acciones son donde  me puedo dedicar a meter mi lógica, mis validaciones, mi comunicación con el exterior, y las mutaciones sólo se preocupan de controlar los estados, de manipularlo. Me parece un proceso bastante natural y bien separado en diferentes conceptos y responsabilidades:

Las acciones se encargan de preparar todo lo necesario para que una mutación confirme un cambio en el estado como si de una transacción se tratase. Es como tener un sistema tricapa en un espacio muy reducido.

Para hacer uso de acciones en un componente, puedo hacerlo por medio del método dispatch. Dentro del componente haré esto:

export default {
    name: 'dashboard-view',
    created() {
        this.fetchGists();
    },
    methods: {
        fetchGists() {
            store.dispatch('fetchGists');
        }
    }
};
Lo mismo que con el resto, contamos con un mapActions que funciona de la misma manera:

import { mapActions } from 'vuex';

export default {
    name: 'dashboard-view',
    created() {
        this.fetchGists();
    },
    methods: mapActions()
};
Como pasaba con las mutaciones, las acciones también permiten un payload:

export default {
    name: 'dashboard-view',
    created() {
        this.fetchGists();
    },
    methods: {
        fetchGists() {
            store.dispatch('fetchGists', 1);
        }
    }
};
Detengámonos un poco en el primer parámetro que le pasamos a una acción, el parámetro context. Al final context es una estancia del propio store con lo que podremos hacer todo aquello que hacemos en un componente. Por ejemplo, yo podría acceder al estado dentro de una acción:

export const ADD_GISTS = 'ADD_GISTS';

export default new Vuex.Store({
    state: {
        gists: []
    },
    mutations: {
        [ADD_GISTS](state, gists) {
            state.gists = gists;
        }
    },
    actions: {
        fetchGists({ commit, state }, gistId) {
            if (state.gists.length === 0) {
                 return axios.get('https//api.github.com/gists')
                     .then(response => response.data)
                     .then(gists => context.commit(ADD_GISTS, gists);
            }
        }
};
Si dentro del estado ya hay gists, no realizo la llamada. Deshago el objeto de esa forma gracias a ES6; es la nueva funcionalidad llamada asignación por Destructuring.

Si esto es así, si tenemos una instancia del propio store en context, yo podría componer una acción determinada, a partir de otras acciones más específicas. Podría realizar varias llamadas asíncronas evitando el temido Callback Hell. Si lo unimos a los Async function de ES6, podemos tener algo parecido a esto:

actions: { 
    async actionA ({ commit }) { 
        commit('gotData', await getData()) 
    }, 
    async actionB ({ dispatch, commit }) { 
        await dispatch('actionA') 
        commit('gotOtherData', await getOtherData()) 
    } 
}
Las posibilidades nos las impondrá negocio, pero con esto estamos más que cubiertos. Tenemos todos lo métodos en un sitio centralizado, cohesionado a sus datos y con muchas posibilidades de aislarlos para ser probados.

Conclusión

Quizá, al igual que me pasó a mí, os sintáis ahora mismo así:

cabraloca.jpg

Es curioso que vuex tenga unos conceptos muy concretos y predefinidos que estudiados por separado son entendibles y coherentes, pero que cuando juntamos todo lo aprendido es un... vale, estoy perdido...

Creo que es normal. Hay mucho concepto de ES6 que se usa para sacarle todo el jugo al lenguaje y que hay que tener muy claros. Además, que usar un patrón más abstracto de lo que hemos visto en otros frameworks, nos va a suponer un esfuerzo. Incluso, aunque todo se conecta, es lógico que veamos mucha magia en sus piezas ya que delegamos mucho trabajo en la herramienta.

Creo que la única forma de enfrentarse a una librería como vuex es practicando mucho y haciendo casos de uso que se vayan complicando. Cometeremos errores, pero poco a poco, iremos viendo que todo tiene sentido y que nuestro código tiene cierta 'armonía'.

Muchos pensaréis: ¿Y tanto lío para qué? Yo con mi jQuery era feliz ¿Por qué todo esta complicación? Y tendréis razón. Si no sufres con jQuery, si no tardas años en encontrar un bug, si todo te escala, si tu equipo trabaja bien ¿Para qué cambiar?

El problema viene cuando esto no es así, cuando escalar tus proyectos y equipos está siendo difícil, cuando el ego de cada desarrollador ha hecho que aquello no tenga un estilo predefinido.

vuex te ayuda a eso, a evitar luchas entre tu equipo por cómo se estructuran o nombras las cosas, a no tener que pensar en eso. ¿Te gusta su estilo? ¿Lo entiendes? ¿Merece la pena el tiempo invertido? Pues adelante, ni lo dudes, pero ten en cuenta que se necesita un proceso de adaptación que tendrá un coste.

Nos leemos :)