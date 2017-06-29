# Capítulo 15. Configurando nuestra primera build

Los conceptos básicos están bien. Ayudan a asentar el conocimiento que necesitamos para desarrollar cosas más complicadas. Pero sin un caso real donde aplicar esos conocimientos, es difícil aprender de forma consistente.

Como no es lo mismo contarlo que vivirlo, hoy vamos a crear una build que podría pasar por la de un proyecto real  de tamaño pequeño-medio. A lo largo de los diferentes pasos a seguir para configurar nuestra build, iremos descubriendo nuevos conocimientos sobre Webpack que nos salvarán de más de un apuro.

> El proceso que vamos a seguir es uno de los miles que podríamos hacer. Que yo lo haga así, no significa que sea prioritario que se haga así. De hecho, no significa que lo esté haciendo del todo bien. El post es una forma práctica de seguir aprendiendo conceptos, pero el flujo puede varias según vuestras necesidades.

Empecemos:

## Pensando en entornos

Lo primero que haremos será pensar en cómo van a ser los entornos en los que se podrá desplegar nuestra aplicación. Por lo general, se cuenta con 3 entornos en casi todos los proyectos: entorno de desarrollo, entorno de test y entorno de producción.

> Creo que los nombres de los entornos son lo suficientemente explícitos como para no tener que explicarlos.

Para simplificar las cosas, dejemos de lado el entorno de test y centrémonos en montar una build que sirva tanto para desarrollo como para producción. Para que la build escale bien, vamos a mantener dos configuraciones separadas y vamos a extraer todo lo que tengan en común en una tercera configuración. De esta manera, deberemos tener en nuestro proyecto algo parecido a esto:

```
- proyecto-real-build
-- /build
---- webpack.base.conf.js
---- webpack.dev.confg.js
---- webpack.pro.config.js
```

Lo primero que hemos hecho es meter todas las configuraciones en una carpeta llamada `/build`. organizando los ficheros así, no ensuciamos la carpeta raíz. Como sabemos, la carpeta raíz ya que de por sí está demasiado llena de ficheros de configuración genéricos. Separar todo lo relativo a la build mejora la legibilidad.

Lógicamente, si hacemos esto, cuando lancemos el comando `webpack` en línea de comandos no funcionará. Esto se debe a que Webpack no encuentra la configuración por defecto. Para solucionar esto, insertaremos un par de comandos nuevos en nuestro proyecto Node. En el `package.json` pondremos estas dos líneas dentro de scripts:

```javascript
// package.json
{
    ...
    "scripts": {
        "build:dev": "webpack --config ./build/webpack.dev.conf.js",
        "build:pro": "webpack -- config .build/webpack.pro.conf.js"
    }
    ...
}
```

Cuando en mi línea de comandos ejecute `npm run build:dev`, se ejecutará la configuración de desarrollo y cuando se ejecute `npm run build:pro`, la de producción.

Volviendo a nuestra carpeta build,  vemos que los nombres de los ficheros son claros e inequívocos. Todo desarrollador tiene claro sobre dónde va a influir cada uno. El fichero base (`webpack.base.conf.js`) contendrá la parte de la configuración más genérica. Este fichero es como el padre de las dos configuraciones especificas `webpack.dev.config.js` y `webpack.pro.config.js`.

Para que estos ficheros específicos hereden la configuración del base, tenemos que hacer uso un módulo de Webpack llamado `webpack-merge`. Así que lo primero que hacemos es instalar las dependencias que necesitamos en nuestro proyecto.

```
$ npm install webpack webpack-merge --save-dev
```

Teniendo esto, pongamos la configuración básica de nuestra aplicación. Por ejemplo, ya sea desarrollo o producción, la entrada y la salida las tenemos bastantes claras, y la generación del index.html también. Por tanto,  incluimos esta configuración:

