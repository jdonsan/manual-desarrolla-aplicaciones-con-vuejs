# Capítulo 5. El ciclo de vida de un componente

Todo componente tiene un ciclo de vida con diferentes estados por los que acaba pasando. Muchos de los frameworks y librerías orientados a componentes nos dan la posibilidad  de incluir funciones en los diferentes estados por los que pasa un componente.

En el caso de VueJS existen 4 estados posibles. El framework nos va a permitir incluir acciones antes y después de que un componente se encuentre en un estado determinado. Estas acciones, conocidas como hooks, tienen varios propósitos para el desarrollador:

* Lo primero que nos van a permitir es conocer el mecanismo interno de cómo se crea, actualiza y destruye un componente dentro de nuestro DOM. Esto nos ayuda a entender mejor la herramienta.

* Lo segundo que nos aporta es la posibilidad de incluir trazas en dichas fases, lo que puede ser muy cómodo para aprender al principio cuando somos novatos en una herramienta y no sabemos como se va a comportar el sistema, por tanto, es un buen sistema para trazar componentes.

* Lo tercero, y último, es la posibilidad de incluir acciones que se tienen que dar antes o después de haber llegado a un estado interno del componente.

A lo largo del posts vamos a hacer un resumen de todos los posibles hooks. Explicaremos en qué momento se ejecuta y cómo se encuentra un componente en dicho estado. Por último, pondremos algunos ejemplos para explicar la utilidad de cada uno de ellos:

## Creando el componente

Un componente cuenta con un estado de creación. Este estado se produce entre la instanciación y el montaje del elemento en el DOM. Cuenta con dos hooks. Estos dos hooks son los únicos que pueden interceptarse en renderizado en servidor (dedicaremos una entrada de esto en próximos capítulos), el resto, debido su naturaleza, sólo pueden ser usados en el navegador.

### `beforeCreate`

Este hook se realiza nada más instanciar un componente. Durante este hook no tiene sentido acceder al estado del componente pues todavía no se han registrado los observadores de los datos, ni se han registrado los eventos.

Aunque pueda parecer poco útil, utilizar este hook puede ser es un buen momento para dos acciones en particular:

* Para configurar ciertos parámetros internos u opciones, inherentes a las propias funcionalidad de VueJS. Un caso de uso común es cuando queremos evitar  referencias circulares entre componentes. Cuando usamos una herramienta de empaquetado de componentes, podemos entrar en bucle infinito por culpa de dichas referencias. Para evitar esto, podemos cargar el componente de manera 'diferida' para que el propio empaquetador no se vuelva loco.

```javascript
const component1 = {
    beforeCreate: function () {
        this.$options.components.Component2 = require('./component2.vue');
    }
};
```

* Para iniciar librerías o estados externos. Por ejemplo, imaginemos que queremos iniciar una colección en `localSotrage` para realizar un componente con posibilidad de guardado offline. Podríamos hacer lo siguiente:

```javascript
const component = {
    destroyed: function () {
        localStorage.setItem('tasks', []);
    }
};
```

### `created`

Cuando se ejecuta este hook, el componente acaba de registrar tanto los observadores como los eventos, pero todavía no ha sido ni renderizado ni incluido en el DOM. Por tanto, tenemos que tener en cuenta que dentro de created no podemos acceder a `$el` porque todavía no ha sido montado.

Es uno de los más usados y nos viene muy bien para iniciar variables del estado de manera asíncrona. Por ejemplo, necesitamos que un componente pinte los datos de un servicio determinado. Podríamos hacer algo como esto:

```javascript
const component = {
    created: function () { 
        axios.get('/tasks') 
            .then(response => this.tasks = response.data) 
            .catch(error => this.errors.push(error));
    }
};
```

## Montando el componente

Una vez que el componente se ha creado, podemos entrar en una fase de montaje, es decir que se renderizará e insertará en el DOM. Puede darse el caso que al instanciar un componente no hayamos indicado la opción el. De ser así, el componente se encontraría en estado creado de manera latente hasta que se indique o hasta que ejecutemos el método $mount que lo que provocará es que el componente se renderice, pero no se monte (el montaje sería manual).

### `beforeMount`

Se ejecuta justo antes de insertar el componente en el DOM, justamente, en tiempo de la primera renderización de un componente. Es uno de los hooks que menos usarás y, como muchos otros, se podrá utilizar para trazar el ciclo de vida del componente.

A veces se usa para iniciar variables, pero yo te recomiendo que delegues esto al hook created.

### `mounted`

Es el hook que se ejecuta nada más renderizar e incluir el componente en el DOM. Nos puede ser muy útil para inicializar librerías externas. Imagínate que estás haciendo uso, dentro de un componente de VueJS, de un plugin de jQuery. Puede ser buen momento para ejecutar e iniciarlo en este punto, justamente cuando acabamos de incluirlo al DOM.

