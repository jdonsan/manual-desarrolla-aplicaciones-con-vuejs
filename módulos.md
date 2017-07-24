# Capítulo 13. Los módulos

Al igual que pasa con nuestra aplicación, cuando un store empieza a crecer demasiado, empieza a ser bastante inmanejable gestionarlo todo en un único fichero. Como vuex presenta una solución donde se gestiona todo el estado en único objeto, tenemos que pensar una forma para poder modularizar, pero a la vez seguir teniendo esta estructura en árbol único.

vuex cuenta con una funcionalidad que nos va a permitir dividir nuestro árbol de datos en módulos más específicos que contarán cada uno de ellos con todo lo necesario para gestionar estas porciones.

La forma de crear un módulo es tan fácil como crear un objeto JSON de la siguiente manera:

```javascript
const moduleA = {
    state: {...},
    getters: {...},
    mutations: {...},
    actions: {...}
};
```

Para incluirlo dentro de nuestro store, usamos la propiedad module y lo asignamos con el nombre que deseemos que tenga:

```javascript
const store = new Vuex.Store({
    state: {...},
    getter: {...},
    mutations: {...},
    actions: {...},
    modules: {
        a: moduleA
    }
});
```

Con esto, ahora puedo acceder al estado de un módulo, en particular, de esta manera:

```javascript
store.state.a;
```

## Estado local de los módulos

Al contar con una jerarquía de módulos y submódulos, es importante saber cómo acceder a las diferentes partes del estado según en qué zona del store nos encontremos.

Por ejemplo, si me encuentro en un módulo en particular, lo que se inyecta tanto en los getters como en los mutations es el estado local al módulo, de tal manera que yo lo haría así:

```javascript
const moduleA = { 
    state: { count: 0 }, 
    mutations: { 
        increment(state) { 
            state state.count++ 
        } 
    }, 
    getters: { 
        doubleCount (state) { 
            return state.count * 2 
        } 
    } 
};
```

Si necesitase acceder al estado del ámbito global, es decir, del estado raiz, podría hacerlo tanto en los getters como en los actions, pues tengo acceso a esta parte del store. En los getters se inyecta un tercer parámetro con este estado y en los actions el objeto context cuenta también con ello. Veamos el ejemplo:

```javascript
const moduleA = {
    getters: {
        sumWithRootCount (state, getters, rootState) {
            return state.count + rootState.count;
        }
    },
    actions: { 
        incrementIfOddOnRootSum ({ state, commit, rootState }) { 
             if ((state.count + rootState.count) % 2 === 1) { 
                 commit('increment');
             } 
        } 
    } 
};
```

## Módulos y su namespace

Aunque definas módulos para tener un buen sistema modularizado, puede que no necesites que tus componentes tengan conocimiento de esta modularización, por lo tanto, todos los getters, mutations y actions son incluidos en el ámbito global del store. Si dos acciones o mutaciones son llamadas igual, ambas reaccionan.

El estado sí es dividido a nivel de espacio de nombres para que no haya conflictos a la hora de que sus variables se llamen igual en diferentes módulos.

Si por un casual, deseamos que nuestros getters, mutations y actions se encuentren separados por espacio de nombres, tendremos que poner a true el atributo namespaced en la raíz del módulo. De esta manera, si hacemos esto:

```javascript
const moduleA = { 
    namespaced: true,
    state: { count: 0 }, 
    mutations: { 
        increment(state) { 
            state state.count++ 
        } 
    }, 
    getters: { 
        doubleCount (state) { 
            return state.count * 2 
        } 
    } 
};
```

La forma en la que yo accedería ahora ese getter y esa mutation sería de la siguiente manera:

```javascript
this.$store.getters['a/doubleCount'];
this.$store.commit('a/increment');
```

Si vas a usar este sistema de espacio de nombres, ten en cuenta usar esta nomenclatura en formato cadena como si accedieras a un fichero de una carpeta también para los `mapGetters`, `mapActions` y `mapMutations`, ya que se usa el mismo.

## Acceso a elementos globales en los módulos

Puede ocurrir que cuando nuestros módulos se encuentren dentro de un espacio de nombres, queramos acceder a un getter de su módulo superior. Para hacer eso,  los getters cuentan con cuarto parámetro donde son inyectados estos rootGetters

```javascript
modules: { 
    foo: { 
        namespaced: true, 
        getters: {
            someGetter (state, getters, rootState, rootGetters) { 
                getters.someOtherGetter;  
                rootGetters.someOtherGetter;  
            }, 
            someOtherGetter: state => { ... } 
        },
    }
}
```

Como vemos, no hay un conflicto de nombres aunque el módulo raíz y el submódulo compartan un método con el mismo nombre porque ambos son accedidos de diferente manera.

