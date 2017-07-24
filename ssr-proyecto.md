# Capítulo 19. Adaptando tu proyecto a SSR

Con lo visto en el post anterior, tenemos una primera aproximación de cómo dejar configurado nuestro primer proyecto con SSR.  Además de esto, necesitamos adaptar ciertas partes de nuestro código para que todo funcione como debe.

Tenemos que pensar que la forma de trabajar en cliente y servidor no es la misma y conseguir código universal necesita de una serie de adaptaciones.

En el post de hoy hablaremos de cómo y por qué realizar estas adaptaciones. Aunque existirán seguramente más partes que adaptar, nos vamos a centrar en los cores del framewrok: la instancia de vue, los routers y los stores.


## Adaptando la instancia de vue

Monohilo. Recuérdalo. NodeJS es monohilo y esto en muchos casos, nos puede traer problemas. Cuando envías una aplicación vue a un navegador, la instancia que creas es única por usuario.

Como cada usuario tiene su propia copia de vue y de sus datos, y los refrescos continuos de navegador van a hacer que se renueve según la necesidad, no tenemos problemas. Podríamos decir que la arquitectura cliente-servidor con webapp SPA se comporta como una aplicación multihilo, ya que la gestión de memoria se hace en el cliente y no en el servidor.

Ahora bien, si estamos renderizando la instancia principal de vue  en servidor, podemos llegar a tener problemas de compartición de datos entre usuarios por crear una única instancia del objeto. Se puede crear una contaminación de contexto que deberíamos evitar.

Lo normal es que el código del post anterior (ese get enorme que creaba instancias, renderizaba y servia peticiones), lo refactorices y que la instanciación de la aplicación vue la lleves a un módulo separado. De esta manera, el módulo sería universal.

Pero ten en cuenta que cuando NodeJS usa un módulo, lo cachea, haciendo que su funcionamiento se asemeje al de una especie de Singleton. Así que te recomiendo que uses una factoría que instancie el objeto raíz de tu app vue.

Podría ser de esta forma:

```javascript
// app.js
const Vue = require('vue'); 

module.exports = function createApp(context) {
     return new Vue({ 
         data: { url: context.url }, 
         template: `<div>The visited URL is: {{ url }}<div>` 
     });
}
```
```javascript
// server.js 
const express = require('express');
const server = express();
const createApp = require('./app');

server.get('*', (req, res) => { 
     const context = { url: req.url };
     const app = createApp(context);

     renderer.renderToString(app, (err, html) => { 
         // handle error... 
        res.end(html); 
     });
});
```

Con esto, creamos una copia de la aplicación diferente para cada uno de los usuarios que quieren utilizar la aplicación. Este será un patrón que usaremos posteriormente en la instanciación de routers y stores.

## Adaptando el router

Si miramos el manejador de express que escribimos, encontramos que dejamos un get con un asterisco (*). Esto significa que cualquier ruta que enviemos a nuestro servidor, será manejado por este código.

Por tanto, no es mala idea pensar en utilizar nuestro router (vue-router, lógicamente) en la parte servidor para que el sistema también se encargue de renderizar el resto de vistas de nuestra aplicación. Para conseguir esto, tendremos que hacer 3 cambios en nuestro proyecto:

1. La instancia del router (`./src/router/index.js`) tendremos que devolverla con una función factoría.

2. La entrada de servidor (`entry-server.js`) tendrá una pequeña gestión para la posible carga perezosa de componentes vista.

3. En el manejador de la parte servidor (`server.js`)  gestionaremos lo que nos devuelva `entry-server.js`.

Escribamos el código. Primero empecemos con la configuración del router:

```javascript
// ./router 
import Vue from 'vue';
import Router from 'vue-router';

Vue.use(Router);

export function createRouter () { 
    return new Router({ 
        mode: 'history', 
        routes: [ // ... ] 
    });
}
```

Poco nuevo en este código. Se devuelve la instancia en una función factoría y ponemos como modo de enrutado `history` para que se envíen las peticiones a servidor y no sean gestionadas por la parte cliente.

Ahora configuramos el `app.js`:

```javascript
// app.js 
import Vue from 'vue'; 
import App from './App.vue'; 
import { createRouter } from './router';
 
export function createApp () { 
    const router = createRouter() 
    const app = new Vue({ 
        router, 
        render: h => h(App) 
    });

    return { app, router };
}
```

Instanciamos el router y lo incluimos en la instancia de vue. Devolvemos tanto la app como el router porque nos será útil en siguientes pasos.

Detengámonos en `entry-server.js` que tiene más complejidad. Lo comentamos en el código:

