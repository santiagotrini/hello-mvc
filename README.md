# Hello MVC

Una aplicación MVC en Express usando el motor de vistas Pug y base de datos con MongoDB.

## Qué vamos a hacer

Vamos a hacer la misma app que hicimos en [hello-fetch](https://github.com/santiagotrini/hello-fetch) pero con otro enfoque. En esta versión haremos un _thin client_ (cliente flaco). Se dice que es flaco porque el navegador el único trabajo que hace es recibir y mostrar el HTML que llega del _backend_ ya todo cocinado. Toda la carga de trabajo recae sobre la aplicación de Express.

En cambio en la otra versión teníamos una API estilo REST y el _backend_ solo tenía que pasar los datos de los usuarios en formato JSON, el resto del trabajo como actualizar la UI lo hacía el cliente con JavaScript, aliviando la carga de trabajo en el servidor.

Los dos enfoques son válidos y tienen sus usos, ventajas y desventajas. Parte del trabajo del equipo del proyecto es decidir qué enfoque conviene en cada situación.

## Modelo Vista Controlador

MVC hace referencia a una forma de estructurar una aplicación, independientemente del lenguaje de programación, _frameworks_ y tecnologías usadas. Si investigan sobre el tema en Internet van a encontrar que en este tipo de cosas los ingenieros y programadores no suelen ponerse de acuerdo. Así que lo que digo a continuación tomenlo como una opinión, lo que yo entiendo que es MVC, otros dirán que eso no es MVC. Esa discusión se la dejo a los académicos de la universidad.

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

## Las vistas

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## El controlador

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
