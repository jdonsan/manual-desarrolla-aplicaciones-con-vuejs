# Capítulo 11. Los estados y getters

Una vez que hemos visto cómo incluir vuex en nuestra aplicación, es momento de explicar los conceptos básicos de la librería.

En el posts de hoy, dedicaremos tiempo a contar cómo podemos almacenar el estado dentro del store. Veremos cómo con muy poco de código podremos crear datos accesibles para varios componentes.

También repasaremos un concepto bastante interesante denominado getters que nos permitirán crear consultas más específicas sobre los datos que nos interesan para cada componente.

## Los estados

Los estados nos van a permitir crear diferentes instantáneas del estado completo de nuestra aplicación. La forma en la que definimos estados dentro de vuex es muy sencilla, solo tenemos que incluir atributos en el objeto state de vuex.Store de la siguiente forma:

```javascript
// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0
    }
});
```

Con esta simple declaración, ya podemos hacer uso de ello dentro de nuestros componentes:

```javascript
import store from '../store/index';

const componentA = {
    data() {
        return {
            count: null
        }
    },
    create() {
        this.count = store.state.count;
    }
};
```

Lo que hemos hecho es iniciar el estado interno count del componente con el estado global del store. Este código lo podemos mejorar ya que se encuentra algo acoplado. Si recordáis, en el post anterior,  inyectamos nuestro store para propagarse por todo el árbol de componentes de esta manera:

```javascript
import Vue from 'vue';
import App from './components/app/app.vue';
import store from './store';

const app = new Vue({
    el: '#app',
    store,
    template: '',
    components: { App }
});
```

Por lo tanto, podríamos hacer esto perfectamente:

```javascript
const componentA = {
    data() {
        return {
            count: null
        }
    },
    create() {
        this.count = this.$store.state.count;
    }
};
```

Con esto quitamos la dependencia 'hardcodeada' y usamos la instancia de store que se encuentra inyectada (`this.$store`). De esta manera, quitamos dependencias tediosas en todos nuestros componentes. Ya se cuenta con él.

El ejemplo sigue teniendo algo que huela mal ya que lo que hemos hecho es iniciar el estado interno, pero no hemos creado un componente que reaccione si cambia el estado del store. Para conseguir esto, nos basamos en la funcionalidad de estados computados con el que cuentan todos los componentes.

El ejemplo se convertiría en algo cómo esto:

```javascript
const componentA = {
    computed: {
        count() {
            return this.$store.state.count;
        }
    }
};
```

De esta forma, conseguimos que si el valor de store.state.count cambie, se ejecute esta función y el componente reaccione a cambios. Vale, parece que lo tenemos listo. Sigamos.

¿Qué ocurre si quiero incluir muchos estados en un componente? Insertar propiedades computadas de esta manera es algo tedioso. Estamos creando una función para llamar a un estado que se llama igual. Es código innecesario.

Hay una funcionalidad muy chula en vuex que nos permite mapear estados a cómputos de una forma más agradable. Dentro de vuex se cuenta con el método `mapState` que permite esto. Podemos hacer esto, por ejemplo:

```javascript
// ./components/componentA/componentA.js
import { mapState } from 'vuex';

export default {
    computed: mapState()
};
```

Este cambio no nos aporta más que azúcar sintáctico, pues el código de antes y el de ahora se comportan exactamente igual. Lo único que hacemos con esta forma es que el componente se ponga a escuchar todos los estados que existan dentro de nuestro store.

Esto está muy bien, pero es poco real que un componente se ponga a computar todo. Por tanto, ¿cómo con `mapState` puedo seleccionar qué parte del estado me interesa? Tenemos varias formas que según el contexto nos podrán interesar más o menos.

Por ejemplo, si tenemos este store:

```javascript
// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    }
});
```

Puede que en mi componente solo me interese `count` y `gists`. Esto lo puedo hacer de esta manera:

```javascript
// ./components/componentA/componentA.js
import { mapState } from 'vuex';

export default {
     computed: mapState(['count', 'gists'])
};
```

De esta manera, solo me mapearía esas dos propiedades y no user. Puede que por cierto contexto, el nombre del estado del store y del estado interno que se computa, no tengan que llamarse igual. Podemos mapearlo también, indicando un objeto de esta manera:

```javascript
// ./components/componentA/componentA.js
import { mapState } from 'vuex';

export default {
    computed: mapState({
        countAlias: 'count'
    })
};
```

De esta forma, dentro del template, haríamos uso de `{{ countAlias }}` desacoplando ambas.

Puede que necesitemos calcular un dato a partir de un estado del store. En ese caso yo puedo indicar una función:

```javascript
// ./components/componentA/componentA.js
import { mapState } from 'vuex';

export default {
    computed: mapState({
        countPlus2(state) {
            return state.count + 2;
        }
    })
};
```

Otra opción es que, no solo necesitemos mapear estados del store, sino que el propio componente tenga estados computados internos que no tengan nada que ver con el global. Podemos tener esto:

