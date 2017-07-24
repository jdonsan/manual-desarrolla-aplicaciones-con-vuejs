# Capítulo 20. Aplicaciones universales con Nuxt

Si has llegado hasta aquí, ¡Enhorabuena! Estamos al final de camino. Hemos aprendido muchas cosas sobre Vue en general y sobre SSR en particular durante estas últimas semanas.

La sensación que nos dejaba vue-server-renderer era la de ser una librería con la que tener que lidiar con demasiada configuración engorrosa, que no nos ayudaba a centrarnos en lo importante: desarrollar aplicaciones.

Como os he ido prometiendo, esto, a día de hoy, tiene solución: el proyecto Nuxt ha nacido para encargarse de toda esa fontanería y ayudarnos en que focalicemos y aportemos valor lo antes posible. Por todo lo probado y leído, Nuxt me parece un proyecto con mucho futuro, por tanto creo que este post será un buen punto y final a la serie de VueJS.

Terminemos lo empezado:

## ¿Qué es?

Es curioso, pero si tengo que explicar qué es Nuxt, lo describiría como un framework de un framework. La idea detrás de Nuxt es coger toda la potencia que tienen VueJS, Webpack y SSR, y permitir crear aplicaciones universales de una forma fácil, rápida y flexible, con menos configuración.

Si recordamos, las aplicaciones universales son aquellas que contienen módulos de código capaces de ser ejecutados tanto en un navegador como en un servidor.  Por tanto, con Nuxt, vamos a poder tener aplicaciones que cumplan con los requisitos que nos habíamos autoimpuesto de SEO y de servir contenido al usuario lo antes posible, sin tener que renunciar a todo el dinamismo en cliente de las SPAs convencionales.

Si has seguido la serie en sus últimos capítulos, comprenderás que Nuxt no resuelve nada nuevo de lo que ya hemos explicado. Sin embargo, lo bueno de este framework es que ya hace todo el trabajo engorroso de configuración y construcción por nosotros.

La idea de Nuxt es permitirnos generar un template de un proyecto base, como hacíamos con vue-cli, pero con una estructura específica para trabajar con todos los elementos de vue - el router, los stores, los componentes... - de una forma más uniforme, más intuitiva y con un sistema basado en convenciones y no tanto en configuraciones.

Por ahora, el proyecto se encuentra en versión alpha, pero las expectativas de la comunidad vue puesta en él son altísimas.

## ¿Qué funcionalidades tiene?

Puede que hasta ahora Nuxt nos pueda parecer confuso, pero las funcionalidades que promete son como para darle una oportunidad. Entre ellas encontramos lo siguiente:

* **Escritura de ficheros vue**: Nada innovador. Una aplicación desarrollada en Nuxt, es una aplicación desarrollada en Vue. Por tanto, el mecanismo de desarrollar nuestros componentes en ficheros de tipo Vue sigue igual. Puede que deseemos migrar un proyecto anterior de Vue a la estructura de Nuxt y la migración será asequible. Al final Nuxt, lo único que sabe es interpretar componentes Vue de una forma más limpia.

* **Separación de código en paquetes de forma automática**: Hemos aprendido a lo largo de la serie cómo usar Webpack y Vue para generar paquetes más pequeños y que se carguen bajo demanda, para disminuir el paquete principal y hacer nuestra aplicación más rápida. En Nuxt no tendremos que hacer nada para activar este sistema. Las vistas ya contienen esta separación en módulos dinámicos y bajo demanda.

* **Renderizado en la parte de servidor**: Sin configuraciones, sin tener que bajar al barro. Todas las vistas de tu aplicación son renderizadas en servidor. Todos los cambios dinámicos son interceptados por Vue en cliente. Nuxt nos hace el SSR transparente.

* **Sistema de rutas y sincronismo de datos avanzado**: Olvídate de configurar vue-router. Nuxt sabe exactamente que rutas generar dependiendo de cómo estructures tus vistas dentro del proyecto. Además, existe funcionalidad extra para la sincronización de datos y componentes.

* **Servir ficheros estáticos**: Tu propio proyecto va a ser capaz de servir estáticos. Ya sea de vistas, imágenes o fuentes. Todo con un servidor integrado por defecto.

* **Transpilación de ES6/ES7, preprocesamiento de SASS/LESS/Stylus y empaquetado/minificado de JS y CSS**: Todo tu Webpack configurado para que no tengamos que preocuparnos de ello, solo de escribir código de negocio.

* **Carga en caliente en desarrollo**: Nuxt configura nuestro proyecto para que en desarrollo se acepte la carga caliente de cambios, con lo que supone en tiempos a la hora de desarrollar.

* **Generación de vistas a formato estático**: la funcionalidad más importante de Nuxt es esta. Poder generar todo un proyecto Vue de manera estática. Renderizar todo en ficheros HTML para que cualquier CDN pueda hospedarlo y sea servido de una forma rápida y optimizada al máximo.

## ¿Cómo empezar?

Nada nuevo: instala NodeJS en tu equipo y ten disponible vue-cli:

```
$ npm install -g vue
```

Nuxt se instala como un nuevo template de Vue de la siguiente manera:

```
$ vue init nuxt/starter <project-name>
```

