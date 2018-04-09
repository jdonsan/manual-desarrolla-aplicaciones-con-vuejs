# Capítulo 23. Testeando nuestros stores

Hemos visto cómo configurar nuestro proyecto para que pueda ser testeado, hemos creado nuestro primeros test sobre componentes de VueJS y hemos jugado con diferentes tipos de componentes y complejidades, desde componentes que tenían dependencias hasta componentes que solo eran visuales.

Nos queda para terminar esta sería, ver cómo somos capaces de testear nuestros stores. Si recordamos, los stores son aquellas piezas de código que configuraban vuex  y que nos permitían gestionar el estado global de nuestra aplicación en pequeños estancos bien modularizados y abstraídos.

Tenemos que ver las aproximaciones que podemos llevar a cabo y cómo probar cada uno de los elementos. Vayamos al lío:

## ¿Qué aproximación seguir a la hora de probar stores?

Tenemos dos formas de probar un store:

* **Separando las diferentes partes de un store \(getters, mutations y actions\) y probarlos de manera independiente**: Lo bueno de vuex es que cada uno de su elementos son funciones normales que pueden ser testadas de manera separada. Si una función trabaja bien por separado, cuando sea incluida en otra pieza, seguirá teniendo ese buen comportamiento. Es la magia de la modularización. En este caso es más fácil ver qué parte del store ha fallado. Pueden venir muy bien como test unitarios al uso.
* **Juntando todos los elementos y hacerles trabajar como un todo dentro de un store real. Se usan los mecanismos que existen en vuex y se comprueba su comportamiento y el estado que han dejado**. En este caso se realizan los tests tal y como luego los componentes los van a usar. Pueden venir muy bien como test de integración.
  Ambas son válidas y pueden ayudarnos en diferentes contextos dependiendo de cómo nos sintamos mas cómodos.

A continuación veremos cómo probar los elementos por separado de un store:

## ¿Cómo pruebo un getter?

Tenemos el siguiente getter de un posible store:

```js
// getters.js 
export default { 
    evenOrOdd: state => state.count % 2 === 0 ? 'even' : 'odd' 
}
```

Si recordamos, un getter​ no era más que una posible consulta sobre un estado determinado de un store. En este caso, la consulta indica si el contador es impar o par.

Bueno, probar esto es fácil, tenemos una función  llamada evenOrOdd  que tiene como parámetro un objeto estado y que contiene un flujo interno. Por tanto, tendremos dos tests: uno para comprobar que nos devuelve even cuando hay un número par en el contador y odd cuando hay un número impar en el contador:

```js
// getters.spec.js 

import getters from './getters' 

describe('getters store'', () => 
    it('evenOrOdd returns even if state.count is even', () => { 
        const state = { count: 2 } 
        expect(getters.evenOrOdd(state)).toBe('even') 
    }) 

    it('evenOrOdd returns odd if state.count is even', () => { 
        const state = { count: 1 } 
        expect(getters.evenOrOdd(state)).toBe('odd') 
    })
})
```

Iniciamos el estado con el contador como deseamos y comprobados que el getter devuelve lo esperado.

## ¿Cómo mockeo un getter en un componente?

Ahora bien, si un componente hace uso de uno de nuestros getter, ¿cómo podemos mapearlo?  Tenemos el siguiente componente:

```js
<template>
  <div>
    <p v-if="inputValue">{{inputValue}}</p>
    <p v-if="clicks">{{clicks}}</p>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
export default{
  computed: mapGetters([
    'clicks',
    'inputValue'
  ])
}
</script>
```

Este componente muestra por pantalla los datos de los getters clicks y ​inputValue. Para hacer un tests de esto, lo que hago es mockear los getters. Esto lo consigo:

* Añadiendo vuex a nuestra instancia de VueJS local al test.
* Creando un mock de nuestro getters
* Incluyendo este store a nuestro componente a probar.

El test sería así:

