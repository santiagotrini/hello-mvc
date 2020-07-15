# Hello MVC

Una aplicación MVC en Express usando el motor de vistas Pug y base de datos con MongoDB.

## Qué vamos a hacer

Vamos a hacer la misma app que hicimos en [hello-fetch](https://github.com/santiagotrini/hello-fetch) pero con otro enfoque. En esta versión haremos un _thin client_ (cliente flaco). Se dice que es flaco porque el navegador el único trabajo que hace es recibir y mostrar el HTML que llega del _backend_ ya todo cocinado. Toda la carga de trabajo recae sobre la aplicación de Express.

En cambio en la otra versión teníamos una API estilo REST y el _backend_ solo tenía que pasar los datos de los usuarios en formato JSON, el resto del trabajo como actualizar la UI lo hacía el cliente con JavaScript, aliviando la carga de trabajo en el servidor.

Los dos enfoques son válidos y tienen sus usos, ventajas y desventajas. Parte del trabajo del equipo del proyecto es decidir qué enfoque conviene en cada situación.

## Modelo Vista Controlador

MVC hace referencia a una forma de estructurar una aplicación, independientemente del lenguaje de programación, _frameworks_ y tecnologías usadas. Si investigan sobre el tema en Internet van a encontrar que en este tipo de cosas los ingenieros y programadores no suelen ponerse de acuerdo. Así que lo que digo a continuación tomenlo como una opinión, lo que yo entiendo que es MVC, otros dirán que eso no es MVC. Esa discusión se la dejo a los académicos, yo les doy la versión de MVC sacada de [este tutorial de MDN](https://developer.mozilla.org/es/docs/Learn/Server-side/Express_Nodejs/Tutorial_local_library_website) pero con una app mucho más modesta.

La idea importante es separar la aplicación en tres capas, una capa más cercana a la base de datos: **los modelos**, una para la interfaz de usuario: **las vistas** y una capa en el medio que interactúa con los modelos para traer información de la base de datos, y lleva esa información a las vistas para mostrarla al usuario: **los controladores**. Como interfaz entre el cliente y los controladores tenemos **las rutas**. Cada una de estas partes de la aplicación tienen su directorio (carpeta) en el proyecto: `models`, `views`, `controllers` y `routes` por sus nombres en inglés. La estructura del directorio del proyecto cuando esté terminado será la siguiente.

```
.
├── controllers
│   └── index.js
├── index.js
├── models
│   └── User.js
├── package.json
├── package-lock.json
├── Procfile
├── README.md
├── routes
│   └── index.js
└── views
    ├── head.pug
    ├── index.pug
    └── layout.pug
```

Lo normal en desarrollo web cuando trabajamos con vistas es usar un _template engine_ (motor de plantillas). Para esta app vamos a usar [Pug](https://pugjs.org) que es un lenguaje y un motor de plantillas soportado por Express. Otras opciones populares son EJS, Handlebars o Hogan.

## Creando el proyecto

En la terminal creamos un proyecto de Node y creamos las carpetas y archivos necesarios.

```
$ mkdir hello-mvc
$ cd hello-mvc
$ npm init -y
$ git init
$ echo node_modules > .gitignore
$ echo web: npm start > Procfile
$ touch index.js
$ mkdir routes models views controllers
$ touch routes/index.js
$ touch controllers/index.js
$ touch models/User.js
$ touch views/layout.pug views/index.pug views/head.pug
$ npm i express mongoose pug
$ npm i -D nodemon
```

Agregamos los scripts de siempre al `package.json`. En `index.js` armamos la app de Express y nos conectamos a la base de datos.

```js
// index.js
const express  = require('express');
const mongoose = require('mongoose');

const port = process.env.PORT        || 3000;
const db   = process.env.MONGODB_URI || 'mongodb://localhost/hellodb';

const app = express();

// conexion a la base de datos
mongoose.set('useUnifiedTopology', true);
mongoose.set('useFindAndModify', false);
mongoose
  .connect(db, { useNewUrlParser: true })
  .then(() => {
    console.log(`DB connected @ ${db}`);
  })
.catch(err => console.error(`Connection error ${err}`));

// listen
app.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

Para decirle a la app que use Pug ponemos debajo de `const app` y antes de la conexión a la base de datos las siguientes líneas.

```js
// set views
app.set('view engine', 'pug');
app.set('views', './views');
```

La primera le dice a la app que motor de plantillas vamos a usar, la segunda le dice en qué directorio buscar las vistas.

## El modelo

El modelo es el archivo `models/User.js` y es exactamente el mismo que en [hello-database](https://github.com/santiagotrini/hello-database/blob/master/models/User.js). Es la interfaz entre la base de datos y la aplicación. Particularmente los métodos `User.find()` y `User.findOne()` que vamos a utilizar. Aunque Mongoose nos permite hacer mucho más que eso, por el momento lo mantenemos bien simple. El modelo va a ser visible solo para el controlador.

## Las rutas

El archivo `routes/index.js` contiene una instancia de `express.Router()` y va montado en el `index.js` del directorio raíz. El código es sencillo, el router lo único que hace es asociar tal ruta con tal función del controlador.

```js
// routes/index.js
const express = require('express');
const controller = require('../controllers/index');
const router = express.Router();

router.get('/', controller.home);

router.get('/search', controller.search)

module.exports = router;
```

Las rutas son sólo dos, una para el _home_ y otra para cuando clickeamos en el botón de buscar. En la segunda línea importamos el controlador donde están definidas las funciones que se ejecutan para cada ruta. Al final exportamos el router para montarlo en `index.js`, un buen lugar es debajo de donde seteamos las vistas.

```js
// router
const router = require('./routes/index');
app.use('/', router);
```

## El controlador

El controlador tiene que definir las dos funciones que usamos en el router y tiene que exportarlas. Además va a hacer uso del modelo, así que lo importamos en la primera línea.

```js
// controllers/index.js
const User = require('../models/User');

exports.home = (req, res) => {
  res.send('Implementar');
};

exports.search = (req, res) => {
  res.send('Implementar')
}
```

En este punto la aplicación responde a las rutas `localhost:3000/` y `localhost:3000/search` aunque con un mensaje de texto. Vamos ahora a definir las vistas para empezar a ver nuestra UI.

## La vista

Si bien tenemos tres archivos dentro de `views` la app tiene una sola vista. Lo separo en tres archivos distintos para mostrar un poco lo que podemos hacer con Pug, pero podríamos tener toda la plantilla en un solo archivo.

El archivo `layout.pug` define la estructura básica del HTML que vamos a enviar al cliente. Acá tenemos un esqueleto básico de una página web, los elementos `html`, `head` y `body`.

```jade
doctype html
html(lang='es')
  head
    include head.pug
  body
    div.container.mt-3
      block content
```

En Pug la indentación es importante, los dos espacios de indentación entre `html` y `head` o `body` indican que esos elementos están anidados. Usen espacios para indentar, yo doy dos espacios de indentación. Los elementos de HTML tienen los nombres habituales, como `div`, `p` o `h1`.

Para indicar los atributos de un elemento lo hacemos dentro de paréntesis como en `html(lang='es')`. Si el atributo es una clase podemos hacerlo con `.` como en `div.container.mt-3` que es equivalente al HTML `<div class="container mt-3"></div>`. Podemos hacer lo mismo con el atributo `id` usando `#`.

La palabra clave `include` en Pug como en `include head.pug` indica que los contenidos del archivo `head.pug` se van a reemplazar debajo del elemento `head`. Por último el `block content` debajo del `div` indica que vamos a usar [herencia](https://pugjs.org/language/inheritance.html) y el archivo `layout.pug` no es más que una plantilla para posiblemente más de una página distinta.

En `head.pug` tenemos las etiquetas requeridas para usar Bootstrap y el título de la página.

```jade
meta(charset='utf-8')
meta(name='viewport' content='width=device-width, initial-scale=1, shrink-to-fit=no')
link(rel='stylesheet', href='https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css')
title Hello MVC
```

El archivo `index.pug` es nuestro contenido principal, la UI va a ser la misma que la otra vez: un _input_ de texto, un botón y una tabla. Para heredar la estructura básica del archivo `layout.pug` usamos `extends layout` en la primer línea y luego especificamos el bloque de contenido.

```jade
extends layout

block content
  h1 Hello MVC
  p Ingresá el ID del usuario que querés buscar.
  hr
  table
    thead
      th id
      th Nombre
      th Email
      th Cumpleaños
      th Edad
    tbody
  hr
  h4 Más info en
    a(href='#') MDN  
```

Podemos ver como quedó, pero primero tenemos que volver al controlador y actualizar la función `home()`. Usamos `res.render()` en Express para responder al cliente con HTML de una vista. Ya podemos ver nuestra interfaz en `http://localhost:3000`.

```js
// controllers/index.js
exports.home = (req, res) => {
  res.render('index');
};
```

Vemos que la tabla necesita algunas clases de Bootstrap.

```jade
table.table.table-striped
  thead.thead-dark
    th id
    th Nombre
    th Email
```

Ahora agregamos el formulario para buscar usuarios por ID, lo hacemos entre el párrafo y el primer `hr`.

```jade
form.form-inline(action='/search' method='get')
  div.form-group.mr-sm-3
    input.form-control(type='text' name='id' placeholder='ID')
  input.form-control.btn.btn-primary(type='submit' value='Buscar')
```

Se trata de un elemento `form` con la clase de Bootstrap `form-inline` y con los atributos `action` y `method`.
Dentro del formulario tenemos un _input_ de tipo texto encerrado en un `div` con clases varias de Bootstrap para el formato. Es importante el atributo `name` del _input_ ya que vamos a usar el valor de ese atributo para recuperar el valor ingresado. Por último usamos un _input_ de tipo `submit` que se muestra como un botón.

Si ponemos un 1 y enviamos el formulario nos envía a la ruta `localhost:3000/search?id=1`.
La ruta `/search` sale del atributo `action` del formulario. Lo que va desde el signo de pregunta al final (`?id=1`) es lo que se conoce como un _query string_. Es una parte especial de la URL que podemos usar en nuestro caso para hacer la _query_ correcta a la base de datos. En Express la _query string_ la podemos encontrar en el objeto `req`, específicamente en `req.query`.

## De nuevo al controlador

Tenemos que implementar las funciones que dejamos pendientes en `controllers/index.js`. Vamos a usar el modelo `User` para realizar las _queries_ a la base de datos. La función que se ejecuta para la primera carga de la página necesita traer todos los usuarios. Podemos usar `User.find()`.

```js
// controllers/index.js
exports.home = (req, res) => {
  User.find().sort('id').exec((err, users) => {
    res.render('index', { users: users });
  });
};
```

Con este código traemos todos los usuarios, los ordenamos por ID y mandamos la vista con un segundo argumento, un objeto que contiene el resultado de la _query_.

Podemos ver el contenido de este objeto en la vista de la siguiente manera. En `index.pug` debajo del primer `hr` y antes de la tabla agreguen:

```jade
pre= users
```

Ahora tenemos que manipular un poco el resultado de la _query_ para poder usarlo en la tabla. La función `home()` quedaría finalmente así, de esta manera le agregamos la propiedad `age` a cada objeto de la colección.

```js
// controllers/index.js
exports.home = (req, res) => {
  User.find().sort('id').exec((err, users) => {
    for (let user of users) {
      user.age = Math.trunc((new Date() - user.birthday) / 31536000000);
    }
    res.render('index', { users: users });
  });
};
```

Para terminar con el controlador implementamos la función que corresponde a la búsqueda por ID.

```js
// controllers/index.js
exports.search = (req, res) => {
  let result = null;
  User.findOne({ id: req.query.id }).exec((err, user) => {
    if (user != null) {
      user.age = Math.trunc((new Date() - user.birthday) / 31536000000);
      result = [user];
    }
    res.render('index', { users: result });
  })
}
```

Esta función se ejecuta para la ruta `/search?id=*` (reemplazar el asterisco por lo que haya en el input del formulario).  Como dijimos antes eso está en `req.query` como la clave es `id` en `req.query.id` tenemos lo que está a la derecha del igual. Usamos `User.findOne()` y si el resultado (`user`) es distinto de `null` entonces calculamos la edad y lo metemos dentro de un array (`result`). Por último enviamos a la vista el resultado en el objeto `{ users: result }`. Si el resultado de la _query_ es `null` porque pusimos algún ID inexistente, entonces la propiedad `users` del objeto que se envía a la vista vale `null`.

Ahora podemos volver a la vista para armar la tabla y terminar la app.

## Condicionales e iteración en Pug

Hay dos posibilidades, o el controlador le pasa datos en `users` a la vista o no le pasa nada (`null`). Podemos mostrar la tabla en función de esto con un `if ... else` en Pug. Entre los dos `hr` de `index.pug`.

```jade
hr
pre= users
if users
  table.table.table-striped
    thead.thead-dark
      th id
      th Nombre
      th Email
      th Cumpleaños
      th Edad
    tbody    
else
  h3 No hay resultados      
hr
```

Que dice básicamente que si `users` no es `null` muestre la tabla y sino el `h3` que dice que no hay resultados.

Ya casi estamos, faltan las filas de la tabla en el `tbody`. Podemos usar `each ... in` para iterar un array en Pug de modo similar a un ciclo for. También eliminamos el elemento `pre` que pusimos antes.

```jade
hr
if users
  table.table.table-striped
    thead.thead-dark
      th id
      th Nombre
      th Email
      th Cumpleaños
      th Edad
    tbody    
      each user in users
        tr
          td= user.id
          td= user.name
          td= user.mail
          td= user.birthday.toLocaleDateString('es-AR')
          td= user.age
else
  h3 No hay resultados      
hr
```

Para cada elemento `user` en el array `users` creamos un `tr` con los cinco `td` usando los valores correspondientes. Y listo, la app está terminada. La pushean a un repo en GitHub, la conectan con Heroku configuran la variable `MONGODB_URI` como hicieron para [hello-database](https://github.com/santiagotrini/hello-database) y listo. Pueden ver el resultado final en [mi app de Heroku](https://hello-pug.herokuapp.com/).

## ¿Y ahora?

Ahora seguimos con [hello-crud](https://github.com/santiagotrini/hello-crud), CRUD es el acrónimo en inglés para Create Read Update Delete, las cuatro operaciones básicas en una base de datos. En esa guía vamos a hacer la API para una app de tomar notas.

Por otro lado también podemos seguir explorando tecnologías que usamos en desarrollo web como WebSockets en [hello-websockets](https://github.com/santiagotrini/hello-websockets) haciendo una sencilla app de chat.

Por último no dejen de ver [hello-postgre](https://github.com/santiagotrini/hello-postgre) si son fanáticos de las bases de datos relacionales y SQL.