Nuxt tiene varios templates propios dependiendo de lo que necesites y tu contexto, por lo que te recomiendo que los analices antes de empezar un proyecto. Nosotros con el proyecto base tenemos suficiente.

Esto nos generará una serie de carpetas y ficheros que nos permitirán trabajar en Nuxt. Lo siguientes en hacer es obvio:

```
$ cd <project-name> 
$ npm install
```

Nos descargará todas las dependencias de nuestro proyecto, entre ellas la nueva utilidad de terminal llamada nuxt.

Si queremos ejecutar el proyecto de ejemplo, lanzamos npm run dev y listo.

## ¿Cuál es la estructura de un proyecto Nuxt?

De lo que se nos ha generado podemos aprender mucho. Nuxt me parece tan intuitivo que sin mucho estudio, sabremos qué debemos guardar en cada carpeta. El proyecto será parecido a este:

![Organización de proyecto Nuxt](/images/nuxt/captura-de-pantalla-de-2017-07-11-16-16-05.png)

Expliquemos cada elemento y qué deberemos guardar dentro:

* **assets**: guardaremos todos aquellos ficheros estáticos de los cuales nuestra aplicación depende: imágenes, fuentes, css principales, librerías legadas que no siguen el sistema actual, tienen cabida aquí.

* **components**: todos aquellos componentes visuales, muy core, muy genéricos que pueden ser reutilizados se guardarán en components.

* **layouts**: Componentes de tipo layout que conforman nuestra aplicación. Por lo general, solo se contará con una layout, pero el sistema permite mucha versatilidad en cuanto a layouts y sublayouts se trata.

* **middleware**: son funciones o pequeñas librerías que deben ejecutarse justamente antes del renderizado de páginas (o grupo de páginas) en servidor.

* **pages**: compuesto por todos aquellos componentes de tipo  página. Todas las pantallas de tu aplicación se encontrarán aquí. La forma en cómo organizamos esta carpeta, supondrá la forma en la que se configurará el vue-router, por tanto hay que mimar mucho su organización.

* **plugins**: contiene toda aquella funcionalidad que extiende el funcionamiento de Vue. Por ejemplo, podemos añadir plugins de filtros, o plugins para la internacionalización de la aplicación (i18n).

* **statics**: parecido a assets. Con la diferencia de que aquí se almacenan estáticos que no sean dependencia directa de nuestros componentes o vistas. Los ficheros que se suelen guardar aquí son: el favicon, el robot.txt o el sitemap.xml.

* **store**: todos nuestros módulos de stores se guardan aquí para que Nuxt los ponga a disposición de los componentes.

* **nuxt.config.js**: contiene configuraciones de nuestra aplicación y del funcionamiento de Nuxt. No todos los proyecto son iguales y puede que ciertos procesos tengan que ser diferentes. Puede que el Webpack que usa Nuxt no sea todo lo que necesitemos y que tengamos que extenderlo. Este fichero nos da la versatilidad para que podamos hacerlo.

Como vemos, solo hay un fichero de configuración en toda la aplicación, no hay nada de builds, ni webpacks. No hay routers, no hay componente inicial, ni puntos de entrada diferentes para el paquete de servidor o de cliente. Todo es automático. Es la gran ventaja de Nuxt.

## ¿Cómo funciona?

Una aplicación creada con Nuxt tiene un ciclo de funcionamiento muy específico. Es el siguiente:

![Flujo de Nuxt](/images/nuxt/nuxt-schema.png)

1. Un usuario realiza una petición de una ruta determinada a servidor.

2. El servidor ejecuta la acción `nuxtServerInit` del store principal si la tiene implementada. Esta acción nos permite cargar datos iniciales (prefetching de datos globales).

3. Se ejecutan todos aquellos middlewares que se encuentren en el fichero de configuración `nuxt.config.js` y los relacionados con el layout, la página raíz y las páginas hijas coincidentes que se hayan implementado.

4. Si existe un validador, se ejecuta. Si se resuelve con un true se sigue el proceso, si no se devuelve un 404.

5. Se obtienen aquellos datos de la página para que sean renderizados.

6. Se renderiza en servidor y se sirve al usuario.

7. Si el usuario navega por la aplicación hacia otra ruta, se repite el ciclo.

## Conclusión

No vamos a profundizar más en Nuxt por ahora. Lo dejamos aquí ya que hay mucho que asimilar y mucho de lo que profundizar en el ecosistema Vue. El proceso para llegar aquí será largo para el lector que decida practicar con Vue, pero creo que la recompensa merecerá la pena.

No recomiendo empezar la casa por el tejado, esto es, empezar a desarrollar en Nuxt sin saber bien sobre Vue, Webpack o SSR. Los principios, como en todo, serían fáciles y amigables, pero Nuxt sigue ocultando mucho trabajo internamente, muchos procesos, muchos conceptos que si no se han asimilado anteriormente, pueden explotarnos en la cara en momento en los que Nuxt no nos esté funcionando como debiera.

Si te ves con fuerzas para seguir, Nuxt no aporta muchos más conceptos o conocimientos sobre Vue. Solo es una herramienta para ser más productivos con Vue. Aprender los entresijos de Nuxt nos va a suponer un par de días de estudio y práctica y  quizá lo único en lo que pensaremos es en cómo no lo habíamos descubierto antes.

Nos leemos :)