```js
import { shallow, createLocalVue } from '@vue/test-utils' 
import Vuex from 'vuex' 
import Actions from '@/components/Getters' 

const localVue = createLocalVue() 
localVue.use(Vuex) 

describe('Getters.vue', () => { 
    let getters 
    let store 

    beforeEach(() => { 
        getters = { 
            clicks: () => 2, 
            inputValue: () => 'input' 
        } 
        store = new Vuex.Store({ getters }) 
    }) 

    it('Renders "state.inputValue" in first p tag', () => { 
        const wrapper = shallow(Actions, { store, localVue }) 
        const p = wrapper.find('p') 
        expect(p.text()).toBe(getters.inputValue()) 
    }) 

    it('Renders "state.clicks" in second p tag', () => { 
        const wrapper = shallow(Actions, { store, localVue }) 
        const p = wrapper.findAll('p').at(1) 
        expect(p.text()).toBe(getters.clicks().toString()) 
    }) 
})
```

El `beforeEach` nos ayuda a iniciar el store para cada tests. Tenemos que tener en cuenta que cada test tiene que ser determinista y aislado. Esto quiere decir que siempre debe darnos el mismo resultado dado el mismo estado de entrada y que si yo ordeno los tests de otra manera, todos siguen siendo correctos. Ningún test debe depender de otro test.

Luego usamos ese store, para que los métodos internos del componente trabajen correctamente. Desta manera hemos aislado el componente de vuex y podemos probar comportamientos internos.

Aunque lo explicaremos. este sistema de mockeo será igual para todas las partes de vuex.

## ¿Cómo pruebo una mutation?

El resto de elementos se testeara y se mockeara en un componente de manera muy parecida. Las mutations son funciones que permiten cambiar o mutar el estado de un store. Son funciones de ejecución síncrona por lo que se comportan parecido que los getters. Podemos ver a los getter como funciones para obtener el estado del store y los mutations como funciones para setear el estado del store.

Entonces, dada la siguiente mutation que permite incrementar un contador:

```js
// mutations.js 
export default { 
    increment (state) { 
        state.count++ 
    } 
}
```

Podemos testearlo de la siguiente forma:

```js
// mutations.spec.js 
import mutations from './mutations' 

test('increment increments state.count by 1', () => { 
    const state = { count: 0 } 
    mutations.increment(state) 
    expect(state.count).toBe(1) 
})
```

Bastante sencillo: inicio el state, ejecuto la mutation y compruebo que el estado tiene el valor esperado. Como paso el estado por referencia, la mutación se hace sobre el mismo objeto todo el rato.

## ¿Cómo mockeo un mutation en un componente?

Para mockear las mutations en un componente, lo hacemos igual que en el caso de los getters.

## ¿Cómo pruebo un action?

Los actions se caracterizan por permitirnos crear lógica que dependa de comportamientos asíncronos. En nuestro caso, el comportamiento asíncrono no es determinante ya que mockearemos todo aquello de lo que tengan alguna dependencia, por tanto, se comporta al final parecido a una mutación.

Tenemos el siguiente store:

```js
// mutations.js
export default {
  SET_DATA (state, data) {
    state.data = data
  }
}

// actions.js
export default {
  getAsync ({ commit }) {
    return axios.get('https://jsonplaceholder.typicode.com/posts/1')
      .then(response => commit('SET_DATA', response.data))
      .catch(err => console.log(err))
  }
}
```

Vemos que tenemos varias complicaciones para probar la acción getAsync de manera aislada. Vemos que el elemento dependn de una llamada AJAX con axios y que una vez que tenemos el dato, debemos mockear la mutación.

Esto lo hacemos de la siguiente manera:

```js
// actions.spec.js
import actions from './actions'
import flushPromises from 'flush-promises'

jest.mock('axios')

describe('actions', () => {
    it('tests with a mock commit', async () => {
        let count = 0
        let data
        let mockCommit = (state, payload) => {
            data = payload
            count += 1
        }

        actions.getAsync({ commit: mockCommit })

        await flushPromises()

        expect(count).toBe(1)
        expect(data).toEqual({ title: 'Mock with Jest' })
    })
})
```

* Con `jest.mock`, mockeamos la llamada get de axios como hicimos en el post anterior.
* Luego creamos una mutation a nuestro gusto que nos permitirá jugar con los datos obtenidos por la action
* Ejecutamos la action getAsync. Le pasamos el mockCommit para que cuando termine ejecute esa mutation.
* Hacemos las comprobaciones sobre los datos obtenidos.

Bueno... pues eso, que entre probar esto y nada, lo mismo nos da XD. Hay tanto mockeo que el código me parece demasiado adulterado. Es por etso que quizá para este caso tan pequeño, el action sea mejor probarlo con un test sobre un store completo y ver el mecanismo completo de action-&gt;mutation-&gt;getter.