```javascript
// entry-server.js 
import { createApp } from './app'; 

export default context => { 
    // Devolvemos una promesa porque puede que nuestras
    // rutas se gestionen de manera asíncrona, así el servidor
    // esperará
    return new Promise((resolve, reject) => { 
        const { app, router } = createApp(); 

        // Registramos la url que nos viene desde cliente
        // para que vue-router decida más tarde si tiene alguna
        // ruta registrada
        router.push(context.url) 

        // Nos quedamos esperando hasta que las rutas se encuentran 
        // listas y cargadas
        router.onReady(() => {
            // Vemos si hay una coincidencia con la ruta enviada 
            const matchedComponents = router.getMatchedComponents();

            // Si no existe, devolvemos un 404
            if (!matchedComponents.length) { 
                return reject({ code: 404 }) 
            } 
 
            // Si existe, indicamos la instancia de vue
            resolve(app); 
        }, reject);
    });
}
```

Lo último que queda es gestionar en `server.js` la decisión de si se ha encontrado una ruta o no. Analizamos en los comentarios de nuevo:

```javascript
// server.js 
const express = require('express');
const server = express();
const createApp = require('/path/to/built-server-bundle.js');

// Escuchamos en todas las rutas (*)
server.get('*', (req, res) => { 
    // Obtenemos la url de la petición
    const context = { url: req.url }; 
    
    // Cuando la promesa sea resuelta
    createApp(context).then(app => {
        // Renderizamos esa ruta
        renderer.renderToString(app, (err, html) => { 
            if (err) { 
                if (err.code === 404) { 
                    res.status(404).end('Page not found'); 
                } else { 
                    res.status(500).end('Internal Server Error');
                } 
            } else { 
                // Devolvemos la vista renderizada
                res.end(html);
            }
        }); 
    }); 
});
```

## Adaptando el store

Si lo pensamos detenidamente, SSR no es otra cosa que la renderización de un 'snapshot' específico del sistema en servidor. Esto significa que si el usuario necesita renderizar datos del usuario, tengamos que tener algún mecanismo para cargarlos. Ya no nos vale solo el sistema de: renderizo en cliente y hago una obtención de datos.

necesitamos algo más sofisticado que nos permita alternar este sistema en cliente, con un sistema de precarga de datos. Lo primero que tenemos que pensar es en obligarnos a incluir vuex. No podemos cargar datos en componentes, pues nos dificulta mucho el trabajo de hacer cargas y renderizados.

Lo ideal es hacer una precarga de datos en el componente vista. Ese componente que se instancia cuando existe una ruta relacionada. Por tanto, vamos a modificar nuestros fichero para que acepten la precarga de datos antes del renderizado.

Lo primero que hacemos es convertir nuestro store en una factoría de la siguiente manera:

```javascript
// ./store/index.js 
import Vue from 'vue'; 
import Vuex from 'vuex'; 
Vue.use(Vuex);

// Como no nos importa la forma en la que obtenemos
// los datos asumimos que tenemos una librería universal
// como axios por ejemplo
import { fetchItem } from './api';

export function createStore () { 
    return new Vuex.Store({ 
        state: { 
            items: {} 
        }, 
        actions: { 
            fetchItem ({ commit }, id) { 
                return fetchItem(id).then(item => { 
                    commit('setItem', { id, item }); 
                }); 
            } 
        }, 
        mutations: { 
            setItem (state, { id, item }) { 
                Vue.set(state.items, id, item); 
            } 
        } 
    }); 
}
```

Lo siguiente es adaptar el fichero `app.js`:

```
// app.js 
import Vue from 'vue';
import App from './App.vue';

import { createRouter } from './router';
import { createStore } from './store';
import { sync } from 'vuex-router-sync';

export function createApp () { 
    const router = createRouter(); 
    const store = createStore();
    
    // sincronizamos router y store comunes
    sync(store, router);
    
    const app = new Vue({ 
        router, store, 
        render: h => h(App) 
    }); 

    // Exponemos todo para su uso posterior
    return { app, router, store };
}
```

Ahora, tenemos que ver en qué componentes de vista se necesita cargar datos antes del renderizado. Para conseguir esto, añadimos un nuevo método al componente llamado asyncData.  

Recuerda que este método será estático. No podrás hacer uso de propiedad de la instancia (todo aquello que este en this), ya que es un método que será ejecutado antes de la instanciación del componente y del renderizado.

Un ejemplo de componente con `asyncData` sería este:

```html
<!-- Item.vue --> 
<template> 
    <div>{{ item.title }}</div>
</template> 

<script> 
export default { 
    asyncData ({ store, route }) { 
       // devolvemos la promesa para gestionarla 
       // en servidor
       return store.dispatch('fetchItem', route.params.id) 
    }, 
    computed: { 
        item () { 
            return this.$store.state.items[this.$route.params.id];
        } 
    } 
} 
</script>
```

Una vez que tenemos esto preparado, tenemos que cambiar las dos posibilidades que tenemos:

### Precarga en servidor

La precarga en servidor, se tiene que hacer antes del renderizado, pero después de que la ruta haya sido relacionada, por tanto, el mejor sitio para hacerlo es en `entry-server.js` que es donde estamos haciendo estos apaños:

```javascript
// entry-server.js 
import { createApp } from './app'; 

export default context => { 
    return new Promise((resolve, reject) => { 
        const { app, router, store } = createApp();
     
        router.push(context.url); 
        router.onReady(() => { 
            const matchedComponents = router.getMatchedComponents();
 
            if (!matchedComponents.length) { 
                return reject({ code: 404 }) 
            } 

            // Ejecutamos los asyncData de todos los componentes
            // vista relacionados
            Promise.all(matchedComponents.map(Component => { 
                if (Component.asyncData) { 
                    return Component.asyncData({ store, route: router.currentRoute }); 
                } 
            })).then(() => {
                // LA CLAVE ESTA AQUÍ, LO EXPLICAMOS MÁS ABAJO 
                context.state = store.state; 
                resolve(app); 
            }).catch(reject); 
        }, reject) 
    }) 
}
```

Lo que hacemos es recorrer todos los componentes vista relacionados y ejecutar `asyncData` si existe en el componente.

Guardamos los estados obtenidos en el contexto. De esta forma, vue-server-renderer sabrá obtener los datos y automáticamente serializará los datos en el HTML para que sean accedidos por el cliente antes del montaje y no se produzca una desinscronización entre DOM real y virtual.

Esta serialización se guarda en `window.__INITIAL_STATE__` por lo que tenemos que tocar un poco el fichero `entry-client.js` para que todo funcione como debe:

```javascript
// entry-client.js 
const { app, router, store } = createApp(); 

if (window.__INITIAL_STATE__) { 
    store.replaceState(window.__INITIAL_STATE__); 
}
```

### Precarga en cliente

Ahora bien, puede que en ciertos escenarios, la carga de datos tenga que darse en cliente. En este contexto tenemos dos aproximaciones:

Que queramos cargar los datos antes del renderizado de la vista. Con lo que haríamos esto:

```javascript
// entry-client.js 

// ...

router.onReady(() => { 
    router.beforeResolve((to, from, next) => { 
        const matched = router.getMatchedComponents(to); 
        const prevMatched = router.getMatchedComponents(from); 
        
        // Hacemos lo siguiente para comprobar si ya se ha hecho
        // renderizado en servidor. De esa manera evitamos una
        // doble precarga
        let diffed = false; 
        const activated = matched.filter((c, i) => { 
            return diffed || (diffed = (prevMatched[i] !== c)); 
        }); 
       
        if (!activated.length) { 
            return next(); 
        } 

        Promise.all(activated.map(c => { 
            if (c.asyncData) { 
                return c.asyncData({ store, route: to }); 
            } 
        })).then(() => { 
            next(); 
        }).catch(next); 
    }); 
  
    app.$mount('#app'); 
});
```

O que la carga de los datos se realice después del renderizado y lo que tengamos que hacer (como posible solución) sea registrar un mixin global que lo gestione:

```javascript
Vue.mixin({ 
    beforeMount () { 
        const { asyncData } = this.$options;
      
        if (asyncData) { 
             this.dataPromise = asyncData({ 
                 store: this.$store, 
                 route: this.$route 
              }); 
        } 
    } 
});
```

Muchos aspectos cubiertos, para según el escenario que necesitemos.

## Conclusión

Puede parecer que incluir SSR en nuestro proyecto se basa en incluir una fontanería que lo que hace es irnos más del foco de solución y del dominio. Yo opino igual. Sin embargo, si decidimos hacer esto por nuestra cuenta o si ya contábamos con un proyecto en CSR que queremos migrar a SSR no tendremos más solución que arremangarnos y empezar a adaptar poco a poco.

Como iremos viendo, existen alternativas un poco más rápidas si empezamos un proyecto con SSR desde cero. Esto lo veremos en el post dedicado a nuxt que veremos a continuación. Por ahora, será mejor que asimilemos estos cambios y que sigamos valorando si el esfuerzo nos va a compensar.

Nos leemos :)