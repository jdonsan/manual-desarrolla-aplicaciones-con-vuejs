# Capítulo 8. Interceptores de navegación entre rutas

Una vez que hemos visto los conceptos básicos de la librería de rutas, es el turno de profundizar en otros conceptos. Dentro de nuestra SPA, nos vendría bien tener algún tipo de mecanismo para saber, en ciertos momentos de una navegación, qué hacer en ciertas situaciones.

La librería de vue-router cuenta en su haber con una funcionalidad para esto llamada Navigation Guards o interceptores de navegación. Estos interceptores son una serie de funciones que nos van a permitir realizar diferentes acciones entre la navegación de una ruta a otra.

Por ejemplo, estos interceptores nos pueden venir bien para hacer ciertas comprobaciones o modificaciones del estado de la aplicación. Nos pueden venir bien para realizar ciertas redirecciones o para abortar una navegación si algo no se encuentra cómo el sistema espera.

vue-router cuenta con varias opciones para incluir estos interceptores que van desde el registro del interceptor de manera global hasta el registro del interceptor de manera local. A lo largo del post vamos a explicar cada uno de ellos y el uso que le podemos dar:

Leer más…

Interceptores globales

`beforeEach`

Dentro de vue-routercontamos con la posibilidad de registrar un interceptor que se ejecutará cada vez que se realice un cambio de ruta. Este interceptor se ejecuta de manera global, es decir, para toda las rutas a las que naveguemos, justamente antes de producirse la navegación.

La forma de registrar un interceptor global es con la siguiente sintaxis:

const router = new VueRouter({ ... });

router.beforeEach((to, from, next) => {
    // ...
});
beforeEach nos inyecta tres parámetros en la función de callback:

to: Es el objeto router con la información de la ruta a la que voy.
from: Es el objeto router con la información de la ruta de la que vengo.
next: Es la función que me permite reanudar la navegación. Es una función que tiene un comportamiento bastante complejo ya que nos permite diferentes datos de entrada. Por ejemplo:
Si la ejecutamos sin parámetro (`next()`), la navegación se reanudará hacia la ruta indicada en `to`.
Si la ejecutamos con una cadena que nos indique otra ruta (`next('/')`), nos redirigirá a la ruta que le hemos indicado.
Si indicamos un valor booleano `false` (`next(false)`), abortará la redirección.
Incluso si yo paso la instancia de un error (`next(new Error()`), la navegación se abortará y se inyectará con esta instancia el callback de la función registrada en `onError`. Como veis, muy completito. Siempre debemos ejecutar esta función para que el flujo continúe.
Puedo incluir dentro de mi aplicación todos los interceptores globales que yo necesite. Por ejemplo, puedo tener esto:

const router = new VueRouter({ ... });

router.beforeEach((to, from, next) => {
    // ...
});

router.beforeEach((to, from, next) => {
    // ...
});
El orden de ejecución es FIFO (El primero en registrarse, es el primero en ejecutarse).

Los interceptores globales, nos puede venir muy bien para comprobar algún estado global de la aplicación. Por ejemplo, puede venirnos muy bien para comprobar si un usuario en particular va a poder tener acceso a una determinada parte de la aplicación o no.

function existToken() {
    return !!localStorage.token;
}

router.beforeEach((to, from, next) => {
    if (to.path != '/login' && existToken()) {
        next();
    } else {
        next('login');
    }
});
El ejemplo es un caso muy simplificado de control de acceso en la parte privada de una aplicación. Lo que hace es comprobar si la aplicación va a navegar a una ruta diferente de login. De ser así, comprueba que tenga un token de sesión con la API, lo que significará que se tiene acceso. Si lo tiene, continúa con la navegación normal. De no ser así, redirige a la vista de login.

Será uno de los interceptores que más usemos.

`afterEach`

Este interceptor se ejecuta después de que todos los interceptores de los componentes se hayan ejecutado. Su sintaxis es esta:

router.afterEach((to, from) => { // ... });
Como podemos apreciar, en este caso no se inyecta la función next por lo que no será posible influir en la navegación.

Es un interceptor que vamos a usar poco y que nos puede venir bien para depurar las rutas de navegación.

Interceptores locales a una ruta

En ocasiones, puede que necesitamos influir en la navegación de una sola ruta y no en cada una de ellas. Cuando necesitamos un uso más específico en una ruta, podemos registrar una función en tiempo de configuración.

Por ejemplo, podemos hacer esto:

const router = new VueRouter({ 
    routes: [ 
        { 
            path: '/login', 
            component: LoginView, 
            beforeEnter: (to, from, next) => { 
                delete localStorage.token;
                next();  
            } 
        } 
    ] 
});
Lo que hacemos es limpiar la sesión del usuario antes de navegar a login. De esta manera nunca se nos olvidará quitar privilegios al usuario si se ha hecho un logout.

Este interceptor puede sernos útil para evitar ir a rutas intermedias en un proceso de compra. Por ejemplo, podemos evitar mostrar  la vista detalle si el usuario no ha seleccionado antes en una vista previa ningún producto.

Interceptores locales a un componente

Podemos localizar todavía más cuándo interceptar la navegación. Podemos incluir interceptores a nivel de un componente. Dependiendo del contexto de navegación en el que se encuentre un componente, podremos hacer unas acciones u otras.

Este mecanismo interno en un componente es muy importante porque la librería vue-router cachea los componentes y evita que ciertos hooks - como los de creación o destrucción - no sean ejecutados siempre. Por tanto, estos interceptores nos podrán ayudar porque tienen acceso a la instancia interna del propio componente.

Si yo quisiera interceptar comportamientos de navegación para un componente, lo haría de la siguiente forma:

const CartSummary = { 
    template: `...`, 
    beforeRouteEnter (to, from, next) { }, 
    beforeRouteUpdate (to, from, next) { }, 
    beforeRouteLeave (to, from, next) { } 
};
Estudiemos cada uno de ellos:

beforeRouteEnter

En este interceptor entramos cuando la navegación ha sido confirmada, pero todavía no se ha creado, ni renderizado el componente. Nos puede venir bien usarlo para saber si el componente tiene algún comportamiento de navegación especial antes de pintarse.

Si nosotros queremos influenciar en el estado de un componente, tenemos que esperar a que sea creado. Para conseguir esto, podemos hacerlo pasándole una función a next() de la siguiente manera:

const CartSummary = { 
    template: `...`, 
    beforeRouteEnter (to, from, next) { 
        next(vm => vm.products = []);
    }
};
Este interceptor, es un buen sitio para iniciar el estado de un componente con datos de un servicio externo. Por ejemplo, voy a traer los productos de mi backend y a cargarlos:

const CartSummary = { 
    template: `...`, 
    beforeRouteEnter (to, from, next) { 
        axios.get('/products', (res) => { 
            next(vm => vm.products = res.data);
        });
    }
};
Como los interceptores detienen la navegación hasta que se ejecuta la función next(), nos vienen bien para gestionar este asincronismo.

beforeRouteUpdate

Se ejecuta cuando la ruta de navegación cambia y el componente está siendo reutilizado para la siguiente ruta o por la misma. Por ejemplo, imaginemos que la ruta ha cambiado en la parte dinámica, en sus parámetros.

Como dijimos, los componentes se encuentran cacheados y no ejecutan sus hooks de creación y destrucción. Como este interceptor tiene acceso a la instancia del componente, puede ser un buen momento para manipular el estado según las necesidades nuevas.

const CartSummary = { 
    data() {
        return {
            products: null,
        };
    },
    template: `...`, 
    beforeRouteUpdate (to, from, next) {
        this.products = [];
    }
};
Sin callbacks, ni artificios. Directamente accediendo a la instancia porque el componente ya se encuentra instanciado. Nada de fontanería.

beforeRouteLeave

Por último, contamos con este interceptor que se ejecuta cuando antes de cambiar de ruta y sabemos que el componente no va a ser utilizado.

Como se ejecuta antes de ir a la nueva navegación, tiene acceso al estado del componente. Es muy utilizado para evitar navegaciones involuntarias y sin querer. Imagínate que el usuario ha dado sin querer a salir de la compra y tiene todo relleno.

Podemos usar ese interceptor para poner un popup e indicar si el usuario está de acuerdo con no guardar los cambios realizados.

const CartSummary = { 
    template: `...`, 
    beforeRouteLeave (to, from, next) {
        this.popup()
            .then(next)
            .catch(() => next(false);
    } 
};
¿Cuál es el flujo de ejecución de estos Guards?

Como parece un poco confuso cuando se ejecuta cada uno de los interceptores. Os pongo una guía del flujo que se sigue:

La navegación es activada.
Se llama a todos los beforeRouterLeave que se hayan registrado y que no van a ser reutilizados en la siguiente ruta a la que voy.
Se llama a todos los interceptores globales beforeEach.
Se llama a todos los beforeRouteUpdate de los componentes que van a ser reutilizados en la siguiente ruta a la que voy.
Se llama a beforeEnter que hemos configurado en la ruta a la que voy.
Se resuelven toda la asincronía de componentes de esa ruta.
Se llama a beforeRouteEnter  de los componentes que van a estar activos.
Se da la navegación como confirmada.
Se llama al interceptor afterEach.
Se llama a los callbacks pasados a next in beforeRouteEnter.
Y vuelta a empezar cuando se lanza una nueva navegación.
Conclusión

Una de las cosas malas de usar frameworks y librerías de terceros es que te tienes que adaptar a lo que ellos entienden por un buen momento para que tu enganches tu funcionalidad. Quizá para muchos estos interceptores cumplan con sus exigencias, quizá para otros esto sea un impedimento.

Lo bueno es que VueJS ha pensado en ello y nos da bastantes opciones para poder redirigir o abortar navegaciones o incluso hacer acciones de borrado, actualización o creación de estados influidos por la navegación que se nos pide.

El problema de los interceptores suele ser el de siempre: el de tener funcionalidad ejecutándose de una manera asíncrona, lo que hace más difícil depurar si no somos metódicos y no tenemos en cuenta cuando incluirlos y cuando no.

Me he encontrado muchas veces interceptores dios que hacen más de lo que deben o que incluso que no respetan la regla de única responsabilidad. Tengamos en cuenta que este uso debería ser un recursos limitado y muy justificado en nuestro contexto.

Nos leemos :)