# Capítulo 18. Configurando Webpack para SSR

Una vez que tenemos claros los conceptos básicos, es hora de trabajar para que nuestro proyecto se adapte a las necesidades del SSR. Empezaremos la adaptación configurando la build.

La forma en la que vamos a empaquetar nuestra aplicación variará debido a las necesidades de ejecutar código de vue en servidor y en cliente de forma indistinta. Necesitamos que el código de nuestra aplicación sea universal y para ello vamos a apoyarnos en Webpack para conseguir este tipo de empaquetados.

Más tarde veremos cómo adaptar el resto del proyecto, pero por ahora centrémonos en este punto: Configurar Webpack a esta nueva situación:

## La estructura de ficheros

Mantener un sistema universal puede ayudarnos a adaptarnos mejor a cambios. Contar con Webpack nos va a dar mucha versatilidad a la hora de poder trabajar en un escenario SSR y/o CSR.

No es una mala idea dejar el sistema adaptado para ambos. Un cambio muy pequeño en la configuración de Webpack y la estructura de ficheros, puede aportarnos mucho.

Además, en muchos escenarios un trabajo mixto suele estar más alineado con la realidad. Esto es, un sistema donde rendericemos vistas en servidor, pero que puedan ser hidratadas con contenido dinámico desde cliente. Por tanto, crear aplicaciones universales es obligatorio si nos acercamos a esta aproximación.

Nos gustaría conseguir un flujo de construcción parecido al de la imagen:

![arquitectura y flujo de webpack con SSR](/images/webpack/786a415a-5fee-11e6-9c11-45a2cfdf085c.png)

Una app con componentes universales, que cuente con dos entradas para que Webpack nos genere dos paquetes. Estos paquetes serán usados dependiendo de la necesidad.

Estructuremos nuestra aplicación de esta forma:

```
src 
├── components 
│   ├── Foo.vue 
│   ├── Bar.vue 
│   └── Baz.vue 
├── App.vue 
├── app.js # entrada universal 
├── entry-client.js # solo se ejecuta en navegador 
└── entry-server.js # solo se ejecuta en servidor
```

Lo único que varía de la estructura típica de un proyecto de vue son los tres ficheros del final. Veamos qué forma tienen:

```javascript
// app.js
import Vue from 'vue';
import App from './App.vue'; 

export function createApp () { 
    const app = new Vue({ 
        render: h => h(App) 
    }); 

    return { app }; 
}
```

Nada nuevo. Una factoría que renderiza el componente raíz de la app. Lo bueno es que es universal. Este código - pasado por Webpack - funciona tanto en NodeJS como en navegador.

Fijaos que la instancia de Vue no se pasa directamente al módulo. Se usa una factoría para que se instancie un contexto para cada usuario. De esa manera no contaminaremos las respuestas de otros usuarios y cada uno tendrá una copia de vue propia al renderizar.
La clave está en los dos ficheros `entry-`. En ellos se ven las particularidades de cada sistema.

```javascript
// entry-server.js
import { createApp } from './app';

export default context => { 
    const { app } = createApp();
    return app 
}
```

Si nos encontramos en el servidor, solo necesitamos la instancia a renderizar. Nada más.

```javascript
// entry-client.js
import { createApp } from './app';

const { app } = createApp();
app.$mount('#app');
```

En cambio, si nos encontramos en el cliente y alguien hace uso de esta paquete, se monta la instancia en el DOM.

### Hidratación desde cliente

Un pensamiento que te puede venir a la cabeza con este sistema mixto es el siguiente:

Si ya he renderizado mi vista en servidor y mi `index.html` obtiene el empaquetado de cliente (`entry-client.js`), ¿vue intentará crear la instancia de nuevo y volver a montar el componente que ya está renderizado desde servidor?

Lo bueno es que en el equipo de vue ya ha pensado en ello y el sistema no produce un conflicto entre el SSR y el CSR. Cuando renderizamos en servidor, la librería vue-server-renderer añade esta etiqueta:

```html
<div id="app" data-server-rendered="true">
```

Esta etiqueta provoca, que el paquete de cliente no monte su instancia, ni se vuelva a crear todo el DOM de nuevo. Vue se pone en modo hidratación.

Cuando se ejecuta `app.$mount('#app');`, simplemente se crea la estructura del Virtual DOM en memoria. Hace esto para poder hacer todos los cambios dinámicos que sean precisos por interacción del usuario. De esta manera, evitamos el conflicto: el servidor renderiza vistas y el cliente hace cambios dinámicos.

## La build

Lo siguiente es cambiar los ficheros de configuración para que acepte estas dos entradas y cumpla con el dibujo que hemos puesto anteriormente.

Para simplificar el ejemplo, vamos a separar la build en tres ficheros: el fichero base, que contiene toda la configuración común del proyecto, la configuración cliente, que contiene toda la configuración para el paquete de cliente, y la configuración de servidor que contiene toda la configuración para el paquete de renderizado en servidor.

De la configuración base, nos vamos a olvidar, pues en nuestro caso solo indica donde se encontrarán las salidas. Como suele ser normal, guardaremos todos los resultados en `/dist`. También incluiremos loaders comunes con el vue-loader y el babel-loader.

Para la parte de cliente, contaremos con una configuración como esta:

```javascript
// webpack.client.config.js
const webpack = require('webpack');
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base.config.js');
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin');

module.exports = merge(baseConfig, { 
    entry: './src/entry-client.js', 
    plugins: [  
        new webpack.optimize.CommonsChunkPlugin({ 
             name: "manifest", 
             minChunks: Infinity 
        }),  
        new VueSSRClientPlugin() 
    ] 
});
```

