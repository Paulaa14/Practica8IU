# iu2425

Repositorio para prácticas de IU, edición 2024-25.

Este año implementamos una interfaz de gestión (simulada) de asignación docente para un departamento, con datos reales del departamento ISIA en cuanto a asignaturas y espacios (pero con nombres generados al azar, y la asignación de quién da qué a medias).

## Instalación

* Descarga nodejs / npm 

    + en los laboratorios *ya está instalado*, puedes saltarte este paso
    + en tu ordenador con Windows, recomiendo el [instalador oficial](https://nodejs.org/) para Windows 64-bits
    + en tu ordenador con Linux, recomiendo [nvm](https://github.com/nvm-sh/nvm), aunque hay otras alternativas que seguramente funcionen igual de bien

* Descarga los fuentes a una carpeta de trabajo,

    + en los laboratorios, la primera vez
    ~~~{.ps}
    cd \hlocal
    git clone https://github.com/manuel-freire/iu2425
    cd iu2425
    ~~~

    + en otros sitios puedes variar la carpeta de destino (`hlocal` seguramente no exista), y si ya habías descargado el proyecto antes, mejor usar lo que ya tenías. 

* Lanza el proyecto (desde dentro de la carpeta `iu2425`)

    ~~~
    npm install
    npm run serve
    ~~~

    **Importante: `iu2425/node_modules`** ocupa mucho, y se regenera cada vez que ejecutas `npm install`, y por tanto **nunca** se debe meter en repositorios.

* Abre un navegador para ver tu página, y un editor para modificarla:

    ~~~
    firefox localhost:8080
    code .
    ~~~

    (puede ser más fácil lanzarlos desde fuera de una consola; el `.` abre la carpeta actual, por lo que deberías estar dentro de `iu2425` al ejecutar `code .` )

* Instala la extensión de VS Code para Vue.js. Para ello, navega a cualquier archivo con extensión `.vue` (por ejemplo, `src/App.vue`), y te aparecerá una ventanita que te ofrece instalar `Vue-Official (v2.1.10)` ó superior.

## Contenido del proyecto

Lo importante está en `src/components`. 

Hay 2 ficheros que son librerías de JS puro, y que no tienen dependencias externas:

* `model.js` - contiene el modelo. En una aplicación web de verdad, esto sería una API que haría peticiones AJAX al servidor para acceder al modelo de verdad, que estaría en una base de datos. Aquí, lo generamos todo en JS...
* `util.js` - funciones sencillas de JS (pero que no quieres escribir desde cero), usadas mucho dentro del modelo.

Los demás ficheros `.js` son:
* `main.js` instancia la aplicación web, y se genera por defecto al crear un nuevo proyecto Vue; sólo está cambiado para incorporar **boostrap**. 
* `state.js` contiene el estado global de la aplicación. En una aplicación más grande, usaríamos un gestor de estado dedicado, como [Pinia](https://pinia.vuejs.org/). El objeto `gState` se usa para acceder a la API (que en realidad es `model.js`). El estado de los filtros y de ordenación de las vistas también se guarda aquí.

En el fichero `grupos2425.json` puedes ver los datos de partida de la aplicación. Cada vez que cambies algo en los fuentes, se volverán a mostrar estos datos. Están generados llamando a `gState.model.saveState()`, y guardando lo que devuelve en un fichero.

El resto de ficheros son de tipo `.vue`, y por tanto componentes-de-único-fichero (SFCs, en inglés); aunque sería posible [usar sólo JS puro](https://vuejs.org/guide/introduction.html#api-styles):

* `App.vue`: el el fichero principal, pero se limita a seleccionar estilos generales de tipografía y delega todo lo importante al componente `PmState`. Los componentes son:

* `PmState.vue`: muestra toda la interfaz, estructurada en una barra de navegación, un listado configurable (de usuarios, asignaturas, grupos o espacios), y detalles del elemento seleccionado

* `SortableGrid.vue` colabora con `FilterAddBox.vue` para mostrar listados filtrables y ordenables. También tiene un botón para añadir más elementos del mismo tipo (que lanza el correspondiente modal)

* `DetailsPane.vue`: muestra detalles del elemento seleccionado, y permite editarlo (vía diálogo modal) o eliminarlo. Delega en 
  
  * `UserDetails.vue`: detalles de un usuario
  * `SubjectDetails.vue`: detalles de una asignatura
  * `GroupDetails.vue`: detalles de un grupo
  * `LocationDetails.vue`: detalles de un espacio (aula o laboratorio) en un semestre concreto

* `BaseModal.vue`: un modal de Bootstrap adaptado para Vue. Especializado en 

  * `UserModal.vue`: diálogo modal para crear y modificar usuarios
  * `SubjectModal.vue`: para crear y modificar asignaturas
  * `GroupModal.vue`: para crear y modificar grupos. También permite gestionar sus horarios.

* Los modales usan algunos controles especializados:

  * `TextBox.vue`: una etiqueta y luego un campo de texto. Para escribir texto.
  * `MemberBox.vue`: elección de elementos de una lista.
  * `SelectBox.vue`: permite elegir (a lo sumo) 1 elemento de entre varios.
  * `SlotBox.vue`: para añadir, modificar o eliminar horarios dentro de un grupo

## Nota

Este proyecto es muy mejorable, pero es suficiente para ver cómo funciona Vue, y además se aprende mucho (de Vue, usabilidad y accesibilidad) mejorando código ajeno. ¡Que no te asuste este código!

