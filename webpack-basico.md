# Capítulo 14. Conceptos básicos

Nos detenemos por un momento en el camino de vue y nos ponemos manos a la obra en entender Webpack. Aunque lo parezca, no nos hemos desviado de nuestra trayectoria.

Como pudimos ver en el capítulo donde estudiamos vue-cli, vimos que la plantilla nos generó una serie de ficheros donde ya se encontraba la construcción de nuestra aplicación de una forma mágica, sin que nosotros tuviésemos que desarrollar o crear nada, simplemente ejecutando un comando.

Estos ficheros se encontraban creados con Webpack y hacían todo lo necesario para no tener que mancharnos las manos nosotros. Como no vamos a basar el destino de un proyecto en la magia y los fuegos artificiales, es hora de que nos atemos los machos y entendamos qué estaba haciendo vue-cli por nosotros.

Puede ser que algún día necesitemos incluir algo en la configuración que no viene por defecto y que estos conocimientos nos sean muy útiles. Así que vamos al lío:

## ¿Qué es?

Webpack es un empaquetador de módulos creado con NodeJS. Entiéndase por módulos todo aquello que se encuentra separado en diferentes ficheros dentro de nuestra aplicación. Y no nos estamos refiriendo solo a los ficheros JS. Todo fichero, con la extensión que tenga, nos da igual, es un buen candidato para poder ser empaquetado en un producto final.

![Conversión de Webpack en paquetes](/images/webpack/what-is-webpack1.png)

Los empaquetadores de módulos tienen su hueco en el mundo Web por dar solución a dos problemas que hemos sufrido casi todos los desarrolladores: el manejo de un elevado número de ficheros y la gestión, a veces muy complicada, de las dependencias de nuestro proyecto y sus acoplamientos.

Expliquemos cada uno de ellos:

### Manejo elevado de fichero

Cuando un proyecto crece, ya estemos trabajando en una SPA o en una aplicación con varios HTML, empezamos a tener un problema a la hora de organizar nuestro código y nuestros assets. Cuando nuestros ficheros empiezan a crecer, es una buena práctica seguir ciertas recomendaciones para dividir el problema de nuestra solución en porciones más pequeñas. Los principios que seguimos basan el desarrollo en crear funcionalidades que tienen una única responsabilidad en ficheros por separado.

Por tanto, el número de nuestros ficheros empiezan a crecer y el número de dependencias que tenemos que incluir en nuestros HTML también crece. Llega un momento que la gran cantidad de scripts, hojas de estilo e imágenes a enlazar es tan grande que empieza a ser inmanejable.

Los empaquetadores de módulos como Webpack son buenas herramientas para solucionar esto porque nos permitirán seguir escribiendo nuestras funcionalidades en ficheros por separado, pero nos proporcionarán un proceso automático, donde todo se concentra en un único fichero para que sea más manejable.

Además, por la propia arquitectura de HTTP 1.1, no podemos olvidar que nos encontramos comunicándonos con servidores sin una sesión específica. Esto quiere decir que cada vez que un navegador hace una petición sobre un recursos (ya sea un JS, un CSS o un PNG) se crea un proceso de petición-respuesta con inicio y fin.

Ninguna de las peticiones que hagamos comparten un canal de difusión de recursos. Esto hace que el consumo de datos de red y los tiempos de carga aumenten y que los procesos de optimización sean importantes para todo desarrollador. Si conseguimos que todos nuestro código JS se encuentre en un único fichero, si conseguimos que todas nuestras clases están en un único CSS, reduciremos el número de llamadas a servidor y por tanto nuestra aplicación será mejor.

Por tanto, ya sea Webpack u otro empaquetador de módulos, necesitamos una herramienta así.

### La gestión de dependencias y su acoplamiento

