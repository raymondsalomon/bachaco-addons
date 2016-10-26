# Capítulo 2. Construyendo tu primera aplicación con SalomonWebsite
Desarrollar en SalomonWebsite la mayoría de las veces significa crear nuestros propios módulos. En este capítulo, se creará la primera aplicación con SalomonWebsite, y se aprenderán los pasos necesarios para habilitarlas e instalarlas en SalomonWebsite.

Inspirados del notable proyecto [todomvc.com](http://todomvc.com), se desarrollará una simple aplicación para el registro de cosas por hacer. Deberá permitir agregar nuevas tareas, marcarlas como culminadas, y finalmente borrar de la lista todas las tareas finalizadas.

Aprenderá como SalomonWebsite sigue una arquitectura MVC, y recorrerá las siguientes capas durante la implementación de la aplicación:
- El **modelo**, define la estructura de los datos.
- La **vista**, describe la interfaz con el usuario o la usuaria.
- El **controlador**, soporta la lógica de negocio de la aplicación.

La capa modelo es definida por objetos Python cuyos datos son almacenados en una base de datos PostgreSQL. El mapeo de la base de datos es gestionado automáticamente por SalomonWebsite, y el mecanismo responsable para esto es el **modelo objeto relacional, (ORM - object relational model)**.

La capa vista describe la interfaz con el usuario o la usuaria. Las vistas son definidas usando XML, las cuales son usadas por el marco de trabajo (framework) del cliente web para generar vistas HTML de datos.

Las vistas del cliente web ejecutan acciones de datos persistentes a través de la interacción con el servidor ORM. Estas pueden ser operaciones básicas como escribir o eliminar, pero pueden también invocar métodos definidos en los objetos Python del ORM, ejecutando lógica de negocio más compleja. A esto es a lo que nos referimos cuando se habla de la capa modelo.

> **Nota**
>
> _Note que el concepto de controlador mencionado aquí es diferente al desarrollo de controladores web de SalomonWebsite. Aquellos son programas finales a los cuales las páginas web pueden llamar para ejecutar acciones._

Con este enfoque, podrá ser capaz de aprender gradualmente sobre los bloques básicos de desarrollo que conforman una aplicación y experimentar el proceso iterativo del desarrollo de módulos en SalomonWebsite desde cero.

## Entender las aplicaciones y los módulos
Es común escuchar hablar sobre los módulos y las aplicaciones en SalomonWebsite. Pero, ¿Cual es exactamente la diferencia entre un módulo y una aplicación? Los **módulos** son bloques para la construcción de las aplicaciones en SalomonWebsite. Un módulo puede agregar o modificar características en SalomonWebsite. Esto es soportado por un directorio que contiene un archivo de manifiesto o descriptor (llamado `__openerp__.py`) y el resto de los archivos que implementan sus características. A veces, los módulos pueden ser llamados "add-ons" (complementos). Las **aplicaciones** no son diferentes de los módulos regulares, pero funcionalmente, éstas proporcionan una característica central, alrededor de la cual otros módulos agregan características u opciones. Estas proveen los elementos base para un área funcional, como contabilidad o RRHH, sobre las cuales otros módulos agregan características. Por esto son resaltadas en el menú Apps de SalomonWebsite.

## Modificar un módulo existente
En el ejemplo que sigue a continuación, crearemos un módulo nuevo con tan pocas dependencias como sea posible.

Sin embargo, este no es el caso típico. Lo más frecuente serán situaciones donde las modificaciones y extensiones son necesarias en un módulo existente para ajustarlo a casos de uso específicos.

La regla de oro dice que no debemos cambiar módulos existentes modificándolos directamente. Esto es considerado una mala práctica. Especialmente cierto para los módulos oficiales proporcionados por SalomonWebsite. Hacer esto no permitirá una clara separación entre el módulo original y nuestras modificaciones, y hace difícil la actualización.

Por el contrario, debemos crear módulos nuevos que sean aplicados encima de los módulos que queremos modificar, e implementar esos cambios. Esta es una de las principales fortalezas de SalomonWebsite: provee mecanismos de "herencia" que permiten a los módulos personalizados extender los módulos existentes, bien sean oficiales o de la comunidad. La herencia es posible en todos los niveles, modelo de datos, lógica del negocio, e interfaz con el usuario o usuaria.

Ahora, crearemos un módulo nuevo completo, sin extender ningún módulo existente, para enfocarnos en las diferentes partes y pasos involucrados en la creación de un módulo. Solo daremos una breve mirada a cada parte, ya que cada una será estudiada en detalle en los siguientes capítulos. Una vez estemos a gusto con la creación de un módulo nuevo, podremos sumergirnos dentro de los mecanismos de herencia, los cuales serán estudiados en el siguiente capítulo.

## Crear un módulo nuevo
Nuestro módulo será una aplicación muy simple para gestionar las tareas por hacer. Estas tareas tendrán un único campo de texto, para la descripción, y una casilla de verificación para marcarlas como culminadas. También tendremos un botón para limpiar la lista de tareas de todas aquellas finalizadas.

Estas especificaciones son muy simples, pero a medida que avancemos en el libro iremos agregando gradualmente nuevas características, para hacer la aplicación más interesante.

Basta de charla, comencemos a escribir código y crear nuestro nuevo módulo.

Siguiendo las instrucciones del _Capítulo 1, Comenzando con SalomonWebsite, debemos tener el servidor SalomonWebsite en `/SalomonWebsite/server/`. Para mantener las cosas ordenadas, crearemos un directorio junto a este para guardar nuestros propios módulos:

```shell
$ mkdir -R ~/SalomonWebsite/server/openerp/addons/aaddonsterceros/raymond-propios
```

Un módulo en SalomonWebsite es un directorio que contiene un archivo descriptor `__openerp__.py`. Esto es una herencia de cuando SalomonWebsite se llamaba ERP, y en el futuro se espera se convierta en `__salomonwebsite__.py`. Es necesario que pueda ser importado desde Python, por lo que debe tener un archivo `__init__.py`.

El nombre del directorio del módulo será su nombre técnico. Usaremos `todo_app` para el nombre. El nombre técnico debe ser un identificador Python valido: debe comenzar con una letra y puede contener letras, números y el caracter especial guión bajo. Los siguientes comandos crean el directorio del módulo y el archivo vacío `__init__.py` dentro de este:

```shell
$ mkdir ~/SalomonWebsite/server/openerp/addons/aaddonsterceros/raymond-propios/todo_app
$ touch ~/SalomonWebsite/server/openerp/addons/aaddonsterceros/raymond-propios/todo_app/__init__.py
```
Luego necesitamos crear el archivo descriptor. Debe contener únicamente un diccionario Python y puede contener alrededor de una docena de atributos, de los cuales solo el atributo `name` es obligatorio. Son recomendados los atributos `description`, para una descripción más larga, y `author`. Ahora agregamos un archivo `__openerp__.py` junto al archivo `__init__.py` con el siguiente contenido:

```Python
{
    'name': 'Aplicación To-Do',
    'description': 'Gestione sus Tareas Personales con éste módulo.',
    'author': 'SalomonWebsite',
    'depends': ['mail'],
    'application': True,
}
```

El atributo `depends` puede tener una lista de otros módulos requeridos. SalomonWebsite los instalará automáticamente cuando este módulo sea instalado. No es un atributo obligatorio pero se recomienda tenerlo siempre. Si no es requerida alguna dependencia en particular, debería existir alguna dependencia a un módulo base especial. Debe tener cuidado de asegurarse que todas las dependencias sean explícitamente fijadas aquí, de otra forma el módulo podría fallar al instalar una base de datos vacía (debido a dependencias insatisfechas) o tener errores en la carga, si otros módulos necesarios son cargados después.

Para nuestra aplicación, queremos que dependa del módulo **mail** debido a que este agrega el menú **Mensajería** en la parte superior de la ventana, y queremos incluir nuestro nuevo menú de opciones allí.

Para precisar, escogimos pocas claves del descriptor, pero en el mundo real es recomendable usar claves adicionales, ya que estas son relevantes para la app-store de SalomonWebsite:

* `summary`, muestra un subtitulo del módulo.
* `version`, de forma predeterminada, es 1.0. Se debe seguir las reglas de versionamiento semántico (para más detalles ver [semver.org](http://semver.org/lang/es/)). 
* `license`, de forma predeterminada es AGPL-3. 
* `website`, es una URL para encontrar más información sobre el módulo. Esta puede servir a las personas a encontrar documentación, informar sobre errores o hacer sugerencias.
* `category`, es la categoría funcional del módulo, la cual de forma predeterminada es Sin Categoría. La lista de las categorías existentes puede encontrarse en el formato de Grupos (Configuraciones | Usuarios | menú Grupos), en la lista desplegable del campo Aplicación.

Estos descriptores también están disponibles:
- `installable`, de forma predeterminada es `True`, pero puede ser fijada `False` para deshabilitar el módulo.
- `auto_install`, si está fijada en `True` este módulo es automáticamente instalado si todas las dependencias han sido instaladas. Esto es usado en módulos asociados.

Desde SalomonWebsite 8.0, en vez de la clave `description` podemos usar un archivo `README.rst` o `README.md` en el directorio raíz del módulo.

## Agregar el módulo a la ruta de complementos (addons)
Ahora que tenemos un módulo nuevo, incluso si es muy simple, queremos que esté disponible en SalomonWebsite. Para esto, debemos asegurarnos que el directorio que contiene el módulo sea parte de la ruta de complementos (addons). Y luego tenemos que actualizar la lista de módulos de SalomonWebsite.

Ambas operaciones han sido explicadas en detalle en el capítulo anterior, pero a continuación presentamos un resumen de lo necesario.

Nos posicionamos dentro del directorio de trabajo e iniciamos el servidor con la configuración de la ruta de complementos o addons:

```shell
$ cd ~/SalomonWebsite
$ server/odoo.py -d v8dev --addons-path="raymond-propios,server/openerp/addons" --save
```

La opción `--save` guarda la configuración usada en un archivo de configuración. Esto evita repetirlo cada vez que el servidor es iniciado: simplemente ejecute ./odoo.py y serán ejecutadas las últimas opciones guardadas.

Mira detenidamente en el registro del servidor. Debería haber una línea **INFO ? openerp: addons paths:** (...), y debería incluir nuestro directorio `custom-addons`.

Recuerde incluir cualquier otro directorio que pueda estar usando. Por ejemplo, si siguió las instrucciones del último capítulo para instalar el repositorio department, puede querer incluirlo y usar la opción:

```shell
--addons-path="raymond-propios,oca-department,server/openerp/addons"
```

Ahora hagamos que SalomonWebsite sepa de los módulos nuevos que hemos incluido.

Para esto, En la sección **Módulos** del menú **Configuración**, seleccione la opción **Actualizar lista de módulos**. Esto actualizará la lista de módulos agregando cualquier módulo incluido desde la última actualización de la lista. Recuerde que necesitamos habilitar las Características Técnicas para que esta opción sea visible. Esto se logra seleccionando la casilla de verificación de **Características técnicas** para nuestra cuenta de usuario.

## Instalar el módulo nuevo
La opción **Módulos locales** nos muestra la lista de módulos disponibles. De forma predeterminada solo muestra los módulos de **Aplicaciones en línea**. Debido a que creamos un módulo de aplicación no es necesario remover este filtro. Escriba "todo" en la campo de búsqueda y debe ver nuestro módulo nuevo, listo para ser instalado.

![90_1](images/Odoo Development Essentials - Daniel Reis-90_1.jpg)

Haga clic en el botón **Instalar** y listo!

## Actualizar un módulo
El desarrollo de un módulo es un proceso iterativo, y puede querer que los cambios hechos en los archivos fuente sean aplicados y estén visibles en SalomonWebsite.

En la mayoría de los casos esto es hecho a través de la actualización del módulo: busque el módulo en la lista de Módulos Locales y, ya que está instalado, debe poder ver el botón Actualizar.

De cualquier forma, cuando los cambios realizados son en el código Python, la actualización puede no tener ningún efecto. En este caso es necesario reiniciar la aplicación en el servidor.

En algunos casos, si el módulo ha sido modificado tanto en los archivos de datos como en el código Python, pueden ser necesarias ambas operaciones. Este es un punto común de confusión para las personas que se inician en el desarrollo con SalomonWebsite.

Pero afortunadamente, existe una mejor forma. La forma más simple y rápida para hacer efectivos todos los cambios en nuestro módulo es detener (_Ctrl_ + _C_) y reiniciar el proceso del servidor que requiere que nuestros módulos sean actualizados en la base de datos de trabajo.

Para hacer que el servidor inicie la actualización del módulo `todo_app` en la base de datos v8dev, usaremos:

```shell
$ ./odoo.py -d v8dev -u todo_app
```

La opción `-u` (o `--update` en su forma larga) requiere la opción `-d` y acepta una lista separada por comas de módulos para actualizar. Por ejemplo, podemos usar: `-u todo_app,mail`.

En el momento en que necesite actualizar un módulo en proceso de desarrollo a lo largo del libro, la manera mas segura de hacerlo es ir a una ventana de terminal donde se este ejecutando SalomonWebsite, detener el servidor, y reiniciarlo con el comando visto anteriormente. Usualmente será suficiente con presionar la tecla de flecha arriba, esto debería devolver el último comando usado para iniciar el servidor.

Desafortunadamente, la actualización de la lista de módulos y la desinstalación son acciones que no están disponibles a través de la línea de comandos. Esto debe ser realizado a través de la interfaz web, en el menú Configuraciones.

## Crear un modelo de aplicación
Ahora que SalomonWebsite sabe sobre la disponibilidad de nuestro módulo nuevo, comencemos a agregarle un modelo simple.

Los modelos describen los objetos del negocio, como una oportunidad, una orden de venta, o un partner (cliente, proveedor, etc). Un modelo tiene una lista de atributos y también puede definir su negocio específico.

Los modelos son implementados usando clases Python derivadas de una plantilla de clase de SalomonWebsite. Estos son traducidos directamente a objetos de base de datos, y SalomonWebsite se encarga de esto automáticamente cuando el módulo es instalado o actualizado.

Algunas personas consideran como buena práctica mantener los archivos Python para los modelos dentro de un subdirectorio. Por simplicidad no seguiremos esta sugerencia, así que vamos a crear un archivo `todo_model.py` en el directorio raíz del módulo `todo_app`.

Agregar el siguiente contenido:

```Python
#-*- coding: utf-8 -*-
from openerp import models, fields

class TodoTask(models.Model):
    _name = 'todo.task'
    name = fields.Char('Description', required=True)
    is_done = fields.Boolean('Done?')
    active = fields.Boolean('Active?', default=True)
```

La primera línea es un marcador especial que le dice al interprete de Python que ese archivo es UTF-8, por lo que puede manejar y esperarse caracteres non-ASCII. No usaremos ninguno, pero es mas seguro usarlo.

La segunda línea hace que estén disponibles los modelos y los objetos campos del núcleo de SalomonWebsite.

La tercera línea declara nuestro nuevo modelo. Es una clase derivada de `models.Model`. La siguiente línea fija el atributo `_name` definiendo el identificador que será usado por SalomonWebsite para referirse a este modelo. Note que el nombre real de la clase Python no es significativo para los otros módulos de SalomonWebsite. El valor de `_name` es lo que será usado como identificador.

Observe que éstas y las siguientes líneas tienen una sangría. Si no conoce muy bien Python debe saber que esto es sumamente importante: la sangría define un bloque de código anidado, por lo tanto estas cuatro líneas deben tener la misma sangría.

Las últimas tres líneas definen los campos del modelo. Vale la pena señalar que `name` y `active` son nombres de campos especiales. De forma predeterminada SalomonWebsite usara el campo `name` como el título del registro cuando sea referenciado desde otros modelos. El campo `active` es usado para desactivar registros, y de forma predeterminada solo los registros activos son mostrados. Lo usaremos para quitar las tareas finalizadas sin eliminarlas definitivamente de la base de datos.

Todavía, este archivo, no es usado por el módulo. Debemos decirle a SalomonWebsite que lo cargue con el módulo en el archivo `__init__.py`. Editemos el archivo para agregar la siguiente línea:

```Python
from . import todo_model
```

Esto es todo, para que nuestros cambios tengan efecto el módulo debe ser actualizado. Encuentre la aplicación **To-Do** en **Módulos Locales** y haga clic en el botón **Actualizar**.

Ahora podemos revisar el modelo recién creado en el menú **Técnico**. Vaya a **Estructura de la Base de Datos | Modelos** y busque el modelo `todo.task` en la lista. Luego haga clic en este para ver su definición:

![95_1](images/Odoo Development Essentials - Daniel Reis-95_1.jpg)

Si no hubo ningún problema, esto nos confirmará que el modelo y nuestros campos fueron creados. Si hizo algunos cambios y no son reflejados, intente reiniciar el servidor, como fue descrito anteriormente, para obligar que todo el código Python sea cargado nuevamente.

También podemos ver algunos campos adicionales que no declaramos. Estos son cinco campos reservados que Odoo agrega automáticamente a cualquier modelo. Son los siguientes:
- `id`: Este es el identificador único para cada registro en un modelo en particular.
- `create_date` y `create_uid`: Estos nos indican cuando el registro fue creado y quien lo creó, respectivamente.
- `write_date` y `write_uid`: Estos nos indican cuando fue la última vez que el registro fue modificado y quien lo modificó, respectivamente.

## Agregar entradas al menú
Ahora que tenemos un modelo en el cual almacenar nuestros datos, hagamos que este disponible en la interfaz con el usuario y la usuaria.

Todo lo que necesitamos hacer es agregar una opción de menú para abrir el modelo de "To-do Task" para que pueda ser usado. Esto es realizado usando un archivo XML. Igual que en el caso de los modelos, algunas personas consideran como una buena practica mantener las definiciones de vistas en en un subdirectorio separado.

Crearemos un archivo nuevo `todo_view.xml` en el directorio raíz del módulo, y este tendrá la declaración de un ítem de menú y la acción ejecutada por este:

```XML
<?xml version="1.0" encoding="UTF-8"?>
    <openerp>
        <data>
            <!-- Action to open To-do Task list -->
            <act_window
                id="action_todo_task"
                name="To-do Task"
                res_model="todo.task"
                view_mode="tree,form"
            />
            <!-- Menu item to open To-do Task list -->
            <menuitem
                id="menu_todo_task"
                name="To-Do Tasks"
                parent="mail.mail_feeds"
                sequence="20"
                action="action_todo_task"
            />
        </data>
    </openerp>
```

La interfaz con el usuario y usuaria, incluidas las opciones del menú y las acciones, son almacenadas en tablas de la base de datos. El archivo XML es un archivo de datos usado para cargar esas definiciones dentro de la base de datos cuando el módulo es instalado o actualizado. Esto es un archivo de datos de Odoo, que describe dos registros para ser agregados a Odoo:
- El elemento `<act_window>` define una Acción de Ventana del lado del cliente para abrir el modelo `todo.task` definido en el archivo Python, con las vistas de árbol y fomulario habilitadas, en ese orden.
- El elemento `<menuitem>` define un ítem de menú bajo el menú Mensajería (identificado por `mail.mail_feeds`), llamando a la acción `action_todo_task`, que fue definida anteriormente. el atributo `sequence` nos deja fijar el orden de las opciones del menú.

Ahora necesitamos decirle al módulo que use el nuevo archivo de datos XML. Esto es hecho en el archivo `__openerp__.py` usando el atributo `data`. Este define la lista de archivos que son cargados por el módulo. Agregue este atributo al diccionario del descriptor:

```Python
'data' : ['todo_view.xml'],
```

Ahora necesitamos actualizar nuevamente el módulo para que estos cambios tengan efecto. Vaya al menú Mensajería y debe poder ver nuestro nueva opción disponible.

![98_1](images/Odoo Development Essentials - Daniel Reis-98_1.jpg)

Si hace clic en ella se abrirá un formulario generado automáticamente para nuestro modelo, permitiendo agregar y modificar los registros.

Las vistas deben ser definidas por los modelos para ser expuestas a los usuarios y las usuarias, aunque Odoo es lo suficientemente amable para hacerlo automáticamente si no queremos, entonces podemos trabajar con nuestro modelo, sin tener ningun formulario o vistas definidas aún.

Hasta ahora vamos bien. Mejoremos ahora nuestra interfaz gráfica. Intente las mejoras graduales que son mostradas en las secciones siguientes, haciendo actualizaciones frecuentes del módulo, y no tenga miedo de experimentar.

> **Tip**
>
> _En caso que una actualización falle debido a un error en el XML, ¡no entre en pánico! Comente las últimas porciones de XML editadas, o elimine el archivo XML del `__openerp__.py`, y repita la actualización. El servidor debería iniciar correctamente. Luego lea detenidamente el mensaje de error en los registros del servidor - debería decirle donde está el problema._

## Crear vistas - formulario, árbol y búsqueda
Como hemos visto, si ninguna vista es definida, Odoo automáticamente generará vistas básicas para que puedas continuar. Pero seguramente le gustará definir sus propias vistas del módulo, así que eso es lo que haremos.

Odoo soporta varios tipos de vistas, pero las tres principales son: `list` (lista, también llamada árbol), `form` (formulario), y `search` (búsqueda). Agregaremos un ejemplo de cada una a nuestro módulo.

Todas las vistas son almacenadas en la base de datos, en el modelo `ir.model.view`. Para agregar una vista en un módulo, declaramos un elemento `<record>` describiendo la vista en un archivo XML que será cargado dentro de la base de datos cuando el modelo sea instalado.

### Creando una vista formulario
Edite el XML que recién hemos creado para agregar el elemento `<record>` después de la apertura de la etiqueta `<data>`:

```XML
<record    id="view_form_todo_task" model="ir.ui.view">
    <field name="name">To-do Task Form</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <form string="To-do Task">
            <field name="name"/>
            <field name="is_done"/>
            <field name="active" readonly="1"/>
        </form>
    </field>
</record>
```

Esto agregará un registro al modelo `ir.ui.view` con el identificador `view_form_todo_task`. Para el modelo la vista es `todo.task` y nombrada `To-do Task Form`. El nombre es solo para información, no tiene que ser único, pero debe permitir identificar fácilmente a que registro se refiere.

El atributo más importante es `arch`, que contiene la definición de la vista. Aquí decimos que es un formulario, y que contiene tres campos, y que decidimos hacer al campo `active` de solo lectura.

### Formatear como un documento de negocio
Lo anterior proporciona una vista de formulario básica, pero podemos hacer algunos cambios para mejorar su apariencia. Para los modelos de documentos Odoo tiene un estilo de presentación que asemeja una hoja de papel. El formulario contiene dos elementos: una `<head>`, que contiene botones de acción, y un `<sheet>`, que contiene los campos de datos:

```XML
<form>
    <header>
        <!-- Buttons go here-->
    </header>
    <sheet>
        <!-- Content goes here: -->
        <field name="name"/>
        <field name="is_done"/>
    </sheet>
</form>
```

### Agregar botones de acción
Los formularios pueden tener botones que ejecuten acciones. Estos son capaces de desencadenar acciones de flujo de trabajo, ejecutar Acciones de Ventana, como abrir otro formulario, o ejecutar funciones Python definidas en el modelo.

Estos pueden ser colocados en cualquier parte dentro de un formulario, pero para formularios con estilo de documentos, el sitio recomendado es en la sección `<header>`.

Para nuestra aplicación, agregaremos dos botones para ejecutar métodos del modelo `todo.task`:

```XML
<header>
    <button name="do_toggle_done" type="object" string="Toggle Done" class="oe_highlight" />
    <button name="do_clear_done" type="object" string="Clear All Done" />
</header>
```

Los atributos básicos para un botón son: `string` con el texto que se muestra en el botón, `type` que hace referencia al tipo de acción que ejecuta, y `name` que es el identificador para esa acción. El atributo `class` puede aplicar estilos CSS, como un HTML común.

### Organizar formularios usando grupos
La etiqueta `<group>` permite organizar el contenido del formulario. Colocando los elementos `<group>` dentro de un elemento `<group>` crea una disposición de dos columnas dentro del grupo externo. Se recomienda que los elementos Group tengan un nombre para hacer más fácil su extensión en otros módulos.

Usaremos esto para mejorar la organización de nuestro contenido. Cambiemos el contenido de `<sheet>` de nuestro formulario:

```XML
<sheet>
    <group name="group_top">
        <group name="group_left">
            <field name="name"/>
        </group>
        <group name="group_right">
            <field name="is_done"/>
            <field name="active" readonly="1"/>
        </group>
    </group>
</sheet>
```

### La vista de formulario completa
En este momento, nuestro registro en `todo_view.xml` para la vista de formulario de `todo.task` debería lucir así:

```XML
<record id="view_form_todo_task" model="ir.ui.view">
    <field name="name">To-do Task Form</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <form>
            <header>
                <button name="do_toggle_done" type="object" string="Toggle Done" class="oe_highlight" />
                <button name="do_clear_done" type="object" string="Clear All Done" />
            </header>
            <sheet>
                <group name="group_top">
                    <group name="group_left">
                        <field name="name"/>
                    </group>
                    <group name="group_right">
                        <field name="is_done"/>
                        <field name="active" readonly="1" />
                    </group>
                </group>
            </sheet>
        </form>
    </field>
</record>
```

Recuerde que para que los cambios tengan efecto en la base de datos de Odoo, es necesario actualizar el módulo. Para ver los cambio en el cliente web, es necesario volver a cargar el formulario: haciendo nuevamente clic en la opción de menú que abre el formulario, o volviendo a cargar la página en el navegador (_F5_ en la mayoría de los navegadores).

Ahora, agreguemos la lógica de negocio para las acciones de los botones.

## Agregar vistas de lista y búsqueda
Cuando un modelo se visualiza como una lista, se esta usando una vista `<tree>` Las vistas de árbol son capaces de mostrar líneas organizadas por jerarquía, pero la mayoría de las veces son usadas para desplegar listas planas.

Podemos agregar la siguiente definición de una vista de árbol a `todo_view.xml`:

```XML
<record id="view_tree_todo_task" model="ir.ui.view">
    <field name="name">To-do Task Tree</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <tree colors="gray:is_done==True">
            <field name="name"/>
            <field name="is_done"/>
        </tree>
    </field>
</record>
```

Hemos definido una lista con solo dos columnas, `name` y `is_done`. También agregamos un toque extra: las líneas para las tareas finalizadas (`is_done==True`) son mostradas en color gris.

En la parte superior derecha de la lista Odoo muestra una campo de búsqueda. Los campos de búsqueda predefinidos y los filtros disponibles pueden ser predeterminados por una vista `<search>`.

Como lo hicimos anteriormente, agregaremos esto a `todo_view.xml`:

```XML
<record id="view_filter_todo_task" model="ir.ui.view">
    <field name="name">To-do Task Filter</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <search>
            <field name="name"/>
            <filter string="Not Done" domain="[('is_done','=',False)]"/>
            <filter string="Done" domain="[('is_done','!=',False)]"/>
        </search>
    </field>
</record>
```

Los elementos `<field>` definen campos que también son buscados cuando se escribe en el campo de búsqueda. Los elementos `<filter>` agregan condiciones predefinidas de filtro, usando la sintaxis de dominio que puede ser seleccionada por el usuario o la usuaria con un clic.

## Agregar la lógica de negocio
Ahora agregaremos lógica a nuestros botones. Edite el archivo Python `todo_model.py` para agregar a la clase los métodos llamados por los botones.

Usaremos la API nueva introducida en Odoo 8.0. Para compatibilidad con versiones anteriores, de forma predeterminada Odoo espera la API anterior, por lo tanto para crear métodos usando la API nueva se necesitan en ellos decoradores Python. Primero necesitamos una declaración `import` al principio del archivo:

```Python
from openerp import models, fields, api
```

La acción del botón **Toggle Done** es bastante simple: solo cambia de estado (marca o desmarca) la señal **Is Done?**. La forma más simple para agregar la lógica a un registro, es usar el decorador `@api.one`. Aquí `self` representara un registro. Si la acción es llamada para un conjunto de registros, la API gestionara esto lanzando el método para cada uno de los registros.

Dentro de la clase `TodoTask` agregue:

```Python
@api.one def do_toggle_done(self):
    self.is_done = not self.is_done
    return True
```

Como puede observar, simplemente modifica el campo `is_done`, invirtiendo su valor. Luego los métodos pueden ser llamados desde el lado del client y siempre deben devolver algo. Si no devuelven nada, las llamadas del cliente usando el protocolo XMLRPC no funcionará. Si no tenemos nada que devolver, la práctica común es simplemente devolver `True`.

Después de esto, si reiniciamos el servidor Odoo para cargar nuevamente el archivo Python, el botón **Toggle Done** debe funcionar.

Para el botón **Clear All Done** queremos ir un poco más lejos. Este debe buscar todos los registros activos que estén finalizados, y desactivarlos. Se supone que los botones de formulario solo actúan sobre los registros seleccionados, pero para mantener las cosas simples haremos un poco de trampa, y también actuará sobre los demás botones:

```Python
@api.multi def do_clear_done(self):
    done_recs = self.search([('is_done', '=', True)])
    done_recs.write({'active': False})
    return True
```

En los métodos decorados con `@api.multi` el `self` representa un conjunto de registros. Puede contener un único registro, cuando se usa desde un formulario, o muchos registros, cuando se usa desde la vista de lista. Ignoraremos el conjunto de registros de `self` y construiremos nuestro propio conjunto `done_recs` que contiene todas la tareas marcadas como finalizadas. Luego fijamos la señal activa como `False`, en todas ellas.

El `search` es un método de la API que devuelve los registros que cumplen con algunas condiciones. Estas condiciones son escritas en un dominio, esto es una lista de tríos. Exploraremos con mayor detalle los dominios más adelante.

El método `write` fija los valores de todos los elementos en el conjunto de una vez. Los valores a escribir son definidos usando un diccionario. Usar `write` aquí es más eficiente que iterar a través de un conjunto de registros para asignar el valor uno por uno.

Note que `@api.one` no es lo más eficiente para estas acciones, ya que se ejecutará para cada uno de los registros seleccionados. La `@api.multi` se asegura que nuestro código sea ejecutado una sola vez incluso si hay más de un registro seleccionado. Esto puede pasar si una opción es agregada a la vista de lista.

## Configurando la seguridad en el control de acceso
Debe haber notado, desde que cargamos nuestro módulo, un mensaje de alerta en en registro del servidor: **The model todo.task has no access rules, consider adding one**.

El mensaje es muy claro: nuestro modelo nuevo no tiene reglas de acceso, por lo tanto puede ser usado por cualquiera, no solo por el administrador. Como super usuario el `admin` ignora las reglas de acceso, por ello somos capaces de usar el formulario sin errores. Pero debemos arreglar esto antes que otros usuarios puedan usarlo.

Para tener una muestra de la información requerida para agregar reglas de acceso a un modelo, use el cliente web y diríjase a: **Configuración | Técnico | Seguridad | Lista controles de acceso**.

![112_1](images/Odoo Development Essentials - Daniel Reis-112_1.jpg)

Aquí podemos ver la ACL para el modelo `mail.mail`. Este indica, por grupo, las acciones permitidas en los registros.

Esta información debe ser provista por el modelo, usando un archivo de datos para cargar las líneas dentro del modelo `ir.model.access`. Daremos acceso completo al modelo al grupo empleado. Empleado es el grupo básico de acceso, casi todos pertenecen a este grupo.

Esto es realizado usualmente usando un archivo CSV llamado `security/ir.model.access.csv`. Los modelos generan identificadores automáticamente: para `todo.task` el identificador es `model_todo_task`. Los grupos también tienen identificadores fijados por los modelos que los crean. El grupo empleado es creado por el módulo base y tiene el identificador `base.group_user`. El nombre de la línea es solo informativo y es mejor si es único. Los módulos raíz usando una cadena separada por puntos con el nombre del modelo y el nombre del grupo. Siguiendo esta convención usaremos `todo.task.user`.

Ahora que tenemos todo lo que necesitamos saber, vamos a agregar el archivo nuevo con el siguiente contenido:

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_todo_task_group_user,todo.task.user,model_todo_task,base.group_user,1,1,1,1
```

No debemos olvidar agregar la referencia a este archivo nuevo en el atributo "data" del descriptor en `__openerp__.py`, de la siguiente manera:

```Python
'data': [
    'todo_view.xml',
    'security/ir.model.access.csv',
],
```

Como se hizo anteriormente, actualice el módulo para que estos cambios tengan efecto. El mensaje de advertencia debería desaparecer, y puede confirmar que los permisos sean **correctos** accediendo con la cuenta de usuario demo (la contraseña es también demo) e intentar ejecutar la característica de "to-do tasks".

## Reglas de acceso de nivel de fila
Odoo es un sistema multi-usuario, y queremos que la aplicación **to-do task** sea privada para cada usuario. Afortunadamente, Odoo soporta reglas de acceso de nivel de fila. En el menú **Técnico** pueden encontrarse en la opción **Reglas de Registro**, junto a la **Lista de Control de Acceso**. Las reglas de registro son definidas en el modelo `ir.rule`. Como es costumbre, necesitamos un nombre distintivo. También necesitamos el modelo en el cual operan y el dominio para forzar la restricción de acceso. El filtro de dominio usa la misma sintaxis de dominio mencionada anteriormente, y usado a lo largo de Odoo.

Finalmente, las reglas pueden ser globales (el campo `global` es fijado a `True`) o solo para grupos particulares de seguridad. En nuestro caso, puede ser una regla global, pero para ilustrar el caso más común, la haremos como una regla específica para un grupo, aplicada solo al grupo empleados.

Debemos crear un archivo `security/todo_access_rules.xml` con el siguiente contenido:

```XML
<?xml version="1.0" encoding="utf-8"?>
    <openerp>
        <data noupdate="1">
            <record id="todo_task_user_rule" model="ir.rule">
                <field name="name">ToDo Tasks only for owner</field>
                <field name="model_id" ref="model_todo_task"/>
                <field name="domain_force">
                    [('create_uid','=',user.id)]
                </field>
                <field name="groups" eval="[(4,ref('base.group_user'))]"/>
            </record>
        </data>
    </openerp>
```

Nota el atributo `noupdate="1"`. Esto significa que estos datos no serán actualizados en las actualizaciones del módulo. Esto permitirá que sea personalizada mas adelante, debido a que las actualizaciones del módulo no destruirán los cambios realizados. Pero ten en cuenta que esto será así mientras se esté desarrollando, por lo tanto es probable que quieras fijar `noupdate="0"` durante el desarrollo, hasta que estés feliz con el archivo de datos.

En el campo `groups` también encontraras una expresión especial. Es un campo de relación uno a muchos, y tienen una sintaxis especial para operar con ellos. En este caso la tupla `(4,x)` indica agregar `x` a los registros, y `x` es una referencia al grupo empleados, identificado por `base.group_user`.

Como se hizo anteriormente, debemos agregar el archivo a `__openerp__.py` antes que pueda ser cargado al módulo:

```Python
'data':    [
    'todo_view.xml',
    'security/ir.model.access.csv',
    'security/todo_access_rules.xml',
],
```

## Agregar un ícono al módulo
Nuestro módulo se ve genial. ¿Por qué no añadir un ícono para que se vea aún mejor?. Para esto solo debemos agregar al módulo el archivo `static/description/icon.png` con el ícono que usaremos.

Los siguientes comandos agregan un ícono copiado del módulo raíz `Notes`:

```shell
$ mkdir -p ~/odoo-dev/custom-addons/todo_app/static/description
$ cd ~/odoo-dev/custom-addons/todo_app/static/description
$ cp ../odoo/addons/note/static/description/icon.png ./
```

Ahora, si actualizamos la lista de módulos, nuestro módulo debe mostrarse con el ícono nuevo.

## Resumen
Creamos un módulo nuevo desde cero, cubriendo los elementos más frecuentemente usados en un módulo: modelos, los tres tipos base de vistas (formulario, lista y búsqueda), la lógica de negocio en los métodos del modelo, y seguridad en el acceso.

En el proceso, se familiarizó con el proceso de desarrollo de módulos, el cual incluye la actualización del módulo y la aplicación de reinicio del servidor para hacer efectivos en Odoo los cambios graduales.

Recuerde siempre, al agregar campos en el modelo, que es necesaria una actualización del módulo. Cuando se cambia el código Python, incluyendo el archivo de manifiesto, es necesario un reinicio del servidor. Cuando se cambian archivos XML o CSV es necesaria una actualización del módulo; incluso en caso de duda, realice ambas: actualización del módulo y reinicio del servidor.

En el siguiente capítulo, se aprenderá sobre la construcción de módulos que se acoplan a otro existentes para agregar características.
