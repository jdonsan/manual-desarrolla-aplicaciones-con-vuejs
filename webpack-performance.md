
# Capítulo 16. Caching, Shimming & Splitting

Con lo trabajado hasta ahora podríamos tirar sin problemas. Sin embargo, Webpack presentan un potencial mayor al de solo crear empaquetados. Webpack es una herramienta que se preocupa mucho por la optimización de nuestros estáticos.

Está bastante sensibilizado con aquellos proyectos que necesitan reducir hasta el más mínimo byte para hacer que la aplicación cargue en nuestro navegador lo más rápido posible. No olvidemos que los desarrolladores Web hemos creado un sistema sobre un protocolo (HTTP) que no estaba pensado para este envío masivo de información.

Por tanto, es bueno saber que con una sola herramienta vamos a poder hacer estas pequeñas mejoras que nos distanciarán con el resto de webs en cuanto a tamaño de ficheros y tiempo de carga.

Hoy, para terminar la serie de Webpack, hablaremos de 3 conceptos avanzados muy orientados a mejorar estos aspectos de optimización: el Caching, el Shimming y el Splitting.

Veamos:

## Caching

O acumular. ¿De qué forma puede ayudarnos Webpack a cachear, si los ficheros van a distribuirse a los navegadores de nuestros usuarios? Como sabemos, los navegadores modernos cuentan con sistemas de caché automáticos por defecto.

Esto hace, que si un usuario vuelve a visitar nuestra web, el navegador compruebe si ya tiene copias guardadas de los ficheros que queremos pedir a un servidor determinado.

Esto nos ha provocado ciertos quebraderos de cabeza a los desarrolladores, ya que cuando se han realizado cambios en nuestros empaquetados, a veces los cambios no han podido ser repercutidos en el navegador porque los ficheros cacheados y cambiados se llamaban igual.

La forma que hemos tenido para engañar al navegador ha sido añadiendo una variable en el queryString, que provoque esta actualización por diferencias en la URI del estático.

Por ejemplo, podemos hacer peticiones de los script así:

```
application.js?build=1
application.css?build=1
```

De esta forma, cuando vamos creando una nueva build, aumentaremos el número y el navegador sabrá que es un nuevo fichero. Para hacer esto hemos usado muchas tretas. Pero con Webpack podemos hacerlo por defecto. En Webpack contamos con una serie de patrones para nombrar a los empaquetados de salida. Por ejemplo, podemos hacer esto:

```javascript
// webpack.config.js
const path = require("path");

module.exports = {
  entry: {
    vendor: "./src/vendor.js",
    main: "./src/index.js"
  },
  output: {
    path: path.join(__dirname, "build"),
    filename: "[name].[hash].js"
  }
};
```

Si nos fijamos, en lo que pone en `otuput.filename`, estamos creando un patrón con palabras reservadas que solo Webpack sabe interpretar cuando se encuentran entre corchetes. Lo que estamos diciendo es que a los empaquetados que se genere les ponga el nombre (`[name]`) indicado en el `entry`, que la extensión final sea `.js` y que en el medio lleve un hash.

Cada vez que se ejecuta un proceso de empaquetado por parte de Webpack se genera un hash. Este hash se puede utilizar para indicar la build que acabamos de construir de esta forma.

Si ejecutamos la configuración anterior, obtendremos algo como esto:

```
Hash: 2a6c1fee4b5b0d2c9285
Version: webpack 2.2.0
Time: 62ms
                         Asset     Size  Chunks             Chunk Names
vendor.2a6c1fee4b5b0d2c9285.js  2.58 kB       0  [emitted]  vendor
  main.2a6c1fee4b5b0d2c9285.js  2.57 kB       1  [emitted]  main
   [0] ./src/index.js 63 bytes {1} [built]
   [1] ./src/vendor.js 63 bytes {0} [built]
```

