# Capítulo 22. Testeando nuestros componentes

En el post anterior, explicamos de manera superficial cómo crear pruebas con Jest. Conseguimos configurar el entorno e incluimos todas las librerías necesarias para poder trabajar en un entorno típico de VueJS.

En el post de hoy, entraremos en más detalle de cómo nos va ayudar `vue-test-utils`  a crear test para VueJS, qué elementos de un componente nos interesa probar y qué peculiaridades vamos a tener a la hora de crear tests para nuestras pequeñas vistas.

Como hemos dicho hasta ahora, por lo general, un test intentará probar unidades de código como si fuesen cajas negras. Creados unos datos de entrada, nuestro código tiene que devolver unos datos de salida, el cómo lo consiga es un problema de implementación.

En nuestro caso, nuestra unidad de código mínima será un componente, un componente visual, un componente con su HTML, CSS y JavaScript. Lo que tendremos que hacer en las pruebas unitarias de componentes es inyectar unos datos de entrada determinados y obtener aquel HTML renderizado de la manera que nosotros deseamos.

También será necesario que lancemos todos aquellos eventos para los que nuestro componentes tienen comportamientos implementados y vigilaremos que el estado se encuentre correcto según nuestras especificaciones de negocio.

Pero bueno... como es mejor verlo que contarlo, empecemos a trabajar con `vue-test-utils`:

## ¿Cómo renderizamos un componente para testearlo?

Lo primero que necesitamos es una forma de renderizar en memoria un componente y contar con alguna herramienta que nos permita confirmar que ciertos elementos se encuentran donde deben.

Vale, no contamos con navegador, no podemos renderizar en un entorno real un componente. ¿Cómo lo vamos a hacer? `vue-test-utils` cuenta con dos funcionalidades muy útiles para renderizar un componente en memoria: `mount` y `shallow`.

Ahora veremos qué diferencia hay entre estas dos funcionalidades por ahora tenemos que saber que nos permite renderizar componentes VueJS. Pensemos en el siguiente componente de VueJS:

```js
<template>
    <div> 
        <span class="count">{{ count }}</span> 
        <button @click="increment">Increment</button> 
   </div>
</template

<script>
export default {
    data () { 
        return { count: 0 } 
    }, 
    methods: { 
        increment () { 
            this.count++ 
        } 
    }
}
</script>
```

Este componente lo único que hace es gestionar un contador que el usuario puede ir aumentando. Este componente se caracteriza por no tener datos de entrada \(`props`\) y no tener como dependencia a otros componentes o librerías.

Para poder hacer pruebas sobre este componente, creamos un fichero llamado `counter.spec.js` e incluimos su primer test:

```js
// counter.spec.js
import { mount } from '@vue/test-utils' 
import Counter from './counter'

describe('Component Counter', () => {
    it('should render the correct markup', () => {
        const wrapper = mount(Counter)
        expect(wrapper.html()).toContain('<span class="count">0</span>')
    })
})
```

Este primer test \(nada útil, pero sí muy didáctico\), lo único que hace es comprobar que el contenido renderizado del componente es el correcto. ¿Cómo hacemos esto?