```javascript
// ./build/webpack.base.conf.js

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const config = {
    entry: ['./src/main.js'],
    output: {
        filename: 'app.min.js',
        path: path.resolve(__dirname, '..', 'dist'),
        publicPath: '/'
    },
    plugins: [
        new HtmlWebpackPlugin({ template: './src/index.html' })
    ]
};

module.exports = config;
```

Como hicimos en el post anterior, tenemos un punto de entrada en `./src/main.js` y una salida en `./dist/app.min.js`. Incluimos el plugin `HtmlWebpackPlugin`  para que nos inyecte los scripts necesarios para que funcione nuestro `index.html`.

Ahora lo que hacemos es rehusar esa configuración para los dos entornos, de la siguiente manera:

```javascript
// Ponemos lo mismo tanto en ./build/webpack.dev.conf.js 
// como ./build/webpack.pro.conf.js

const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');

const config = merge(webpackBaseConf, {

});

module.exports = config;
```

De esta manera, hemos separado las 2 builds.

## Gestión de Assets

En el post anterior, hablamos mucho sobre cómo empaquetar módulos JS, pero hablamos poco de cómo añadir todos aquellos ficheros que no tienen nada que ver con JS. Ya dijimos que todo lo que tenga enlaces o dependencias es un candidato para ser empaquetado, pero, lógicamente, no todos los ficheros querremos que se empaqueten de la misma manera.

Por ejemplo, habrá equipos que prefieran que el CSS de su proyecto, se añada a la build inyectándolo inline en el HTML, otros, por ejemplo, preferirán que se genere un fichero CSS por separado y que sea enlazado en el HTML. Con las imágenes pasa lo mismo. Habrá equipos que, por rendimiento, las necesitarán optimizadas y transformadas en String Base64 inyectadas en el HTML, y habrá equipos que las necesitará en una carpeta aparte.

Yo voy a usar la forma habitual de ir guardando todo en ficheros por separado. Veamos los diferentes tipos de assets que existen en un proyecto y cómo poder ponerlos donde queramos.

Para explicar el proceso, voy a añadir en src un componente que tenga todo lo necesario para funcionar:

```
- proyecto-real-build
-- /build
-- /src
---- /components
------ /users
-------- data.csv
-------- icon.png
-------- index.js
-------- style.css
---- index.html
---- main.js
```

He decidido desarrollar un componente porque va a demostrar mejor que no importa donde se encuentren nuestros assets. Ya no hace falta que se encuentren en carpetas genéricas. Podemos crear cosas más especificas y Webpack se encargará por nosotros de colocar todo donde debe.

Como vemos en este caso, mi componente hace uso de un ficheros de estilos, de una imagen y de un ficheros de datos. Veamos cómo configuro yo esto para que se meta cada cosa en su sitio:

### Cargando CSS

La idea es que, con loaders, podamos hacer todo. Por ejemplo, cada vez que mi JS o mi HTML haga uso de un fichero CSS especifico, Webpack tendrá que meterlo en la carpeta dist en un fichero separado. Veamos el código del componente:

```javascript
import './style.css';
import Data from './data.csv';

export default {
    render() {
        const items = Data.reduce((html, user) =>`${html}<li>${user.name} ${user.lastname1} ${user.lastname2}</li>`, '');
        return`<ul class="users">${items}</ul>`;
    }
};
```

Nuestro componente importa su CSS específico `(import './style.css'`). No es la mejor forma de hacerlo y en vue nunca importaremos el CSS así, pero para el ejemplo nos vale.

Lo que hacemos ahora es configurar nuestro `webpack.base.conf.js` para decirle qué tiene que hacer con este código. Lo hacemos así:

```javascript
// webpack.base.conf.js

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

const config = {
    ...
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    use: 'css-loader'
                })
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: './src/index.html' }),
        new ExtractTextPlugin('app.min.css')
    ]
};
```