```javascript
// ./components/componentA/componentA.js
import { mapState } from 'vuex';

export default {
     computed: {
         myLocalComputed() {
             // code
         }, 
         ...mapState(['count', 'gists'])
     }
};
```

De esta forma, y gracias al nuevo Spread Operator de ES6, podemos combinar objetos diferentes de una manera muy sencilla.  Ahora contaría con tres propiedades computadas: las dos del store y la interna (`myLocalComputed`).

> Estas utilidades con prefijo `map-` serán creadas para todos los elementos de vuex por lo que aprendérselo bien,  será necesario ya que lo usaremos en lo que queda de serie.

## Los getters

Una cosa es cómo almaceno los datos en mi store global y otra diferente qué datos me son útiles dentro de un componente. Por ejemplo, puede que dentro de mí store se almacene un número considerable de gists, pero que un componente solo necesite aquellos que son públicos.

Bueno, no parece muy grave, a fin de cuentas, si tengo acceso al store en un componente, podría hacer algo como esto:

```javascript
// ./components/componentA/componentA.js

export default {
     computed: {
         publicGists() {
             return this.$store.state.gists.filter(gist => gist.public);
         }
     }
};
```

Nada mal ¿no? Ahora, el 'pequeño' problema viene cuando otro componente necesita también conocer solo los gists públicos. Para conseguir esto, podría coger la función anterior, llevármela a una librería y reutilizar la utilidad en ambos componentes. Sin embargo, estamos creando código que se encuentra muy acoplado a vuex y que nos va a hacer incluir dependencias en los componentes.

vuex ha pensado en ello y nos ha dado una funcionalidad dentro del store llamada getters. Los getters son funciones que permiten obtener datos parciales de nuestro estado y que son reutilizables por todas los componentes de una forma más cómoda que creando librerías e incluyendo dependencias.

Refactoricemos el componente anterior, llevémonoslo a nuestro store de esta manera:

```javascript
// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    getters: {
        publicGists(state) {
            return state.gists.filter(gist => gist.public);
        }
    }
});
```

Lo que hago es definir consultas dentro del objeto `getters` de mi store. Ahora en mi componente puedo hacer esto:

```javascript
// ./components/componentA/componentA.js

export default {
     computed: {
         publicGists() {
             return this.$store.getters.publicGists;
         }
     }
};
```

Estamos reutilizando consultas con solo llamar a las propiedades que tengamos almacenadas en getters. Con vuex ,  puedo componer getters de esta manera:

```javascript
// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    getters: {
        publicGists(state) {
            return state.gists.filter(gist => gist.public);
        },
        totalPublicGists: (state, getters) => {
            return getters.publicGists.length;
        }
    }
});
```

Los getters son inyectados internamente. De esta manera, puedes usarlo getters dentro de otros getters.

Incluso hay que pensar que esto no se limita a búsquedas estáticas. Puedo indicar desde el componente que elemento me puede interesar de una colección;

```javascript
// store/index.js
...

export default new Vuex.Store({
    state: {
        count: 0,
        gists: [],
        user: {}
    },
    getters: {
        publicGists(state) {
            return state.gists.filter(gist => gist.public);
        },
        totalPublicGists: (state, getters) => {
            return getters.publicGists.length;
        },
        publicGistById: (state, getters) => (id) => {
            return getters.publicGists.filter(gist => gist.id === id);
        }
    }
});
```

Para usar este getter en un componente, lo haría así:

```javascript
// ./components/componentA/componentA.js

export default {
     computed: {
         publicGists() {
             return this.$store.getters.publicGistById(2);
         }
     }
};
```

También contamos con una utilidad de maps que se comportan igual que el de `mapState`. En este caso solo tenemos que usar `mapGetters`:

```javascript
// ./components/componentA/componentA.js
import { mapGetters } from 'vuex';

export default {
     computed: mapGetters()
};
```

De esta forma, incluyo todos los getters que existan configurados en el store.

Me gustan los getters porque nos desacoplan mucho (es bueno para testear todo este código de consultas de forma aislada) y nos permiten reutilizar código entre componentes.

## Conclusión

Usemos la conclusión para hacer una pequeña reflexión:

Contar con vuex no significa tener que almacenar exactamente todos los estados de mi aplicación en el store. Se trata de tener claros qué datos pueden ser utilizados por más partes del sistema, qué estados son claves para el módulo o el negocio de la aplicación y encapsularlos en este mecanismo.

Pero esto no quiere decir que, en muchos casos, los componentes no cuenten con su propio estado interno. Habrá estados que solo incumban al correcto funcionamiento interno del componente ¿Qué sentido tendría sacar este estado a un store? ¿Cuándo tiene sentido que se encuentre encapsulado en la pieza de código que más cohesionado se encuentra de él?

Tener esto claro puede suponernos, como desarrolladores, un tiempo de análisis y de diseño clave para que nuestro estado global no se convierta en un cajón desastre de variables. Tengámoslo en cuenta.

Nos leemos :)