Con esto, cada vez que se haga un cambio en nuestro código, se actualizarán las copias del navegador. Sin embargo, seguimos teniendo un problema. Enlazar todos los assets al mismo hash, provoca que todos los assets tengan que ser actualizados por el navegador.

Esto no es un comportamiento real ya que en muchas ocasiones nuestro código cambiará, pero el código de las dependencias (`vendor`), por ejemplo, no lo hará tan asíduamente. Necesitamos un mecanismo que nos desacople este funcionamiento.

La solución es generar un hash por cada paquete creado. Este hash se genera a partir del contenido del fichero. De esta forma, si el fichero no cambia, el hash sigue siendo el mismo.

Lo que hacemos es, no indicar la palabra reservada hash, y sí la palabra reservada `chunkhash`.

```javascript
module.exports = {
  /*...*/
  output: {
    /*...*/
    filename: "[name].[chunkhash].js"
  }
};
```

De esta manera, si ejecuto mi configuración, observaremos que los hashes no se comparten. Cada fichero es independiente en el sistema de caché.

```
Hash: cfba4af36e2b11ef15db
Version: webpack 2.2.0
Time: 66ms
                         Asset     Size  Chunks             Chunk Names
vendor.50cfb8f89ce2262e5325.js  2.58 kB       0  [emitted]  vendor
  main.70b594fe8b07bcedaa98.js  2.57 kB       1  [emitted]  main
   [0] ./src/index.js 63 bytes {1} [built]
   [1] ./src/vendor.js 63 bytes {0} [built]
```   

## Shimming

O calzar. Webpack sabe trabajar con módulos escritos con ES6, CommonJS o AMD entre otros. Sin embargo, hay librerías de terceros que no se encuentran escritas con estos sistemas de módulo y que, o siguen otro sistema, o tienen incluido todo lo que necesitan para funcionar en el ámbito global.

Webpack nos aporta mecanismos para insertar estas librerías en los paquetes y evitar que se rompa la construcción de nuestro paquete. Al final, lo que Webpack hace es crear una serie de envoltorios sobre estas librerías para que sepan adaptarse bien a nuestras necesidades.

Por ejemplo, aunque Webpack sabe buscar dependencias dentro de node_modules para incluirlas en el paquete, muchas veces, podremos ser nosotros los que indiquemos a por dónde tiene que ir a buscarla. Si yo hago esto:

```javascript
// webpack.config.js

module.exports = {
    ...
    resolve: {
        alias: {
            jquery: "jquery/src/jquery"
        }
    }
};
```

Le estoy diciendo a Webpack que cada vez que encuentre una dependencia global que ponga `jquery`, no vaya a la carpeta node_modules y que en su lugar vaya a la ruta que hemos indicado, `jquery/src/jquery`.

Esto es una buena práctica de optimización. En vez de coger el fichero minificado por defecto, hacemos que Webpack vaya a por el código normal y que sea él el encargado de hacer el proceso.

Otro problema viene cuando una librería está haciendo uso de una variable global de otra librería. Por ejemplo, imaginemos en un plugin desarrollado con jQuery, lógicamente tiene una dependencia de una variable global, pero queremos seguir haciendo uso del sistema de módulos.

Si empaquetamos con Webpack sin hacer nada, cuando usemos la aplicación, nos dirá que esa variable no existe en  módulo correspondiente. Webpack nos proporciona un sistema por el cual el es capaz de inyectarnos los `imports` que necesitamos en cada momento. Si hago esto:

```javascript
module.exports = {
  plugins: [
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    })
  ]
};
```

Cada vez que Webpack en encuentre el uso de $ o jQuery en un módulo, incluirá al principio del mismo un `var $ = require('jquery')`. De esta manera todo funcionará correctamente.

Puede darse el caso también, que una de estas librerías que se encuentran en el ámbito global, hagan uso de `this`. Cuando se usa `this`  dentro de un sistema de módulos como Webpack, el puntero o apunta a `window` sino a `module.exports`. Esto se puede solucionar gracias al loader `import-loader`:

```javascript
module.exports = {
  module: {
    rules: [{
      test: require.resolve("some-module"),
      use: 'imports-loader?this=>window'
    }]
  }
};
```

​Cuando una librería sufra este mal, ejecutaremos este loader para que sus `this` apunten a `window`.

## Splitting

O dividir. Lo más seguro es que no queramos que todo nuestro código de aplicación se encuentre en un único paquete. Por ello, Webpack nos proporciona formas de dividir nuestros empaquetados en diferentes trozos para cumplir con ciertos aspectos claves de la optimización.

Dentro esta opción de división, contamos con 3 tipos:

### Separación del CSS

El primero trata sobre el hecho de poder separar nuestro CSS de nuestro JavaScript. Esta opción ya la comentamos en el post anterior y no vamos a profundizar, pero recuerda que existe un plugin llamado `ExtractTextWebpackPlugin` que nos ayudará en esta posibilidad de aislar el CSS en un empaquetado por separado.

### Separación de librerías externas

El segundo aborda la separación de código propio con el de código de librerías de terceros. Aunque lo hemos ido comentando en la serie, no hemos profundizado en este hecho. Poder separar las librerías externas en un compilado aparte, nos va a ayudar en procesos de optimización.

Si podemos separar este código externo, que por lo general va a cambiar menos que el nuestro propio, podemos hacer que un buen porcentaje de código sea cacheado por los navegadores sin que nosotros tengamos que preocuparnos de él.

Para conseguir esto, lo primero que tenemos que hacer es indicar que queremos que se generen diferentes compilados. Lo hacemos como explicamos en el primer post de Webpack. Indicamos dos puntos de entrada diferentes. Imaginemos que usamos la librería `moment` como librería externa y queremos separarla del paquete principal:

```javascript
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'moment'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        }
    }
}
```

Con esto se generan dos paquetes dentro de `dist` uno para `main` y otro para el `vendor`.

La configuración anterior presenta varios problemas. El primero es que aunque estemos separando `moment` en un paquete aparte dentro de `vendor`, no significa que no se esté incluyendo también en `main`. Estamos duplicando código. El segundo es un problema de flexibilidad. Estamos haciendo que en nuestro paquete de `vendor` solo se incluya la librería `moment`. Esto no es real pues muchas dependencias se deberán incluir aquí.

Para solucionar el primer problema,  usamos el plugin `CommonsChunkPlugin` que lo que hace es extraer todas las dependencias repetidas de los paquetes generados a donde nosotros le digamos. Por lo tanto hacemos esto:

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'moment'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor'
            })
        ]
    }
}
```

Incluimos el plugin e indicamos dónde queremos guardar los módulos comunes. En este caso en `vendor`.

El segundo caso va muy relacionado con el primero ya que lo que vamos a hacer es quitar el punto de entrada de `vendor` y vamos a dejar que el plugin de `CommonsChunkPlugin` se encargue de generar el paquete de forma implícita.

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function() {
    return {
        entry: {
            main: './index.js'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor',
                minChunks: function (module) {
                   return module.context && module.context.indexOf('node_modules') !== -1;
                }
            })
        ]
    };
}
```

Lo que hemos hecho es decirle a Webpack que saque las módulos comunes en el paquete `vendor` de todo lo que se encuentra dentro de la carpeta node_modules. Como todas nuestras dependencias externas se encontrarán en esta carpeta, ya tenemos la separación en un paquete independiente solucionado.