Nos ocurre lo mismo con los actions ya que el contexto también cuenta con esta propiedad `rootGetters`.

```javascript
actions: { 
    someAction ({ dispatch, commit, getters, rootGetters }) { 
        getters.someGetter;
        rootGetters.someGetter;
    },
    someOtherAction (ctx, payload) { ... } 
}
```

Si dentro de esta acción yo quiero llamar a otras acciones o mutaciones, lo que tengo que hacer es pasar un objeto `{ root: true }` como tercer parámetro de la siguiente manera:

```javascript
actions: { 
    someAction ({ dispatch, commit, getters, rootGetters }) { 
         // -> 'foo/someOtherAction' 
         dispatch('someOtherAction'); 

         // -> 'someOtherAction' 
         dispatch('someOtherAction', null, { root: true }); 

         // -> 'foo/someMutation' 
         commit('someMutation'); 

         // -> 'someMutation'
         commit('someMutation', null, { root: true }); 
    },
    someOtherAction (ctx, payload) { ... } 
}
```

Se pasa como tercer parámetro porque el segundo puede ser el payload.

## Registro de módulos dinámicamente

Puede sernos útil registrar diferentes módulos en otros momentos de la aplicación, sin tener que definirlos todos en el store en un instante determinado. Para lograr esto hacemos uso del método registerModule de la siguiente manera:

```javascript
store.registerModule('myModule', { // ... });
```

El primer parámetro indica el nombre que va a tener el módulo y el segundo el módulo en sí. Si necesitamos crear un submódulo dentro de un módulo concreto, podemos registrar estos anidamientos de la siguiente manera:

```javascript
store.registerModule(['nested', 'myModule'], { // ... });
```

Donde el acceso interno sería `this.$store.nested.myModule.state`.

Esta forma de registrar módulos dinámicamente, nos viene muy bien para que plugins de vue puedan hacer uso de la potencia de vuex para que puedan enganchar su módulo de datos a nuestra estructura en árbol del estado.

## Reutilizando módulos

Puede que necesites reutilizar módulos. Sí, tener más de una instancia del mismo módulo en tu store. Si se da este caso, ten cuenta que el estado es un objeto y que si lo reutilizas de esta manera, compartes su referencia, provocando, que si cambias el valor en una de ellas, cambie el valor en todas sus instancias.

Para solucionar esto, hacemos como lo mismo que hacíamos con el objeto data de los componentes: asignar una función que devuelva nuestro objeto. De esta forma cada instancia contará con una referencia diferente del estado (Una pequeña factoría dentro de vuex, fijate tú :) ). Lo hacemos así:

```javascript
const MyReusableModule = { 
    state () { 
        return { foo: 'bar' } 
    }, 
    // mutations, actions, getters... 
};
```

## Conclusión

Como vemos, tener un árbol de datos complica entender el todo. Tener un cajón desastre de datos no nos va aportar más que quebraderos de cabeza, y saber cuándo crear buenos estancos de datos nos ayudará mucho en el momento que crezca el proyecto. Piensa en tu store como si de un gran buque se tratase, piensa que tienes en tus manos un petrolero.

Estos barcos son tan largos que con el movimiento interno que produce el fuel se podrían llegar a desestabilizar y hundir. Es por eso que existen estancos que dividen estos compartimientos para evitar esto y que el fuel sea transportado. Piensa en tu gran store como si fuera este caso.

Como en todo, el arte del desarrollo está en diseñar y nombrar adecuadamente las cosas. De nada nos sirve contar con esta funcionalidad de modularización sino hemos dedicado un tiempo a comprender el negocio y las formas en que tenemos que moldearlo.

Obsesionarnos con modular en las primeras fases de desarrollo nos va a servir de poco y la refactorización y el tiempo en un proyecto, nos irá diciendo que necesidades pueden surgir. Encontrar ese equilibrio entre no dejarnos llevar por la deuda técnica y empezar a hacer sobre-ingeniería desde el principio es la gran responsabilidad que tendrá todo desarrollador...


...Y con esto acabamos, por ahora, con vuex. 

Lo que vamos a hacer en los próximos posts del blog es dejar, por unas semanas, el ecosistema de vue a un lado y centrarnos en algo que creo primordial para poder trabajar cómodamente en la creación de aplicaciones SPA: el aprendizaje de webpack.

Hemos postergado esto y en algunas etapas nos hemos fiado de la magia que desentrañaba esta herramienta y creo que es importante que nos sintamos cómodos con ella, por eso, haremos una pequeña serie que nos ayudará a comprender mejor nuestro stack de trabajo.

Nos leemos :)