## ¿Cómo mockeo un action en un componente?

El mockeo de un action en un componente es igual que en el caso de los getters y los mutations. Se trata de incluir vuex  en la instancia local y mockear los métodos de los que depende el componente.

Dado el siguiente componente:

```js
<template>
  <div class="text-align-center">
    <input type="text" @input="actionInputIfTrue" />
    <button @click="actionClick()">Click</button>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
export default{
  methods: {
    ...mapActions([
      'actionClick'
    ]),
    actionInputIfTrue: function actionInputIfTrue (event) {
      const inputValue = event.target.value
      if (inputValue === 'input') {
        this.$store.dispatch('actionInput', { inputValue })
      }
    }
  }
}
</script>
```

Lo testeamos de esta forma:

En el fichero `action-mock.spec.js` importamos las dependencias necesarias. En este caso `vue-test-utils`, `vuex` y el propio componente:

```js
import { shallow, createLocalVue } from '@vue/test-utils' 
import Vuex from 'vuex' 
import Actions from '@/components/ActionMock'
```

Instanciamos una copia local de Vue y le incluímos vuex:

```js
const localVue = createLocalVue() 
localVue.use(Vuex)
```

Dentro del test, definimos  las actions a mockear y generamos un store como mockeo:

```js
describe('ActionsMock.vue', () => { 
    let actions 
    let store 

    beforeEach(() => { 
        actions = { 
            actionClick: jest.fn(), 
            actionInput: jest.fn() 
        } 
        store = new Vuex.Store({ state: {}, actions }) 
    })
```

Ahora ya podemos crear los diferentes casos de uso sobre el componente. Simplemente, cuando montemos un componente, deberemos indicarle el store del que debe tirar:

```js
it('calls store action "actionInput" when input value is "input" and an "input" event is fired', () => { 
    const wrapper = shallow(Actions, { store, localVue }) 
    const input = wrapper.find('input') 
    input.element.value = 'input' 
    input.trigger('input') 
    expect(actions.actionInput).toHaveBeenCalled() 
})
```

Podemos ver el ejemplo todo junto:

```js
import { shallow, createLocalVue } from '@vue/test-utils' 
import Vuex from 'vuex' 
import Actions from '@/components/ActionMock'

const localVue = createLocalVue() 
localVue.use(Vuex)

describe('ActionsMock.vue', () => { 
    let actions 
    let store

    beforeEach(() => { 
       actions = { 
          actionClick: jest.fn(), 
          actionInput: jest.fn() 
       } 
       store = new Vuex.Store({ state: {}, actions }) 
    })

    it('calls store action "actionInput" when input value is "input" and an "input" event is fired', () => { 
        const wrapper = shallow(Actions, { store, localVue }) 
        const input = wrapper.find('input') 
        input.element.value = 'input' 
        input.trigger('input') 
        expect(actions.actionInput).toHaveBeenCalled() 
    }) 
})
```

Se trata de añadir un poco de fontanería para que trabajemos sobre lo que nos interesa.

## Conclusión

Como observamos, hacer pruebas en vuex tiene más que ver con probar piezas JavaScript separadas que con algo que tenga que ver con el propio framework. Hay poca interacción de las librerías y habrá que tener cuidado, como siempre, con las dependencias de cada pieza y con qué llamadas mockeamos.

Por ahora, con que tengamos estos conocimientos sobre testing en VueJS podemos empezar a trabajar. Con el tiempo necesitaremos profundizar y mejorar en técnicas, pero las peculiaridades a las que nos lleva el frameworks las tenemos cubiertas.

Si os habéis quedado con ganas de más, os vuelvo a recomendar el libro ‘[Testing VueJS components with Jest](https://leanpub.com/testingvuejscomponentswithjest)‘ de [Alex Jover](https://twitter.com/alexjoverm) y el Curso de Codely ‘[Testing con VueJS y Jest](https://www.youtube.com/watch?v=esFGn_8S_mw)‘ de [Javi Rubio](https://twitter.com/jrubr) y [Alberto Gualis](https://twitter.com/gualison) que os van ayudar a profundizar y a tener una sensibilidad mayor a cómo deben probarse los elementos en VueJS.

Nos leemos :\)

