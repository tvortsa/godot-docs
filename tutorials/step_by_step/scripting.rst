.. _doc_scripting:

Скриптинг
=========

Введение
------------

Мы много говорили об инструментах которые позволяют обойтись без программирования.
Это мечта многих разработчиков - программировать игры без обучения кодингу.
Эта потредность есть даже внутри компаний в которых разработчики хотели бы больше контроля
над потоком игры .

Многие обещают среду без программирования, но чаще всего результат оказывается неполным, 
слишком сложным и неэффективным по сравнению с классическим кодингом. 
Так что программирование сохраниться еще надолго. Общее направление развития
игровых систем в том чтобы дать инструменты сокращающие необходимость написания кода
для типовых задач.

Поэтому, Godot принял пару важных решений. Во-первых система сцен. Она облегчает
программисту зависимость от архитектуры.

При азработке игр с использованием системы сцен, весь проект фрагментирован на 
*комплкментарные* сцены (а не отдельные). Сцены составляют друг друга,вместо того 
чтобы быть раздельными. 

Имеющие опыт знают о чем-то вроде MVC. Godot обещает эффективность без привычного MVC
заменяя его на паттерн *сцены как составляющие*.

Godot также использует паттерн  `extend <http://c2.com/cgi/wiki?EmbedVsExtend>`__
для скриптинга, это означает что скрипты доступны для всех доступных классов движка.

GDScript
--------

:ref:`doc_gdscript` это динамически типизированный скриптовый язык работающий
внутри Godot. Он разработан с прицелом на:

-  Быть простым и узнаваемым на сколько это возможно.
-  Создаваемый код читабелен и безопасен. Синтаксис в основном заимствован из Python.

Программистам обычно нужно пара дней чтобы выучить его, и пару недель на комфортное освоение.

Как и большинство типизированных языков, удобство
(простота чтения кода, скорость написания, остутствие компиляции, и т.п.)
это компромис с потерей производительности. Но основной критический код 
написан на C++ прямо в движке (vector ops, physics, math, indexing, etc),
в результате его производительности более чем достаточно для большинства
типов игр.

В любом случае, если необходимо больше производительности, критические разделы
можно переписать на C++ и exposed прозрачно в script. Это позволяет заменить
GDScript класс на C++ класс без изменения остальной игры.

Скриптинг сцены
-----------------

Перед тем как продолжить, обязательно прочтите справку по :ref:`doc_gdscript`.
Это простой язык и справочное по нему короткое, это не займет дольше нескольких минут
и позволит ознакомиться с основными концепциями.

Настройка сцены
~~~~~~~~~~~

Урок начинается со скриптинга небольшой GUI сцены. Используйте окно добавления узла
для создания следующей иерархии со следующими узлами:

- Panel

  * Label
  * Button

Это должно выглядеть примерно так:

.. image:: /img/scripting_scene_tree.png

Используйте 2D редактор чтобы расположить и масштабировать кнопку и label 
так как на картинке. Можете задать text на панели Inspector.

.. image:: /img/label_button_example.png

Сохраните сцену, с названием "sayhello.scn"

.. _doc_scripting-adding_a_script:

Добавление скрипта
~~~~~~~~~~~~~~~

Правым кликом на панели node, в контекстном меню выберите "Add Script" :

.. image:: /img/add_script.png

Появится окно создания скрипта. Это окно позволяет выбрать язык, имя класса, и т.п.
GDScript не использует имена классов в файле скрипта, так что это поле не редактируемо.
Скрипт должен наследовать от "Panel" (это значит расширить узел, который имеет тип Panel,
это заполняется автоматически).

Введите path name для скрипта и затем нажмите "Create":

.. image:: /img/script_create.png

После чего, скрипт должен создасться и добавиться к узлу. 
Это видно как дополнительный значок на узле, а также как свойство script:

.. image:: /img/script_added.png

To edit the script, select either of the highlighted buttons. 
This will bring you to the script editor where an existing template will
be included by default:

.. image:: /img/script_template.png

There is not much in there. The "_ready()" function is called when the
node (and all its children) entered the active scene. (Remember, it's
not a constructor, the constructor is "_init()" ).

The role of the script
~~~~~~~~~~~~~~~~~~~~~~

A script adds behavior to a node. It is used to control the
node functions as well as other nodes (children, parent, siblings, etc).
The local scope of the script is the node (just like in regular
inheritance) and the virtual functions of the node are captured by the
script.

.. image:: /img/brainslug.jpg

Handling a signal
~~~~~~~~~~~~~~~~~

Signals are used mostly in GUI nodes (although other nodes have them
too). Signals are "emitted" when some specific kind of action happens,
and can be connected to any function of any script instance. In this
step, the "pressed" signal from the button will be connected to a custom
function.

An interface for connecting signals to your scripts exists in the editor. 
You can access this by selecting the node in the scene tree and then
selecting the "Node" tab. Make sure that you have "Signals" selected.

.. image:: /img/signals.png

In any case, at this point it is clear that we are interested in
the "pressed" signal. Instead of using the visual interface, we will opt
to code the connection.

For this, a function exists that is probably the one most used by Godot
programmers, namely :ref:`Node.get_node() <class_Node_get_node>`.
This function uses paths to fetch nodes in the current tree or anywhere
in the scene, relative to the node holding the script.

To fetch the button, the following must be used:

::

    get_node("Button")

Next, a callback will be added that will change the label's text when
the button is pressed:

::

    func _on_button_pressed():  
        get_node("Label").set_text("HELLO!")

Finally, the button "pressed" signal will be connected to that callback
in _ready(), by using :ref:`Object.connect() <class_Object_connect>`.

::

    func _ready():
        get_node("Button").connect("pressed",self,"_on_button_pressed")

The final script should look like this:

::

    extends Panel

    # member variables here, example:

    # var a=2
    # var b="textvar"

    func _on_button_pressed():
        get_node("Label").set_text("HELLO!")

    func _ready():
        get_node("Button").connect("pressed",self,"_on_button_pressed")

Running the scene should have the expected result when pressing the
button:

.. image:: /img/scripting_hello.png

**Note:** As it is a common misunderstanding in this tutorial, let's clarify
again that get_node(path) works by returning the *immediate* children of
the node controlled by the script (in this case, *Panel*), so *Button*
must be a child of *Panel* for the above code to work. To give this
clarification more context, if *Button* were a child of *Label*, the code
to obtain it would be:

::

    # not for this case
    # but just in case
    get_node("Label/Button") 

Also, remember that nodes are referenced by name, not by type.