El tener muchos ficheros enlazados en mi HTML no es solo un problema de rendimiento, es también un problema a la hora de no cometer errores. Nos pasa con la Web que tenemos que ser muy cuidadosos para no mantener sucio el contexto global ya sea para no pisar variables de manera indeseable ya sea por no cometer otros fallos.

El orden en que yo ordeno la carga de ficheros en mi HTML puede ser clave para que todo funcione correctamente. Puede que muchos de mis ficheros hagan uso de jQuery, pero puede pasar que si yo no coloco la librería exactamente en el lugar preciso donde el navegador tenga que cargarla, cuando vaya a usar la librería, mi código no funcione.

Los empaquetadores de módulos trabajan muy bien resolviendo dependencias. Webpack por ejemplo, una de las primeras cosas que hace es acceder al fichero de entrada de mi aplicación y construir un grafo con las dependencias que se van teniendo. De esta forma va incluyendo todo aquello que se necesita. Estos empaquetadores usan los sistemas de módulos como pueda ser el descrito por CommonJS, AMD, la implementación nativa de ES6 o el `@import` de CSS3.

Lo bueno de usar un empaquetador como Webpack es que no tendremos el problema de dependencias muertas. ¿No os ha pasado alguna vez que habéis incluido una librería en vuestro index.html y que por lo que sea no la habéis usado porque no hacía falta? Sin darnos cuenta estamos penalizando el rendimiento de nuestra webapp con algo que no se utiliza.

Con Webpack esto no puede pasar. Como el empaquetador se encuentra construyendo el grafo de dependencias desde el fichero inicial, no hay manera de incluir una librería que no estaba indicada como dependencia, haciendo nuestras aplicaciones más livianas y específicas.

## ¿En qué más nos ayuda Webpack?

Pero lo bueno de Webpack es que no es un simple empaquetador de ficheros al uso. Webpack nos ayuda en varias tareas más.

### Permite transformaciones del código

Con Webpack podemos dejar de lado gestores de tareas como gulp o grunt ya que vamos a poder crear un sinfín de tareas en tiempo de empaquetado. Una de estas tareas es la transformación del código. Dentro de Webpack podemos escribir código en diferentes lenguajes como TypeScript, CoffeScript o ES6 y Webpack se encargará de transpilarlos en JavaScript.

No solo ocurre esto con los ficheros en lenguaje JavaScript. Como decíamos, Webpack trabaja con muchos tipos de ficheros y nos permite escribir ficheros en Jade y ser transformados en HTML al ser utilizados. Podemos escribir nuestros estilos con SASS, Stylus o LESS y Webpack nos devolverá un CSS minificado y optimizado. Lo mismo con las imágenes: Webpack nos permitirá transformar imágenes en cadenas de base64.

Webpack es un sistema muy modularizado por lo que todas estas funcionalidades podrán ser añadidas en cualquier momento.

### Permite empaquetados parciales

Puede darse el caso que nuestra aplicación sea tan grande, que no deseemos crear un solo paquete de la aplicación. O puede darse el caso de que no estemos en una SPA y que queramos hacer paquetes diferentes para los HTML de nuestra aplicación.

Podremos crear módulos intermedios que interaccionan con otros módulos y que se podrán ir componiendo en otros paquetes según necesidad. Las alternativas de configuración aquí son las que nos de la imaginación y los límites que desde negocio nos impongan.

### Permite configurar empaquetados según el entorno y el target

También será posible el crear diferentes empaquetados dependiendo del entorno en el que deseemos desplegar. Puede darse el caso que ciertas partes de la construcción no tengan que ser iguales para desarrollo que para producción.

Puede darse el caso también de que el paquete que queremos utilizar, se vaya a usar en más de un tipo de contexto diferente. Como sabemos, JavaScript es cada vez más agnóstico a la plataforma y esto hace que seamos capaces de ejecutar código JavaScript tanto en navegadores, como en servidores, como en aplicaciones de escritorio y móvil. Puede que según las necesidades de la plataforma no queramos que nuestros módulos tengan la misma funcionalidad o que se encuentran empaquetados de formas distintas.

