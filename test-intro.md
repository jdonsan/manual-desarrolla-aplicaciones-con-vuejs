# Capítulo 21. Introducción

Buenas! Os tenía muy abandonados y abandonadas! Perdonadme, entre una cosas y otras, no he sacado ni el tiempo ni las ganas suficientes para escribir algo que me apeteciera de verdad por aquí. Vengo con las pilas cargadas y nuevos temas de los que hablar.

Os había comentado en post anteriores que nos íbamos a alejar una temporada de VueJS, pero me está siendo difícil... Como sabréis la mayoría, la serie de artículos que escribí aquí en El Abismo sobre VueJS se hizo bastante viral y eso ha hecho que mucha gente preguntase sobre qué pasó con ese gran olvidado de siempre: El testing.

Tengo que contaros que llegué un poco flojo de fuerzas hacia el final de la serie y sacrifiqué la parte de cómo testear una aplicación VueJS. Estaba en la planificación, lo prometo, pero se quedó por el camino.

Como bien me comentó [Alberto Moratilla](https://twitter.com/4lberto) \(principal precursor y cabeza pensante del manual de GitBook\), no incluir esta parte en la serie da una sensación 'mala'. Olvidándonos del testing, estamos dando a entender que la calidad del código que estamos desarrollando no nos es importante. Sin embargo, sabemos que esto no es así; testear y probar nuestro código de una manera automática es la clave para que nuestra aplicación resista mejor al tiempo y a los cambios.

Por otra parte, he ido posponiendo la escritura de esta parte porque la comunidad ya cuenta con dos recursos buenísimos sobre testing en VueJS.  Estos son, el libro '[Testing VueJS components with Jest](https://leanpub.com/testingvuejscomponentswithjest)' de [Alex Jover](https://twitter.com/alexjoverm) y el Curso de Codely '[Testing con VueJS y Jest](https://www.youtube.com/watch?v=esFGn_8S_mw)' de [Javi Rubio](https://twitter.com/jrubr) y [Alberto Gualis](https://twitter.com/gualison) que puede que sean bastante mejores y amplios que esta serie por el material y la experiencia que tienen los tres en esta materia.

Pero bueno, como las cosas hay que terminarlas y hay que intentar terminarlas bien y con una cobertura aceptable, empecemos, con muchas ganas e ilusión, a hablar de testing y VueJS:

## ¿Por qué necesito testear?

Un test es una serie de pasos que hay que seguir en una aplicación para que se de un determinado resultado esperado. Hay muchos tipos de test según lo que queramos probar ya sea de manera manual \(una persona se encarga de ejecutar los pasos\) o automática \(el propio ordenador es programado para ejecutar estos pasos\).

Nosotros en todo momento nos centraremos en explicar mecanismos automáticos. De esta manera, podremos agilizar procesos y evitar realizar tareas tediosas que un ordenador puede hacer mejor que nosotros.

El tipo de test en el que nos centraremos es el denominado test unitario. Los test unitarios son los tests encargados de probar, de manera aislada, cada una de las piezas y sus posibles configuraciones de las que está compuesta una unidad de código. Por unidad de código podemos pensar en un componente, una clase, una función o un procedimiento. Dependerá de la prueba que queramos llevar a cabo.

Los tests nos ayudan en muchas cosas importantes. Las que a mi más me gustan y me aportan son estas:

* **Nos dan feedback: **de una manera muy rápida de cuál es el estado de mi código ante un cambio que yo haya podido realizar. Es decir, que si yo tengo una batería de pruebas incluida en mi aplicación, puedo saber en todo momento si he roto algo o no con cada uno de mis cambios inmediatos.
* **Nos alertan de posibles bugs. **Son una buena red sobre la que hacer equilibrismo. Pensemos. Estamos en JavaScript, un lenguaje débilmente tipado y dinámico. Es un lenguaje muy propicio para cometer errores. Qué bueno sería tener un sistema automatizado que se encargue de comprobar que dadas unas entradas de configuración de mis unidades lógicas, siempre se reciban las mismas salidas como respuesta.
* **Nos ayudan a escribir mejor código**. Escribir tests es algo complejo que necesita cierta disciplina y orden. El cómo nombremos las cosas, el cómo modularicemos, dividamos y aislemos es clave para testear y conseguir piezas reutilizables. El testing nos ayuda a evitar acoplamientos y rápido nos señala si estamos cometiendo alguna mala decisión de diseño.
* **Nos generan una documentación viva** de lo que puede y de lo que no puede hacer nuestro código. Cuando un desarrollador o desarrolladora entra nuevo en un equipo de front, lo primero que suele hacer es mirar el \`package.json\` para saber con qué dependencias va a trabajar y mirar la batería de test para empezar a asimilar los diferentes caminos y bifurcaciones por las que el código posiblemente le pueda llevar.

Hay más razones, pero creo que son suficientes cómo para que tomemos las técnicas de pruebas automáticas como una tarea importante. Así que, veamos que necesitamos para probar nuestra aplicación de manera eficiente.

## ¿Qué necesito para testear?

Estamos en un ecosistema bastante peculiar. Hablamos de front, un front suele necesitar un navegador para renderizar elementos. En este caso, elementos Web. Además, nos hará falta alguna herramienta que esté pendiente de los cambios y de cuando ejecutar los test.

Nos vendría bien algún tipo de framework que nos facilite el trabajo a la hora de probar cosas... y no solo eso. Trabajamos con VueJs, con ES6, con Webpack... necesitamos herramientas que se integren bien en nuestro flujo de trabajo y que sepan trabajar juntas.

Así que bueno... empecemos a ver qué necesitamos en realidad:

### Necesitamos un test runner

Un test runner es una herramienta encargada de ejecutar una batería de pruebas de manera automática y según una configuración previa. Los test runner suelen encargase de orquestar todas las piezas para que podamos ejecutar test en un entorno aislado.

Hay muchos. Los más comunes Karma y Jest. En nuestro caso en particular, tenemos que elegir un test runner que cumpla una serie de funcionalidades mínimas.

1. Lo primero es **que sepa trabajar con JSDOM**. JSDOM es una forma de virtualizar el mecanismo del DOM de los navegadores pero sin los navegadores. JSDOM es una librería de NodeJS que implementa el estándar 
   [WHATWG DOM](https://dom.spec.whatwg.org/). De esta manera, si quiero ejecutar tests en un sistema de integración continua, no necesito tener instalados navegadores como Chrome o Firefox para probar partes muy acopladas al API DOM gracias a esta herramienta.
2. Lo segundo es **que se lleve bien con los Single File Component.** Los SFC son los ficheros `.vue` donde se encuentra todo lo necesario de un componente Vue \(todo el HTML, CSS y JS\) para que funcione. Webpack y vue-loader son los encargados de romper estos ficheros y dividirlos en sus unidades de responsabilidad y los tests tiene que ser capaces. Por tanto, nos tiene que permitir esto.
3. Luego están las funcionalidades de performance. Pensemos que nuestro proyecto puede tener una batería de pruebas bastante grande y que necesitaremos que los test se ejecuten periódicamente y que me den feedback en cada cambio que realice, por lo tanto, necesitamos algo que compile, renderice y ejecute test de la manera más rápida posible.

De todos los test runners que hay en el mercado, **Jest es el que más o menos cumple con todas ellas.** Jest es el test runner + test framework \(ya veremos luego esto\) creado por Facebook, sobre todo pensado para probar aplicaciones React, pero que es tan agnóstico que otros ecosistemas como VueJS lo están incluyendo en sus flujos de trabajo.

Jest se caracteriza por ser un test runner rápido, con una CLI muy intuitiva y clara que permite filtrar y ordenar la ejecución de test según nuestras necesidades, así como la de cachear y estar atento a cuándo y cómo se ha cambiado un tests o el código que se prueba. Además, es un test runner que está configurado por defecto con JSDOM y que por lo general, lleva a muy poca configuración, como veremos a continuación.

Para trabajar con Jest en nuestro proyecto de VueJS tenemos que hacer lo siguiente:

> NOTA: Doy por hecho que trabajaremos en un proyecto donde Webpack y vue-loader se encuentran en el flujo de trabajo. A lo largo de la serie hemos explicado todos estos conceptos

En un terminal, que se encuentre en la raíz de nuestro proyecto, ejecutamos lo siguiente:

```bash
$ npm install --save-dev jest
```

Con esto hemos instalado Jest y ya podemos trabajar con él. Lo siguiente es crear una tarea de npm para que podamos ejecutar los tests. Es tan sencillo como esto:

```js
// package.json

{ 
    "scripts": { 
        "test": "jest"
    } 
}
```

Y bueno... a partir de aquí está casi todo. Como te digo, Jest lleva muy poca configuración y todo va por convención. Si ejecutamos ese comando, Jest lo que va a hacer es buscar todos los ficheros que acaben en `.spec.js` o `.test.js` dentro del proyecto y ejecutar su código.

[Este patrón puede ser cambiado](https://facebook.github.io/jest/docs/en/configuration.html#testmatch-array-string). No es obligatorio. ¿Dónde colocamos estos ficheros de testing? Pues depende dónde más te guste. La convención indica que todos los tests se encuentren bajo una carpeta llamada `__tests__` en cada una una de las carpetas importantes del proyecto. Es decir:

```
/app
  /src
    /components
      /__tests__
    /stores
      /__tests__
    /routes
      /__tests__
```

Otros equipos colocan todos los tests a nivel de `src`. Es decir:

```
/app
  /src
  /tests
```

Esto ya es más a gusto de consumidor. Si usas la `vue-cli` e indicas que quieres tests unitarios con Jest, te lo incluirá de esta última manera.

Vale, hasta aquí guay, pero dónde configuro otro tipo de cosas. Jest es poco intrusivo, pero como se ve en su documentación, tiene una [gran cantidad de configuraciones.](https://facebook.github.io/jest/docs/en/configuration.html) Para configurar funcionalidades diferentes a las de por defecto, añadiremos un nuevo apartado dentro de `package.json` de esta manera:

```js
// package.json
{ 
    "jest": { 
        // ...
       "collectCoverage":true, 
       "collectCoverageFrom": [ 
           "**/*.{js,vue}", 
           "!**/node_modules/**"
       ], 
       "coverageReporters": ["html", "text-summary"],
       "moduleNameMapper": { 
           "^@/(.*)$": "<rootDir>/src/$1"
       }
    } 
}
```

Esta configuración está indicando que se active la opción de comprobar la cobertura de test \(`collectCoverage`\), que se recopile toda la cobertura de los ficheros `.js` y `.vue` que no se encuentren dentro de `node_modules` \(`collectCoverageFrom`\) y que la forma de mostrarlo sea por el terminal y en html \(`coverageReporters`\).

La última configuración lo que hace es permitirnos usar un alias \(en este caso @\) para poder hacer referencias absolutas a la raíz del proyecto \(`moduleNameMapper`\). Esto es algo que ya hacíamos en Webpack y a que nos va a ser útil ahora para importar módulos de una manera más sencilla.

> NOTA: Si no te gusta ensuciar el \`package.json\` con configuraciones extra, podemos llevarnos toda la configuración de Jest a un fichero separado.

A lo largo del post veremos nuevas configuraciones extra que tendremos que incluir por estar trabajando con VueJS.

### Necesitamos un traductor SFC-ES6

Los SFC son bastante peculiares. Necesitamos que de alguna manera Jest sepa cómo tratarlos para que luego podamos importar estos componentes dentro de nuestros tests y Jest no tenga problemas al ejecutarlo. Jest no tiene por defecto un convertidor de código `.vue` a código `.js`, pero sí nos deja incluir transformadores. Algo parecido a lo que hace Webpack con sus loaders.

Jest no tiene por defecto este transformador, pero el core de VueJS ha creado `vue-jest`, una librería encargada de transformar estos SFC a código ES6.

Para incluir esto en nuestro proyecto, tenemos que hacer lo siguiente:

Instalamos la librería `vue-jest`:

```bash
$ npm install --save-dev vue-jest
```

Y ahora incluimos la siguiente configuración en nuestro `​package.json` o en nuestro fichero de configuración Jest:

```js
// package.json
{ 
    // ... 
    "jest": { 
        "moduleFileExtensions": [ "js", "json", "vue" ], 
        "transform": { 
            ".*\\.(vue)$": "<rootDir>/node_modules/vue-jest" 
        } 
    } 
}
```

Con esto le decimos a Jest que los módulos que se podrán importar en nuestros ficheros de tests tienen la posible extensión `js`, `json` y `vue` \(`moduleFileExtensions`\) y que cuando encuentre una importación con un fichero que termine en `.vue` que lo transforme con la librería `vue-jest` a ficheros `.js`.

Sencillo.

### Necesitamos un traductor ES6-ES5

Nos podría valer esta transformación, pero si nos ponemos un poco paranoicos, hay que tener en cuenta que nuestros componentes están escritos en ES6, por lo general.

Vale que Jest se ejecuta sobre NodeJS y que NodeJS ya soporta casi todas las funcionalidades de ES6, pero qué necesidad hay de preocuparnos cuando podemos convertir nuestro código a ES5.

Lo que vamos a hacer es incluir otra librería que nos permite usar Babel en Jest. Para ello hacemos lo siguiente:

Instalamos `babel-jest` como de costumbre:

```bash
$ npm install --save-dev babel-jest
```

E incluimos un nuevo transformador en Jest:

```js
{ 
    // ... 
    "jest": { 
        // ... 
        "transform": { 
            // ... 
            "^.+\\.js$": "<rootDir>/node_modules/babel-jest" 
        }, 
        // ... 
    } 
}
```

Al igual que antes, lo que hace Jest es ejecutar un transformador sobre todos los ficheros con extensión `.js`

### Necesitamos una herramienta que manipule componentes Vue

Por parte del runner y de los preprocesamiento tenemos todo. ¿Ahora qué? Pensemos que un componente VueJS tiene un ciclo de vida y unas peculiaridades visuales. Testear elementos tan arraigados al DOM suele ser complejo.

Necesitamos un mecanismo que nos permita renderizar componentes, montarlos, crearlos, destruirlos, lanzar sus eventos nativos y personalizados a nuestro gusto o mockear partes si fuera necesario.

Para ello, la gente de Vue ha pensado en cómo hacerlo y han creado una librería que nos permite realizar todo esto. Se llama `vue-test-utils`. Es una librería agnóstica al framework de testing que uses y al test runner. Por lo que si Jest no te convence para probar componentes de Vue, siempre podrás seguir usando `vue-test-utils` con tu ecosistema favorito.

Para incluir la librería en nuestro proyecto, hacemos lo siguiente:

```bash
$ npm install --save-dev @vue/test-utils
```

Por ahora solo la instalamos. En el siguiente capítulo veremos todo su potencial.

### Necesitamos un framework de testing

Todo test suele tener un sistema para modularizar y jerarquizar los tests por medio de suites. Estas suites a su vez cuenta con casos de uso que se encargan de confirmar un comportamiento específico de un elemento.

Necesitamos también una serie de matcher o checkers que se encarguen de confirmar de una manera más semántica si las cosas son correctas o si no lo son. En definitiva, necesitamos un framework JavaScript de tests que nos permita escribir casos de una manera clara y rápida.

En este caso, no pasa como otros test runners que solo ejecutan test, si no que Jest además incluye toda la librería de pruebas de manera global para trabajar en el desarrollo de tests. No tenemos que instalar nada, con tener Jest, lo tenemos.

Aunque lo veremos a lo largo de la serie, [es bueno que eches un vistazo a la sintaxis de esta parte de Jest](https://facebook.github.io/jest/docs/en/getting-started.html).

## Nuestro primer test en Jest

Si toda la configuración ha ido bien, ya estaríamos preparados para escribir nuestro primer test. Como en todo el mundo del desarrollo, nuestro primer test va a ser para probar que hemos hecho bien un \`Hello World'. En nuestro caso un 'Hello Name'.

Así que lo primero que hacemos es crear un fichero `hello-name.js` en nuestro proyecto con el siguiente código:

```js
// ./src/hello-name.js
export default function (name) {
    return 'Hello ' + name;
}
```

Ahora lo que hacemos es crear un fichero de test. Lo llamamos `hello-name.spec.js` y escribimos los siguiente:

```js
// ./test/hello-name.spec.js
import helloName from '@/hello-name'

describe('Tests File hello-name.js', () => {
    it('Should get 'Hello Jose', () => {
         const name = 'Jose'
         const message = helloName(name)
         expect(message).toEqual('Hello ' + name)
    })
})
```

Todos los test, más largos, más pequeños, más complejos, más simples tienen una estructura parecida a esta.

Lo primero que se hace es importar el módulo que se quiere probar en una batería completa de tests. En este caso importamos ' hello-name'. Lo siguiente es generar una suite \(`describe`\) con una descripción clara de qué unidad lógica vamos a probar. En este caso vamos a probar casos posibles de 'hello-name'.

Los siguiente es incluir casos de uso \(`it`\). Cada uno de los casos de uso suelen tener un mecanismo parecido: Configuración-ejecución-comprobación. En este caso la configuración es la variable `name`. La ejecución es la ejecución de la función del módulo `helloName` y la comprobación es el matcher final con el `expect` y el `toEqual`.

Con esto, si ejecutamos en el terminal `npm test`, debería de funcionar y de conseguir nuestro primer test correcto. Un test mu simple, pero que ya demuestra toda la potencia de la automatización.

Esta suite no está acabada. ¿Qué ocurre si yo no indico un parámetro a la función? ¿Cómo se comporta nuestro módulo e este caso? Recuerda que crear tests unitarios es una forma de forzar todas las posibles salidas de un sistema. Se imaginativo y piensa en usos de caso que podrían darse. Usa mucho los informes de cobertura para comprobar si un módulo tiene partes sin cubrir con un test.

## Conclusión

Al final, la instalación y los primeros conceptos suele ser algo laborioso. Pero creedme si os digo que Jest es de las herramientas JavaScript que menos configuración tiene. En este caso las peculiaridades de VueJs ha hecho que se complique un poco más. Pero ya está listo para trabajar con VueJS.

Como he dicho en los primeros apartados. Este bloque de testing en VueJS quiero que sirva para concienciar de la importancia de incluir test. Quiero concienciar sobre los tiempos que nos puede ahorrar, sobre las decisiones que nos puede hacer tomar.

Muchos cargos y mucha gente con la que me he encontrado en estos años suele ver el testing automático como una pérdida de tiempo o como un tiempo que no genera los beneficios esperados. Yo os digo que aprender cuesta y que cambiar actitudes y hábitos es complicado.

Yo mismo estoy en pleno proceso de aprendizaje y cambio de mentalidad, pero creedme que cuando los tests salen, empiezan a dar feedback y empiezas a ver un mejor código, vuestra mentalidad también cambiará. Hay que tener mucha paciencia y mucha constancia.

En el próximo capítulo empezaremos a explicar en detalle cómo empezar a testear componentes y veremos las facilidades que nos dará `vue-test-utils` para crear test más legibles y fáciles de desarrollar. Veremos cuales serán las mejores partes de testear los componentes e intentaremos entender mejor todo el flujo de trabajo con Jest.

Por ahora lo dejamos aquí

Nos leemos :\)