Con esta nueva regla estamos diciéndole que todos los ficheros que terminen en `.css`, queremos que se encuentren extraídos en el fichero que hayamos indicado en el plugin, en este caso `app.min.css`. Lo que le decimos también es que, mientras tanto,  vaya ejecutando el loader `css-loader` que es muy útil para resolver los `@import` de CSS3.

De esta forma, podemos ir escribiendo nuestro CSS en ficheros separados, y Webpack se encargará de ir juntándolos respetando el orden de las dependencias.

### Cargando imágenes

Si observamos el fichero CSS:

```css
// ./src/components/users/style.css

.users {
    list-style: none;
}

.users li:before {
    content: '';
    background: url(./icon.png);
    background-size:cover;
    width: 64px;
    height: 64px;
    display: inline-block;
    vertical-align: middle;
    margin: 1rem;
}
```

Vemos que tiene como dependencia una imagen (`icon.png`). Necesitamos una manera de indicarle a Webpack la forma en la que tiene que tratar a esta imagen. Para ello, incluiremos un nuevo loader que se encarga de mover ficheros a la ruta que decidamos:

```javascript
// webpack.base.conf.js

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

const config = {
    ...
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    use: 'css-loader'
                })
            },
            {
                test: /\.(png|svg|jpg|gif)$/,
                loader: 'file-loader',
                options: {
                   name: 'img/[name].[hash:7].[ext]'
                }
             },
        ]
    },
    ...
};
```

Lo que indico es una nueva regla que se ejecuta cuando, un fichero cargado termine en `.png`, `.svg`, `.jpg`  o `.gif` y se ejecutará el loader `file-loader`. Este loader lo que hace es guardar una copia en `dist`. A su vez, cambia la url del resto de ficheros que lo enlazasen para que apunten a la nueva ubicación.

El loader cuenta con una opción de configuración donde podemos indicar el nombre de  la carpeta donde queremos guardarlo o nombre específico que debe tener: En este caso, le estamos diciendo, que guarde las imágenes en `./dist/img`.

Hemos metido, en el nombre, un hash que Webpack genera automáticamente. De esta forma, el navegador puede saber si la imagen ha cambiado o sigue siendo la misma y por tanto puede mantenerla cacheada. Esto lo explicaremos en el siguiente post dedicado al Caching.

### Cargar ficheros de datos

Otro tipo de ficheros que pueden existir en nuestra aplicación son JSON, CSV o XML. Existen loaders para estos ficheros. Puedo hacer uso de un CSV dentro de mi aplicación. Lo bueno es que estos loaders se encargan de convertir mis datos en objetos JSON, incluyéndoles incluso, una pequeña API para hacer iteraciones sobre los datos.

Parece una tontería, pero la forma en la que se suelen cargar estos ficheros de datos, suele ser por medio de llamadas AJAX y parseos manuales. Si el fichero no es muy grande, Webpack nos lo incrustará en nuestra build con este loader:

```javascript
{
    test: /\.(csv|tsv)$/,
    use: 'dsv-loader'
}
```

Todos los ficheros que hayan sido referenciados con formato `.csv` o `.tsv` serán procesados por el loader `dsv-loader`.

En nuestro programa hacemos caso de un CSV de datos con usuarios de ejemplo. Como comprobarás en el repositorio, no hemos tenido que hacer parseos. Para nosotros se ha convertido en un fichero JS más con el que trabajar.

### Otros assets

Las fuentes o los HTML estáticos, por poner otros ejemplos, también tienen sus loaders. Podremos hacer con estos ficheros procesos parecidas a las antes mencionados. Es nuestra responsabilidad comprobar que loaders y plugins existen de forma oficial para realizar todas estas transformaciones y tareas.

## Resultado final

Con la configuración puesta, hemos conseguido crear esta build que funciona como nosotros deseamos. Dentro de dist obtendremos estos ficheros:

```
- proyecto-real-build
-- /dist
---- /img
------ icon.b8c6544.png
---- app.min.css
---- app.min.js
---- index.html
```