Webpack cuenta con una serie de plugins y librerías que nos harán este trabajo más fácil a los desarrolladores.

### Permite hacer compilaciones en caliente

Webpack cuenta con una funcionalidad que puede ayudar mucho en tiempo de desarrollo y es que existe algo llamado HMR (Hot Module Replacement). Lo que hace esta tecnología es mantener en tiempo de desarrollo una especie de compilador que nos permite actualizar aquellas partes del código que hemos ido cambiando sin tener que refrescar el navegador.

De esta manera, un desarrollador que ha encontrado un bug, puede cambiar su código, hacer CTRL + S y Webpack se encargará de actualizar el código y compilarlo de tal manera que mi webapp no se vea afectada. De esta forma evitamos perdida de datos o estados de manera innecesaria  cuando estamos haciendo alguna prueba en concreto. Es algo que ayuda mucho a la hora de perfeccionar una funcionalidad.

## ¿Cómo empezar?

Cómo utilizar todas funcionalidad tiene que ser uno de nuestros objetivos a lo largo de esta serie, vamos a empezar a explicar cómo utilizar y configurar todo lo necesario para conseguirlo:

Lo primero que tenemos que hacer es crear un nuevo proyecto de Node como es de costumbre:

```
$ mkdir testing-webpack
$ cd testing-webpack
$ npm init
```

Instalamos Webpack como dependencia del proyecto. Aunque Webpack presenta una CLI, es mejor que la usemos de manera local como dependencia para evitar los problemas de versiones entre proyectos:

```
$ npm install webpack --save
```

Después de configurar nuestro proyecto, crearemos la siguiente estructura de ficheros:

```
- testing-webpack
|-- dist
|-- src
    |-- utils.ts
    |-- main.ts
    |-- index.html
|-- webpack.config.js
```

Donde `src` es la carpeta en la que desarrollaremos nuestro código, `dist` la carpeta donde se incluirá todo aquello que queremos desplegar a producción y `webpack.config.js` es el fichero donde configuraremos todo lo relativo a Webpack y a cómo se tiene que empaquetar la aplicación.

Los ficheros `utils.ts` y `app.ts` son los dos módulos de la aplicación que queremos empaquetar. Están escritos en TypeScript y son estos:

```javascript
// utils.ts
export default {
    sayMyName(name: string) {
        console.log('Hello ' + name);
    }
};

// main.ts
import utils from './utils';

utils.sayMyName('Heisenberg');
```

Mi fichero `index.html` es este:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>testing webpack</title>
    </head>
    <body>
    </body>
</html>
```

Por último, vamos añadir en el fichero `package.json` el siguiente comando para que luego en el futuro nos sea más intuitivo ejecutar la construcción de nuestra aplicación:

```javascript
// package.json
{
    "name": "testing.webpack",
    "version": "0.0.1",
    "description": "Testing Webpack",
    "author": "jdonsan",
    "scripts": {
        "build": "webpack",
    },
    ...
}
```

Con esto, lo que conseguimos es que si ejecutamos:

```
$ npm run build
```

Se nos lance Webpack yendo al fichero por defecto `webpack.config.js`. Como ahora mismo no tiene configurado el proceso, el comando no hará nada en especial.

Para conseguir que haga cosas, primero tenemos que estudiar un poquito.

## Conceptos básicos

Al final todo parece muy complejo, pero entender Webpack se basa en estudiar sobre estos 4 conceptos:

### Los puntos de entrada

Dentro de una aplicación tenemos que indicar cuales son los puntos de entrada por los que queremos que un empaquetador empiece a trabajar. Tenemos que indicarlo para que se empiece a generar el grafo de dependencias.

En nuestro caso el punto de entrada de la aplicación es `/src/main.ts`. Para indicar a Webpack que empiece la inspección y el empaquetado de módulos por un punto de entrada concreto, lo hacemos así:

```javascript
// webpack.config.js
const config = {
    entry: './src/main.ts'
};