Ponemos como entrada el fichero `entry-client` que hemos creado y que se encarga de montar la aplicación en el navegador. Después, añadimos dos plugins:

El primer plugin se encarga de extraer el core de Webpack llamado manifest se encarga de las tareas de inyección de descubrimiento de dependencias. De esta manera, como iremos dividiendo nuestro aplicación en módulos que se cargarán de manera dinámica, solo se incluirá una vez este motor en lo más alto de la cadena de dependencias.

El segundo plugin es el propio de la librería vue-server-renderer y se encarga de generar un fichero en formato JSON llamado `vue-ssr-client-manifest.json` que contiene metainformación del paquete de cliente y que nos será útil a la hora de servir peticiones por medio de la funcionalidad Bundle Renderer que explicaremos en un momento.

Con esto tendríamos el paquete de cliente.

La configuración de servidor es la siguiente:

```javascript
const merge = require('webpack-merge');
const nodeExternals = require('webpack-node-externals');
const baseConfig = require('./webpack.base.config.js');
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin');

module.exports = merge(baseConfig, { 
     entry: './src/entry-server.js', 
     target: 'node', 
     devtool: 'source-map', 
     output: { libraryTarget: 'commonjs2' }, 
     externals: nodeExternals({ 
        whitelist: /\.css$/ 
     }), 
     plugins: [ new VueSSRServerPlugin() ] 
});
```

En esta configuración tenemos más cosas. Lo primero, lo de siempre, indicar el fichero de entrada que creamos específicamente. Seguidamente, indicamos que el contexto de ejecución será para NodeJS. Esto hace que los loaders se optimicen y orienten para un funcionamiento en servidor.

El atributo `devtool` lo indicamos con `source-map`. De esta manera, se nos generará un sourcemap del paquete de servidor. Indicamos también que el sistema de módulos a usarse debería ser CommonJS para que sea utilizable desde NodeJS. El atributo `externals` nos ayuda a quitar todo aquello superfluo para el empaquetado de servidor. Por ejemplo, el CSS no será necesario procesarlo en servidor, por tanto, evitamos su procesamiento y aligeramos los tiempo.

El ultimo atributo es usar el plugin de vue-server-renderer pero para la parte de servidor. Lo que nos crea es un JSON llamado `vue-ssr-server-bundle.json` con la metainformación del paquete que estamos creando, pero para procesos del renderizado en servidor. Este fichero será interpretado por la funcionalidad Bundle Renderer que explicamos a continuación.

## El Bundle Renderer

Como hemos visto en el apartado anterior, Webpack y los plugins de vue-renderer-server nos van a añadir en `/dist` dos JSON que contienen metainformación de los paquetes generados para cliente y servidor. Es una forma de desacoplar los paquetes creados y su uso en cliente y servidor. Esto tiene su sentido:

Hay algo molesto al desarrollar para NodeJS y es que cada vez que se realiza un cambio, tenemos que parar el servidor y volver a lanzarlo. Existen procesos para que no sea así cuando trabajamos para desarrollos de NodeJS específicos, pero cuando estamos desarrollando una aplicación vue, el proceso tiene algo de fricción.

La gente de vue ya ha pensado en ello y ha creado una funcionalidad denominada Bundle Renderer. Esta funcionalidad se encarga de escuchar en los JSON que hemos creado con Webpack y renderizar a partir de los cambios, sin tener que parar e iniciar servidor.

De esta manera, podemos desarrollar nuestra webapp, ejecutar webpack y generar los nuevos JSON. El servidor encargado del renderizado ya sabrá que tiene que renderizar con el nuevo paquete generado porque los cambios han sido reflejados en el JSON.

Lo mismo ocurre para el paquete de cliente. La funcionalidad tiene sistema de carga en caliente (HRM) por lo que si existe un cambio, el paquete de cliente sabe cambiarlo en tiempo de ejecución.

Para usar esta funcionalidad, deberemos cambiar nuestro servidor a algo parecido a esto:

```javascript
// ./index.js
const express = require('express');
const server = express();
const template = require('fs').readFileSync(__dirname + '/index.html', 'utf-8');
const serverBundle = require('./dist/vue-ssr-server-bundle.json');
cont clientManifest = require('./dist/vue-ssr-client-manifest.json');
const { createBundleRenderer } = require('vue-server-renderer');

const renderer = createBundleRenderer(serverBundle, { 
    template,  
    clientManifest 
}); 

server.get('*', (req, res) => { 
    const context = { url: req.url } 

    renderer.renderToString(context, (err, html) => { 
        // se quita el manejo de error para simplificar... 
        res.end(html) 
    });
});
```

Lo único que cambia es cómo generamos el renderer. Tenemos que indicarle que lo genere a partir de los dos JSON.

## Conclusión

Con esto, tenemos un buen porcentaje de nuestro proyecto adaptado. Como veis, el proceso de renderizado en servidor, supone una fe ciega hacia un sistema poco intuitivo y que hace cosas como por arte de magia.

Un sistema como estos nos esta suponiendo un nivel de fontanería elevado. No hemos desarrollado nada de nuestro negocio y todo lo que hacemos es configurar y configurar para que todo vaya de manera óptima. Las opciones de vue-server-renderer + webpack pueden llegar a abrumar, así que, si no nos queda más remedio que hacer uso de un sistema como este, tendremos que seguir practicando y estudiando para asimilar los conceptos.

En el próximo posts, nos centraremos en adaptar el router y el store de la aplicación.

Nos leemos :)