## Configurando Webpack en desarrollo

¿Qué necesidades sueles tener en desarrollo y quizá no sean necesarias en producción? A mi principalmente se me ocurren 3:

* Poder depurar mi código en el navegador
* Poder contar con un servidor ligero para lanzar el resultado
* Poder ver los resultados que he cambiado sin tener que recargar nada

Empecemos por el primero

### Poder depurar mi código en el navegador

La mejor forma de depurar nuestro código es desde el navegador, desde las devTools de Chrome, por ejemplo. El problema que tienen los empaquetadores tiene que ver con que el código a depurar queda poco legible. Como el código empaquetado, por lo general, cuenta con demasiada fontanería, es difícil saber en que lugar se ha producido un error.

Por esto, es importante contar con una herramienta que permita mapear los ficheros que nosotros hemos escrito con código cargado en el navegador. Esto se puede conseguir por medio de los sourcemaps.

En Webpack es muy sencillo añadir un sourcemap a nuestro paquete. Con poner esta configuración, nos valdría:

```javascript
const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');

const config = merge(webpackBaseConf, {
    devtool: 'cheap-eval-source-map'
});

module.exports = config;
```

Con esto, si vamos al Developer Tools, comprobaremos que cuando volvamos a generar el paquete, tendremos esto en sources:

![Imagen de los sourcesmaps descargados](/images/webpack/captura-de-pantalla-de-2017-06-20-13-02-131.png)

Ya podemos poner puntos de parada y depurar nuestro código.

### Poder contar con un servidor ligero para lanzar el resultado

Existen muchas formas de levantar un servidor web en Node. Podemos hacer uso de módulos como `http-server` que nos permitirá hacer peticiones sobre nuestra webapp.

Sin embargo, podemos sacarle mucho más potencial a Webpack gracias a un módulo llamado `webpack-dev-middlerware`. Este módulo permite cargar nuestra build empaquetada dentro de un servidor Web. De esta forma tenemos mucho más margen para configurar cosas por medio de código.

El módulo es muy potente porque nos cargará la build en memoria. Esto significa que cuando estemos en desarrollo no se creará la carpeta `dist` en disco. lo que provocará que la construcción sea más rápida.

Es buen módulo también porque nos puede servir como proxy. Si nuestra aplicación hace llamadas a una API de nuestro servidor, nos permitirá redirigir el tráfico hacia donde digamos. De esta forma, podemos tener desacopladas aplicaciones front con back que en producción se encontrarán dentro del mismo dominio.

Para poner este modulo tendremos que hacer varias cosas. Lo primero es añadir un fichero `dev-server.js` a la carpeta build. Dentro de este fichero incluiremos la configuración de nuestro servidor:

```javascript
const webpack = require("webpack");
const webpackDevMiddleware = require("webpack-dev-middleware")
const webpackDevConf = require('./webpack.dev.conf');
const app = require("express")();

const compiler = webpack(webpackDevConf);

app.use(webpackDevMiddleware(compiler, {
    publicPath: '/',
    quiet: true
}));

app.listen(3000, function () {
    console.log("Listening on port 3000!");
});
```

Lo que estamos haciendo es levantar un servidor con Express en el puerto 3000. Lo que hacemos es compilar nuestra configuración en tiempo ejecución del script. Luego le pasamos la configuración compilada al módulo webpack-dev-middleware. Gracias a esto, conseguimos que nuestro empaquetado final se sirva desde memoria.

Lo bueno de este módulo es que ya pone nuestro proyecto en `mode watch = on`, lo que significa que si hemos cambiado algo de nuestro código, el módulo sabe lanzar de nuevo la compilación.

El único cambio que tenemos que hacer ahora tiene que ver con cómo ejecutamos este build. Volvemos al `package.json`:

```javascript
// packege.json
{
    ...
    "scripts": {
        "build:dev": "node ./build/dev-server.js,
        "build:pro": "webpack --config .build/webpack.pro.conf.js"
    }
    ...
}
```