module.exports = config;
```

En nuestro caso, al realizar aplicaciones SPA suele ser normal contar con un solo punto de entrada, pero puede darse el caso de que queramos hacer paquetes separados para las diferentes páginas de nuestra aplicación.

Para lograr eso, podríamos hacer algo tal que así:

```javascript
// webpack.config.js
const config = {
    entry: {
        pageHome: './src/home/main.ts',
        pageProfile: './src/profile/main.ts',
        pageLogin: './src/login/main.ts'
    }
};

module.exports = config;
```

En este caso tenemos tres puntos de entrada para tres paquetes diferentes.

Otro caso puede ser el de que queramos separar nuestro código del resto de librerías externas. Podríamos hacer algo como esto:

```javascript
// webpack.config.js
const config = {
    entry: {
        app: './src/app.ts',
        vendors: './src/vendors.ts'
    }
};
```
### Las salidas

Una vez que hemos decidido que punto o puntos de entrada queremos en nuestra configuración, es el turno de indicarle a Webpack donde tiene que dejar los paquetes que ha ido generando.

Al igual que tenemos un punto de entrada, podemos indicar un punto de salida de la siguiente forma:

```javascript
// webpack.config.js

const config = { 
    entry: './src/main.ts',
    output: { 
        filename: 'app.min.js', 
        path: __dirname +  '/dist'
    }
}; 

module.exports = config;
```

De esta forma estamos indicando que queremos que lo empaquetado se guarde en la ruta `dist` y con el nombre de fichero `app.min.js`.

Si lo que deseamos es dar una salida a una configuración con varios puntos de entrada. Podemos hacerlo de la siguiente manera:

```javascript
// webpack.config.js
const config = { 
    entry: {
        home: './src/home/main.js',
        profile: './src/profile/main.js',
        login: './src/login/main.js'
    },
    output: {
        filename: '[name].min.js',
        path: __dirname + '/dist'
    }
}; 

module.exports = config;
```

Con este sistema, lo que le estamos diciendo a Webpack es que nos guarde los tres ficheros dentro de `dist` con el nombre que hayamos indicado en `entry`. De esta manera, nos guardará tres ficheros con el siguiente nombre: `home.min.js`, `profile.min.js`, `login.min.js`.

El atributo `output` cuenta con bastantes parámetros más que iremos viendo a lo largo de los posts de manera salteada. Si deseas estudiar un poco más cuales son, los tienes en este listado.

### Los loaders

Esta parte de la configuración nos sirve para indicar las transformaciones que queremos hacer a los ficheros. Los loaders se comportan muy parecido a los plugins que existen en gestores de tareas como gulp o grunt.

Los loaders son en sí partes modularizadas de Webpack por lo que cualquier desarrollador que necesite una transformación de sus ficheros, puede crear uno y engancharlo en su configuración.

Por ejemplo, yo quiero indicar en mi configuración de Webpack, que todos los ficheros que tengan la extensión `.ts` tengan que ser transpilados con tsc para que mi código escrito en TypeScript, se convierta en ES5.

Para conseguir esto, lo primero que tengo que hacer, es descargarme el loader preciso, de la siguiente forma:

```
$ npm install ts-loader --save-dev
```

Con este loader, ahora ya podemos hacer lo siguiente:

```javascript
// webpack.config.js 
const config = { 
    entry: './src/main.ts', 
    output: { 
        filename: 'app.min.js', 
        path: __dirname + '/dist' 
    },
    module: {
        rules: [
            {
                test: /\.ts?$/, 
                exclude: /node_modules/, 
                loader: "ts-loader"
            }
        ]
    }
}; 