Lo usaremos mucho porque es un hook que nos permite manipular el DOM nada más iniciarlo. Un ejemplo sería el siguiente. Dentro de un componente estoy usando el plugin button de jQuery  UI (Imaginemos que es un proyecto legado y no me queda otra). Podríamos hacer esto:

```javascript
const component = {
    mounted: function () {
        $(".selector").button({});
    }
};
```

## Actualizando el componente

Cuando un componente ha sido creado y montado se encuentra a disposición del usuario. Cuando un componente entra en interacción con el usuario pueden darse eventos y cambios de estados. Estos cambios desembocan la necesidad de tener que volver a renderizar e incluir las diferencias provocadas dentro del DOM de nuevo. Es por eso, que el componente entra en un estado de actualización que también cuenta con dos hooks.

### `beforeUpdate`

Es el hook que se desencadena nada más que se provoca un actualización de estado, antes de que se se comience con el re renderizado del Virtual DOM y su posterior 'mapeo' en el DOM real.

Este hook es un buen sitio para trazar cuándo se provocan cambios de estado y se desembocan renderizados que nosotros no preveíamos o que son muy poco intuitivos a simple vista. Podríamos hacer lo siguiente:

```javascript
const component = {
    beforeUpdate: function () {
        console.log('Empieza un nuevo renderizado de component');
    }
};
```

Puedes pensar que es un buen sitio para computar o auto calcular estados a partir de otros, pero esto es desaconsejado. Hay que pensar que estos hooks son todos asíncronos, lo que significa que si su algoritmo interno no acaba, el componente no puede terminar de renderizar de nuevo los resultados. Con lo cual, cuidado con lo que hacemos internamente de ellos. Si necesitamos calcular cómputos, contamos con funcionalidad específica en VueJS por medio de Watchers o Computed properties.

### `updated`

Se ejecuta una vez que el componente ha re renderizado los cambios en el DOM real. Al igual que ocurría con el hook mounted es buen momento para hacer ciertas manipulaciones del DOM externas a VueJS o hacer comprobaciones del estado de las variables en ese momento.

Puede que tengamos que volver a rehacer un componente que tenemos de jQuery, Aquí puede ser buen momento para volver a lanzarlo y hacer un refresh o reinit:

```javascript
const component = {
    updated: function () {
        $(".selector").button("refresh");
    }
};
```

## Destruyendo el componente

Un componente puede ser destruido una vez que ya no es necesario para el usuario. Esta fase se desencadena cuando queremos eliminarlo del DOM y destruir la instancia de memoria.

### `beforeDestroy`

Se produce justamente antes de eliminar la instancia. El componente es totalmente operativo todavía y podemos acceder tanto al estado interno, como a sus propiedades y eventos.

Suele usarse para quitar eventos o escuchadores. Por ejemplo:

```javascript
const component = {
    beforeDestroy() {
        document.removeEventListener('keydown', this.onKeydown);
    }
};
```

### `destroyed`

Tanto los hijos internos, como las directivas, como sus eventos y escuchadores han sido eliminados. Este hook se ejecuta cuando la instancia ha sido eliminada. Nos puede ser muy útil para limpiar estados globales de nuestra aplicación.

Si antes habíamos iniciado el `localStorage` con una colección para dar al componente soporte offline, ahora podríamos limpiar dicha colección:

```javascript
const component = {
    destroyed: function () {
        localStorage.removeItem('tasks');
    }
};
```

## Otros hooks

Existen otros dos hooks que necesitan una explicación aparte.  Dentro de VueJS yo puedo incluir componentes dinámicos en mi DOM. De esta manera, yo puedo determinar, en tiempo de JavaScript, que componente renderizar. Esto lo veíamos en el post anterior y nos puedes ser muy útil a la hora de pintar diferentes vistas de una WebApp.

Pues bien, VueJS no cuenta solo con eso, sino que cuenta con una etiqueta especial llamada keep-alive. Esta etiqueta, en combinación con la etiqueta component, permite cachear componentes que han sido quitados del DOM, pero que sabemos que pueden ser usados en breve.  Este uso hace que tanto las fases de creación, como de destrucción, no se ejecuten por obvias y que de tal modo, se haya tenido que dar una opción.

VueJS nos permite engancharse a dos nuevos métodos cuando este comportamiento ocurre. Son los llamados `actived` y `deactived`, que son usados del mismo modo que el hook created y el hook `beforeDestroy` por los desarrolladores.

## Conclusión

Conocer el ciclo de vida de un componente nos hace conocer mejor VueJS. Nos permite saber cómo funciona todo y en qué orden.

Puede que en muchas ocasiones no tengamos que recurrir a ellos, o puede que en tiempo de depuración tengamos un mixin que trace cada fase para obtener información. Quién sabe. Lo bueno es la posibilidad de contar con ello. Lo bueno es el poder registrarnos en estos métodos y no depender tanto de la magia interna de un framework.

A mi por lo menos, que un framework cuente con estos mecanismos, me suele dar seguridad para llevar productos a producción con ella.

Os dejo el diagrama que resume el ciclo de vida de un componente:

lifecycle.png

Nos leemos :)