Cuando lanzo `npm run build:dev`. Se levanta un servidor, se compila el proyecto en memoria, me genera un sourcemap para depurar y encima es sensible a cambios.

### Poder ver los resultados que he cambiado sin tener que recargar nada

Lo último que nos queda es que, ese observador que recompila cada vez que haya un cambio, sepa indicar al navegador que se recargue. Como decimos, hay varias formas de hacer esto pues Webpack cuenta con la funcionalidad de carga de módulos en caliente, sin embargo, ya que estamos haciendo uso de un servidor 'ad hoc' con Express, usaremos otro middleware para conseguir esto.

El módulo que necesitamos es el siguiente: `webpack-hot-middleware`. Este módulo es el encargado de indicarle al navegador que existen cambios a actualizar. Para hacer uso de él, tenemos que seguir los siguientes pasos:

Tenemos que indicar a Webpack que nos habilite toda la funcionalidad de HRM. Para eso, incluimos el plugin `webpack.HotModuleReplacementPlugin` de la siguiente manera:

```javascript
// ./build/webpack.dev.conf.js

const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');

const config = merge(webpackBaseConf, {
    devtool: 'cheap-eval-source-map',
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin()
    ]
});

module.exports = config;
```

Lo siguiente es incluir el módulo como middleware de express para que esté monitorizando y comprobando si hay nuevos cambios o nuevas recompilaciones:

```javascript
// ./build/webpack.dev.conf.js

const webpack = require("webpack");
const webpackDevMiddleware = require("webpack-dev-middleware")
const webpackHotMiddleware = require('webpack-hot-middleware');
const webpackDevConf = require('./webpack.dev.conf');
const app = require("express")();
const compiler = webpack(webpackDevConf);

app.use(webpackDevMiddleware(compiler, {
    publicPath: '/',
    quiet: true
}));

app.use(webpackHotMiddleware(compiler));

app.listen(3000, function () {
    console.log("Listening on port 3000!");
});
```

Este módulo es capaz de actualizar el código en caliente en el navegador, pero no es capaz de hacer un reload del navegador. Para ello, el módulo cuenta con una librería que permite suscribirnos a un evento que nos indica si debemos hacer un reload. Lo hacemos de la siguiente manera:

```javascript
const hotClient = 
   require('webpack-hot-middleware/client?noInfo=true&reload=true');

hotClient.subscribe(function (event) {
    if (event.action ==='reload') {
        window.location.reload()
    }
});
```

Este fichero lo tenemos que incluir en nuestro compilado. Para ello, cambiamos el `entry` de `dev` para que lo tenga en cuenta. Lo incluimos el primero en el array o no funcionará:

```javascript
const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');

webpackBaseConf.entry = ['./build/dev-client'].concat(webpackBaseConf.entry);

const config = merge(webpackBaseConf, {
    devtool: 'cheap-eval-source-map',
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin()
    ]
});

module.exports = config;
```

En esta línea:

```javascript
webpackBaseConf.entry = ['./build/dev-client'].concat(webpackBaseConf.entry);
```

Incluimos este pequeño script que hemos generado.

De esta forma, cuando cambiemos un módulo de nuestro proyecto, el navegador se actualizará sin que nosotros hagamos nada. Un poco de fontanería, pero bueno. Una vez hecho, no nos molestará más.

## Configurando Webpack en producción

Toda la configuración anterior no es innecesaria en nuestro paquete de producción. La fase de empaquetado en producción es una fase de resultados finales óptimos. Es decir, todo lo que se genera, tienen que ser ficheros estáticos que se encuentren en disco y que tengan un tamaño lo más reducido posible, sin que perdamos funcionalidad.

Tareas como la depuración y la carga dinámica dejan de ser importantes y minificar y comprimir pasan a ser unas de las cuestiones prioritarias.

### Optimizando el index.html