module.exports = config;
```

Dentro del objeto `module` existe un parámetro llamado `rules`. Este parámetro es un array que nos permite indicar todas las reglas de transformación que queremos que se ejecuten sobre nuestro código.

Lo que tendremos que hacer siempre es indicar la expresión regular del tipo de ficheros al que queremos que el loader haga efecto, en este caso todos aquellos ficheros que terminen en `.ts`.

Si a un fichero le afectan más de una regla porque coincide con la expresión regular, el orden en cómo se encuentren los loaders colocados es importante ya que se ejecutarán por el orden en que se encuentren en el array.

Los loaders pueden tener opciones. Es recomendable que cuando tengas una necesidad, compruebes si ya existe el loader. De ser así, observa la documentación y estudia las opciones que te puede dar.

### Los plugins

Pero no solo de transformaciones vive Webpack. Los loaders tienen bastantes limitaciones. Están muy bien para realizar ciertas transformaciones, pero a veces se quedan cojos en otras tareas que no tienen que ver con el empaquetamiento en sí.

Para esto han venido a ayudarnos los plugins. Son otra forma de extender y añadir funcionalidad extra a Webpack. Por ejemplo, tendremos plugins para incluir trazas entre loader y loader, para saber cómo ha ido la ejecución. Hay plugins para copiar estáticos de unas carpetas a otras, o par inyectar las dependencias en nuestro index.html.  Este será el ejemplo que hagamos.

Lo primero que haremos será incluir la dependencia en nuestro proyecto del plugin:

```
$ npm install html-webpack-plugin --save-dev
```

La forma de hacer uso de este plugin es la siguiente:

```javascript
// webpack.config.js 
const HtmlWebpackPlugin = require('html-webpack-plugin');

const config = { 
    entry: './src/main.ts', 
    output: { 
       filename: 'app.min.js', 
       path: __dirname + '/dist' 
    },
    module: {
        rules: [
            {
                test: /\.ts?$/, 
                exclude: /node_modules/, 
                loader: "ts-loader"
            }
         ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: './src/index.html' })
    ]
}; 

module.exports = config;
```

Existe una sección en nuestro fichero para que los incluyamos llamada `plugins`, obviamente. Esta sección es un array de objetos que hacen cosas. Webpack sabrá cuando ejecutarlos y cuándo hacer uso de ellos.

En nuestro caso hemos instanciado la clase `HtmlWebpackPlugin`. Lo que hace este plugin es generarnos un fichero `index.html` con los paquetes generados referenciados directamente de serie y nos guarda la copia en el `output` que hayamos indicado.

Nosotros le hemos indicado como opción que coja como referencia una plantilla del fichero que se encuentra en `./src/index.html`. Un plugin muy sencillo, muy útil y con un montón más de configuraciones extra.

Y por ahora con lo explicado, podemos trabajar para proyectos que están empezando o que son sencillos de empaquetar, no necesitaremos mucho más.

## Conclusiones

Webpack me parece una herramienta muy potente. Te ofrece todo lo necesario para que con muy poco puedas hacer mucho. Es muy modularizable y fácil de extender. Su forma declarativa hace que incluir una nueva transformación, una nueva funcionalidad sea tan sencillo como copiar y pegar la documentación del desarrollador que lo hizo.

Sin embargo, su mecanismo por medio de configuración me parece personalmente poco intuitivo en ciertas ocasiones. No es difícil aprender y comprender los conceptos básicos, pero cuando se necesita unas configuraciones más complejas, el fichero de configuración llega a ser bastante difícil de manejar. Lo veremos cuando estudiemos los ficheros de construcción que nos genera vue-cli y cómo el potencial que tiene es lastrado por contener una configuración un tanto ilegible.

Esto que digo es una simple objeción personal y no se atiene a ningún hecho técnico. Habrá gente que entienda mejor estos sistemas de configuración declarativos y habrá otros que prefieran sistemas más imperativos como gulp. Como todo, es hacerse a la herramienta y trabajar mucho con ella.

Nos leemos :)