Seguimos teniendo un último problema. Webpack incluye en todos los empaquetados un motor de ejecución para gestionar estos módulos en los paquetes que realiza. Si hemos creado dos paquetes dentro de nuestra aplicación, hemos hecho que Webpack incluya este runtime en cada uno de ellos. Para optimizar esto del todo tenemos una solución: Sacar este motor a un nuevo paquete denominado por Webpack como manifest. Lo hacemos así:

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function() {
    return {
        entry: {
            main: './index.js' //Notice that we do not have an explicit vendor entry here
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor',
                minChunks: function (module) {
                   // this assumes your vendor imports exist in the node_modules directory
                   return module.context && module.context.indexOf('node_modules') !== -1;
                }
            }),
            //CommonChunksPlugin will now extract all the common modules from vendor and main bundles
            new webpack.optimize.CommonsChunkPlugin({
                name: 'manifest' //But since there are no more common modules between them we end up with just the runtime code included in the manifest file
            })
        ]
    };
}
```

Con volver a incluir el plugin para que se vuelva a ejecutar el proceso de separación de porciones comunes e indicarle que el paquete se llamará manifest, Webpack ya sabe que tiene que extraer el runtime en un nuevo fichero.

Con esto tenemos una forma optima de empaquetado de nuestra aplicación.

### Separación bajo demanda o carga perezosa

El último caso que nos queda es la posibilidad de cargar zonas de nuestro código bajo demanda del usuario. Puede que nuestra aplicación sea tan grande y el número de funcionalidades tan profunda, que no merezca la pena hacer que el usuario se descargue todo al principio.

Con Webpack, podemos indicar que se genere un paquete mínimo con lo que más se suele usar y después ir cargando otros módulos cuando sean usados.

La forma de conseguir esto no es por medio de configuración de nuestro Webpack, sino que nos supone un cambio a la hora de que desarrollemos. Dentro de la especificación de JavaScript ya existe un apartado dedicado a la carga de módulos bajo demanda. La forma de usarlo es por medio de la nueva palabra reservada `import()`.

Este método es asíncrono y permite ser usado por medio de promesas y funciones asíncronas. Por ejemplo, si quisiéramos cargar `moment` bajo demanda, podría hacer algo como esto:

```javascript
function determineDate() {
  import('moment').then(function(moment) {
    console.log(moment().format());
  }).catch(function(err) {
    console.log('Failed to load moment', err);
  });
}

determineDate();
```

Webpack entiende perfectamente este código y se encargará de cargar `moment` cuando le hayamos indicado. vue hará mucho uso de esta funcionalidad también, por lo que es importante tenerla clara.

Con funciones asíncronas sería de la siguiente manera:

```javascript
async function determineDate() {
  const moment = await import('moment');
  return moment().format('LLLL');
}

determineDate().then(str => console.log(str));
```

Hay mucha documentación al respecto sobre casos específicos de esta funcionalidad. Por ahora no creo que sea preciso que entremos en más detalle, pero os dejo la documentación por aquí por si alguien lo ve oportuno profundizar.

## Conclusión

Con lo aprendido en el post tenemos un conocimiento mucho más profundo de cómo podemos exprimir Webpack para que nos permita configurar y optimizar los paquetes creados de la mejor manera posible.

Conocer una herramienta como Webpack es difícil sino se llega a probar y trastear. Aunque en la serie hemos podido explicar muchos conceptos, en vuestros proyecto podrán salir nuevos casos de uso que tendréis que mirar y analizar. Como todo proyecto no es igual, ni tiene el mismo nivel de complejidad, solo puedo deciros eso.

Con Webpack va a ser posible hacer casi todo y si no se puede, siempre podréis optar por pedir ayuda a la comunidad o por ampliar la funcionalidad de Webpack.

A lo largo de la serie no he podido hablar de cómo de rápido o lento llega Webpack a trabajar. Tengo constancia de que las primeras versiones tenían unos tiempos de trabajo bastante grandes como para tener la herramienta en cuenta, pero si es cierto que todo lo que llega de la comunidad de Webpack últimamente son buenas noticias en cuanto al rendimiento de las nuevas versiones.

Si algún día empiezo a usar Webpack en producción, os comentaré la jugada, por el momento es todo.

A partir de ahora, volveremos a vue para terminar con la serie. Nos dedicaremos a estudiar cómo podemos renderizar nuestros componentes y vistas en servidor para ganar en tiempos y en posicionamiento para los buscadores. Mucho trabajo todavía :)

Nos leemos :)