En nuestro caso, estamos generando 3 ficheros que estaría bien que optimizaramos. Para hacerlo incluiremos una serie de plugins que harán por nosotros el trabajo. Lo primero que haremos será indicar que el `index.html` se encuentre minificado. Para esto, lo hacemos así:

```javascript
// ./build/webpack.pro.conf.js

const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const config = merge(webpackBaseConf, {
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: {
               removeComments: true,
               collapseWhitespace: true,
               removeAttributeQuotes: true
            }
        })
    ]
};

module.exports = config;
```

Añadimos un nuevo parámetro al plugin `HtmlWebpackPlugin` llamado minify. Nos permite configurar varias cosas. Por poner un ejemplo de todo lo que se puede hacer, he decidido eliminar los comentarios del HTML final, los espacios blancos que no aporten marcado y las dobles comillas de atributos que sean posibles quitarla. Con esto reducimos el tamaño y el funcionamiento sigue siendo el mismo.

### Optimizando el app.min.css

Hacemos algo muy parecido con el CSS. Añadimos un nuevo plugin que permite optimizar estilos. Tan fácil como añadir el plugin `OptimizeCSSPlugin`:

```javascript
// ./build/webpack.pro.conf.js

const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const OptimiceCSSPlugin = require('optimice-css-assets-webpack-plugin);

const config = merge(webpackBaseConf, {
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: {
                removeComments: true,
                collapseWhitespace: true,
                removeAttributeQuotes: true
            }
       })
       new OptimiceCSSPlugin()
   ]
};

module.exports = config;
```

### Optimizando `app.min.js`

Por último, añadimos plugins para optimizar el JS. Es igual que el anterior, pero con los nuevos plugins:

```javascript
const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConf = require('./webpack.base.conf');
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const config = merge(webpackBaseConf, {
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: {
                removeComments: true,
                collapseWhitespace: true,
                removeAttributeQuotes: true
            },
        }),
        new webpack.optimize.UglifyJsPlugin(),
        new OptimizeCSSPlugin()
    ]
});

module.exports = config;
```

`webpack.optimize.UglifyJsPlugin` es un plugin del propio Webpack que sirve para optimizar JS.
 
### Marcar paquete como producción

Es buena práctica que marquemos el paquete como producción. Esto es así porque cuando la variable de entorno se encuentra en producción hace que se desconecten mecanismos que no deberían verse.

Por ejemplo, en vue, cuando el compilado se encuentra en producción, las Developers Tools especiales de vue se desactivan para que nadie pueda depurar las rutas o los flujos. También, muchas librerías usan este mecanismos para esconder trazas de log. Con ello ganamos en seguridad y en velocidad.

Para incluir esto, mete este nuevo plugin de esta manera:

```javascript
// webpack.pro.config.js

new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify('production')
});
```

## Conclusiones

Con todo esto, tenemos mucho avanzado. Nos hemos dejado mucho por el camino. No hemos explicado como incluir un linter que nos evalúe la homogeneidad de nuestro código, ni hemos incluido plugins que ejecuten tests. No hemos añadido plugins para cambiar las cadenas de i18n, ni hemos transpilado nuestros ficheros SASS para convertirlos en CSS.

Sin embargo, creo que lo más difícil está conseguido que es saber cómo empezar y cómo continuar en una herramienta con Webpack. El resto de cosas tendrán que irse incluyendo según necesidad. Puede que en mi proyecto ahora mismo la internacionalización no sea importante, pero dentro de 3 meses sí. Quién sabe.

En el último post de la serie comprobaremos los conocimientos que hemos adquirido, analizando la build que nos genera vue-cli por defecto. Si entendemos un build tan complejo como ese, estaremos preparado para seguir nuestro camino.

Nos leemos :)

> [Os dejo el repositorio por aquí por si queréis ver lo que hemos hecho en este post](https://github.com/jdonsan/elabismo-webpack-primera-build).