* Primero importamos la utilidad `mount` que nos permite renderizar componentes en memoria
* Importamos el componente  \(`Counter`\) a probar.
* Creamos una nueva suite para el componente con `describe`.
* Creamos un caso de uso con `it`.
* Internamente del caso de uso, empieza el test. Primero montamos y renderizamos el componente `Counter`  con `mount`. Esto nos devuelve un envoltorio o `wrapper` que tiene un montón de métodos que nos permite jugar con facilidad con el elemento renderizado en memoria. Los que hayáis trabajado con jQuery, vais a disfrutar mucho con este envoltorio 
  [ya que nos permite hacer pequeñas consultas sobre este DOM renderizado en memoria](https://vue-test-utils.vuejs.org/en/api/wrapper/).
* Por último esperamos que dentro del html renderizado \(`wrapper.html()`\), exista el contenido indicado en `toContain.`

Fácil ¿no?

Como habéis visto, en el ejemplo hemos usado `mount`. La diferencia entre `mount` y `shallow` es que `mount` te renderiza y monta el componente en su totalidad y `shallow` no, Esto significa que si nuestro componente tuviese en su interior otros componentes hijos, `mount` también te los renderizaría, mientras `shallow` los ignoraría.

Que exista ambos sistemas tiene sentido. Puede que muchas veces queramos probar un componente de manera aislada, sin contar con sus dependencias \(`shallow`\) para hacer pruebas de ciclo de vida, de entradas y salidas, de gestión de eventos. Renderizar el resto de elementos solo va a aumentar tiempos de ejecución al test y además, arrastrarnos dependencias que no necesitamos.

Pero puede darse el caso que nuestro tests se comporte más como un test de integración y que lo que nos interese sea ver cómo un componente orquesta a componentes hijos, ver cómo obtengo información de un componente y ver si la inyección en otro funciona correctamente.

Saber cuando elegir uno y otro será la clave para el tester.

## ¿Qué partes deberíamos testear de un componente y cómo?

Un componente de VueJS tiene estas posibles entradas y salida:![](https://elabismodenull.files.wordpress.com/2018/04/captura-de-pantalla-de-2018-04-02-17-55-43.png "Captura de pantalla de 2018-04-02 17-55-43.png")Dependiendo de cómo configuremos un componente \(`props`\), cómo sea la interacción del usuario y cómo sea el ciclo de vida, se darán unas salidas u otras.  Además, hay que pensar que un componente puede tener en su interior otras componentes de manera estática o dinámica \(slots\).

### Manipulando el estado del componente

Un componente tiene un estado externo o propiedades de configuración llamadas `props` y tiene un estado interno llamado `data`.

En todo momento un componente montado locálmente cuenta con dos métodos para realizar renderizados reactivos. Estos son los siguientes:

```js
wrapper.setData({ count: 10 }) 
wrapper.setProps({ foo: 'bar' })
```

También podemos 'mockear' las propiedades de un componente. Tenemos el mismo componente `Counter` de antes, solo que ahora permitimos que se pueda configurar por defecto

```js
<template>
    <div> 
       <span class="count">{{ count }}</span> 
       <button @click="increment">Increment</button> 
   </div>
</template 

<script> 
export default {
    props: {
        count: {
            type: Number,
            default: 0
        }
    },
    methods: { 
        increment () { 
            this.count++ 
        } 
    }
}
</script>
```

Para este componente con la `prop` numérica `count` tengo dos posibles casos de uso: La propiedad al ser opcional, me va a permitir indicarle una propiedad o no. Por tanto hagamos los dos casos de uso:

El primero es sencillo:

```js
// counter.spec.js 
import { mount } from '@vue/test-utils' 
import Counter from './counter' 

describe('Component Counter', () => { 
   it('should set count with data default', () => { 
        const wrapper = mount(Counter)
        expect(wrapper.find('.count').text()).toBe(0)
   })
   // ...
})
```

Lo que hacemos es montar el componente `Counter` sin pasar ninguna propiedad 'mockeada' y comprobar que el texto que hay en el elemento con clase `count` es el valor por defecto \(en este caso 0\).

El segundo caso es  en el que el desarrollador sí le va a indicar un valor al componente. Lo hacemos así:

```js
// counter.spec.js 
import { mount } from '@vue/test-utils' 
import Counter from './counter' 

describe('Component Counter', () => { 
     //...
     it('should set count with data default', () => { 
         const countDefault = 10
         const wrapper = mount(Counter, {
             propsData: { count: 10 }
         }) 
         expect(wrapper.find('.count').text()).toBe(countDefault)
     })
     // ...
})
```

Montamos el componente con lo que se indique en `propsData`. El ejemplo es tontísimo, pero se ve la potencia de cómo se comporta el componente internamente dependiendo del valor indicado.

Si ahora un desarrollador decide que la propiedad por defecto es otra o que ya no tiene que tener valor por defecto, el test nos lo chivará.

### Simulando las interacciones del usuario

Estamos tan cerca del usuario, que la mayoría de los cambios de estado los va a provocar el propio usuario. Es por eso que necesitamos un mecanismo para lanzar esos eventos y que el componente reaccione a los cambios registrados.

Siguiendo con el componente `Counter`, hagamos un caso de uso dónde el usuario decida incrementar el contador en uno. El test podría ser este:

```
//...
it('button click should increment the count', () => { 
    const wrapper = mount(Counter)
    expect(wrapper.vm.count).toBe(0) 
    const button = wrapper.find('button') 
    button.trigger('click') 
    expect(wrapper.vm.count).toBe(1) 
})
//...
```

Lo que hacemos es comprobar primero que, al instanciarse el componente, el valor sea el esperado por defecto \(0\). Lo siguiente es acceder al botón del componente y luego lanzar un evento `click` por medio del método que tiene el `wrapper` llamado `trigger`.

Esto ejecutará funciones internas \(o no\). El caso esperado es que el valor de `count` se haya incrementado en 1.

Con `trigger` podemos lanzar todos los eventos nativos y podemos usar los modificadores de eventos.

### Confirmando eventos personalizados

Cuando un componente hijo quiere transmitir información acciones a un componente padre lo hace por medio de eventos personalizados. ¿Pero cómo confirmo si un componente ha emitido un evento personalizado?

El `wrapper` es nuestro mejor amigo y tiene métodos que nos confirman si estas cosas han ocurrido o no.

Volvemos a tener el componente contador. Solo que en este caso además de incrementar el valor, emite el nuevo valor para que cada padre haga con el valor lo que desee:

```javascript
<template>
     <div> 
         <span class="count">{{ count }}</span> 
         <button @click="increment">Increment</button> 
     </div>
</template>

<script> 
export default {
    props: {
        count: {
            type: Number,
            default: 0
        }
    },
    methods: { 
        increment () { 
            this.count++ 
            this.$emit('increment', this.count)
        } 
    }
}
</script>
```

El caso de uso, sería de esta manera:

```javascript
import { mount } from '@vue/test-utils'
import Counter from './Counter'

describe('Component Counter', () => {
    //...
    it('should emit event increment', () => {
        const wrapper = mount(Counter)
        const button = wrapper.find('button')
        button.trigger('click')

        expect(wrapper.emitted().increment).toBeTruthy()
        expect(wrapper.emitted().increment[0]).toEqual([1])
   })
   // ...
})
```

Vuelvo a ejecutar el `click` que es el que provoca la emisión de `increment`. Lo que hacemos luego es comprobar que se ha registrado un evento de tipo `increment` con el método del `wrapper` llamado `emitted`.

## ¿Cómo testeo un componente que tiene dependencias?

Nos va a pasar que algunos componentes tengas dependencias de otros plugins de VueJS. Imagina que tu componente usa `vuex` o `vue-router`.  Como los componentes se renderizan de manera aislada y fuera del propio framework en una instancia de VueJS local, todas estas dependencias que usa, van a provocar comportamientos inesperados. Son efectos laterales que deberíamos evitar.

Para conseguir que nuestro plugin tenga estas dependencias, contamos con `createLocalVue`. Esta pieza nos permite renderizar y montar nuestro componente bajo una copia local de VueJS donde ya sí podemos incluir plugins.

Lo hacemos de la siguiente forma:

```js
import { createLocalVue } from '@vue/test-utils'
import { MyPlugin } from './my-plugin'

const localVue = createLocalVue() 
localVue.use(MyPlugin) 

mount(Component, { localVue })
```

Lo único que hacemos es importar `createLocalVue` y nuestro plugin. Creamos una instancia de VueJS y le instalamos el plugin con `localVue.use()`. Lo siguiente es decirle a `mount` o `shallow` que use nuestra instancia local de Vue porque es la que cuenta con las dependencias globales necesarias.

Puede que a veces incluir toda una librería global sea matar moscas a cañonazos. Puede darse el caso que en un componente se use algo ínfimo y que no merezca la pena. En este caso podemos inyectar un mock para que se hagan pruebas con él.

Imaginemos que queremos mockear `vue-router` en un componente. Podríamos hacer esto:

```js
import { mount } from '@vue/test-utils'

const $route = { 
    path: '/', 
    hash: '', 
    params: { id: '123' }, 
    query: { q: 'hello' } 
} 

mount(Component, { 
    mocks: { 
        $route 
    } 
})
```

Cuidado con lo explicado en este apartado ¿vale? Que tengamos esta facilidad no significa que estemos usando bien el framework. Existirán casos donde nuestro componentes estén tan acoplados al proyecto, que hacer esto no suponga un problema. Es decir, un componente muy acoplado al proyecto, que no sea posible reutilizarlo, no tiene problemas a que tenga como dependencias elementos globales del framework.

Ahora bien ¿Qué pasa si necesitamos componentes muy reutilizables y que se puedan usar en varios proyectos? Que tu tengas un plugin instalado en un proyecto, no tiene que significar que en otro sí. Por lo que si queremos usar ese componente, también nos tendremos que llevar el plugin. Abusar de `createLocalVue` y la inyección de plugins puede ser un buen 'Code Smell'.

Intenta que solo los componentes orquestadores \(también llamados controladores o tipo page\) tengan estas dependencias. Evita que componentes muy visuales puedan realizar uso de piezas tan globales o las estarás acoplando.

Tenlo en cuenta.

## ¿Cómo pruebo comportamientos asíncronos?

Tenemos el siguiente componente:

```javascript
<template> 
    <button @click="fetchResults" /> 
</template> 

<script> 
import axios from 'axios' 
export default { 
    data () { 
        return { 
            value: null 
        } 
    }, 
    methods: { 
        async fetchResults () { 
            const response = await axios.get('mock/service') 
            this.value = response.data 
        } 
    } 
} 
</script>
```

Testear este componente nos va a dar problemas por dos razones:

1. Tenemos una dependencia super hardcodeada en el componente como es el uso de 
   `axios`
2. Tenemos un método con comportamiento asíncrono, por tanto puede dar problemas en lo resultado esperados.

### Dependencias hardcodeadas y los mocks de Jest

El primer problema se soluciona de una manera simple: Mockeando la parte de `axios` que nos interesa y que devuelva los datos que deseamos. Para hacer esto, lo vamos a hacer de manera genérica. Jest nos permite mockear métodos de librerías.

Para ello incluimos una carpeta de test que tenga este nombre `__mocks__`. La nomenclatura es la típica de Jest, así que nada que objetar. Lo siguiente que hacemos es incluir el siguiente fichero `axios.js` dentro:

```js
// ./test/__mocks__/axios.js

export default { 
    get: () => new Promise(resolve => { 
        resolve({ data: 'value' }) 
    }) 
}
```

De esta manera, Jest lo que hará será usar este código cuando decidamos usar `axios` como un mock.

Para indicarle a Jest dentro de una suite que haremos uso del mocks, solo tenemos que indicar lo siguiente:

```js
import { shallow } from '@vue/test-utils'
import Foo from'./Foo'
 

jest.mock('axios') 

describe('Foo', () => { })
```

Con `jest.mock('axios')`, Jest ya sabe que tiene que ir a por un mock y usarlo en nuestro código. Lo inyecta de manera automática, no tengo que preocuparme de nada más.

Volvemos a lo mismo de antes, habrá casos en lo que no nos quedará más remedio que hardmockear de esta manera las dependencias, pero por lo general indica un 'Code Smell' de que no se están inyectando las dependencias correctamente.

Usa las propiedades para pasar funciones a su componentes hijos, emite eventos para que componentes más acoplados se encarguen de estas llamadas, envuelve las librerías en piezas que sean fácil de intercambiar por mocks. Todo con tal de evitar estos mecanismos tan extremos.

### Las llamadas asíncronas

Vale, ya sabemos que `axios.get` nos devolverá una promesa a nuestro gusto. Pues hagamos su test:

```js
import { shallow } from '@vue/test-utils'
import Foo from './Foo'

jest.mock('axios')

describe('Foo', () => {
    it('fetches async when a button is clicked', () => {
        const wrapper = shallow(Foo)
        wrapper.find('button').trigger('click')
        expect(wrapper.vm.value).toBe('value')
    })
})
```

Vaya... el test ha fallado. No hemos gestionado la promesa correctamente y no hemos dado tiempo a que `wrapper.vm.value` tenga el valor correcto.

Para esto, `vue-test-utils` ya ha pensado en ello y nos permite esperar hasta que la promesa está resuelta. Veamos:

```js
import { shallow } from '@vue/test-utils'
import Foo from './Foo'

jest.mock('axios')

describe('Foo', () => {
    it('fetches async when a button is clicked', () => {
       const wrapper = shallow(Foo)
       wrapper.find('button').trigger('click')
       wrapper.vm.$nextTick(() => {
           expect(wrapper.vm.value).toBe('value')
       })
    })
})
```

Lo que hacemos es hacer uso de `$nexTick` de VueJS. Esta función se ejecuta cuando el DOM se vuelve a ejecutar. Como la promesa va a provocar cambios, esperamos a esos cambios.

El test no es correcto todavía porque bueno... el componente se comporta de manera asíncrona, pero el caso de uso, el test, es secuencial, no entiende de asincronismo tal y como está planteado, por tanto el test acabará antes de esperar a las confirmaciones \(`expect`\).

Para arreglarlo, Jest ya ha pensado en ello y nos permite una función de `callback` que nos permite indicar cuándo un test se tiene que dar por terminado. Lo hacemos así:

```js
import { shallow } from '@vue/test-utils'
import Foo from './Foo'

jest.mock('axios')

describe('Foo', () => {
    it('fetches async when a button is clicked', (done) => {
        const wrapper = shallow(Foo)
        wrapper.find('button').trigger('click')
        wrapper.vm.$nextTick(() => {
             expect(wrapper.vm.value).toBe('value')
             done()
        })
    })
})
```

¿Veis ese `done`? Hasta que no es ejecutado, no se da por terminado el test. Esto tiene un tiempo máximo, si pasado ese tiempo no se ha ejecutado `done`, Jest sigue con el resto de tests.

Puede que el sistema genere demasiados anidamientos o que sea lioso gestionar asincronía para un test. Podemos simplificar el código si usamos la librería `flush-promise`. Esta librería nos devuelve una promesa que no es resuelta hasta que no se han resuelto el resto de manejadores. El código ahora quedaría muy limpio:

```js
import { shallow } from '@vue/test-utils'
import flushPromises from 'flush-promises'
import Foo from './Foo'

jest.mock('axios')

describe('Foo', () => {
    it('fetches async when a button is clicked', async () => {
        const wrapper = shallow(Foo)
        wrapper.find('button').trigger('click')

        await flushPromises()
        expect(wrapper.vm.value).toBe('value')
    })
})
```

Sin `done`, sin `$nextTick`. Lo que hacemos es una `async function` y esperar a que todos los manejadores se den por terminados. En ese momento es cuando estamos preparado para hacer las confirmaciones.

## Conclusiones

Parece que hemos explicado mucho, pero solo hemos explicado la superficie del testeo de componentes. Sí, sabemos lo principal, pero queda mucho trabajo de cómo inyectar mocks, stubs y demás mecanismos de testing.

¿Qué pasa cuando hay mucho código repetido en una suite? ¿Qué pasa con otros elementos como los `slots` o los `watcher`​? ¿Qué pasa si quiero mockear un CSS o un PNG?  Creo que son temas interesantes que por ahora se van a quedar fuera de este material.

Al igual que pasaba con todo el manual, estamos en una serie introductoria, hay mucho trabajo de práctica y profundización que tendremos que hacer después, pero creo que esto es un buen comienzo para conocer `vue-test-utils`.

Una cosa que queda patente en el posts, es que con la inclusión de tests en nuestro proyecto, no es suficiente. Los tests nos dan feedback, pero en muchas ocasiones sin un buen diseño de componentes, será difícil trabajar bien y hacer desarrollos de calidad \(término bastante subjetivo y que siempre lleva a debate\).

Evitemos poner dependencias a fuego. Hagamos piezas pequeñas, con una única responsabilidad. Encapsulemos bien los estados, definamos buenas entradas y salidas. Tengamos en mente siempre SOLID, KISS y YAGNI y nos haremos la vida más fácil.

Por el momento, es todo.

Nos leemos :\)

