.. _doc_inputevent:

InputEvent
==========

Что это?
---------

Управление вводом обычно сложно, вне зависимости от OS или платформы. Чтобы
немного это упростить, предоставлен специальный встроенный тип, :ref:`InputEvent <class_InputEvent>`.
Этот тип данных может быть сконфигурирован так, чтобы содержать несколько типов
событий ввода. Input Events проходят через движок и могут быть приняты в
разных местах, в зависимости от целей.

Как это работает?
-----------------

Каждое событие ввода возникает от пользователя/игрока (но можно
генерировать InputEvent и направлять их обратно в движок,
это полезно для жестов). Объект OS для каждой платформы будет считывать
события с устройства, и затем передавать их в MainLoop. Поскольку :ref:`SceneTree <class_SceneTree>`
дефолтная реализация MainLoop, события попадают в нее. Godot
предлагает функцию для получения текущего объекта SceneTree :
**get_tree()**.

Но SceneTree не знает что делать с событиями, так что передаст их
во viewports, начиная с "root" :ref:`Viewport <class_Viewport>` (первый
из узлов в scene tree). Viewport делает довольно много с принятым вводом,
по порядку:

.. image:: /img/input_event_flow.png

1. Прежде всего, будет вызвана стандартная функция _input
   в любом узле с включенной обработкой ввода (с включенной
   :ref:`Node.set_process_input() <class_Node_set_process_input>` and override
   :ref:`Node._input() <class_Node__input>`). Если какая то функция потребляет событие, она может вызвать :ref:`SceneTree.set_input_as_handled() <class_SceneTree_set_input_as_handled>`,
   и событие больше не будет распространяться. Это гарантирует что вы можете 
   фильтровать все интересующие события, даже прежде GUI. 
   Для геймплейного ввода, _unhandled_input() обычно подходит лучше, 
   поскольку позволяет GUI перехватывать события.
2. Во-вторых, Viewport пытается передать событие в GUI, и смотрит может ли
   control принять его. Если да, то будет вызван :ref:`Control <class_Control>` 
   через виртуальную функцию :ref:`Control._input_event() <class_Control__input_event>` 
   и сгенерирован сигнал "input_event" (эта функция повторно реализуется через скрипт
   наследуя от него). Если GUI элемент (control) хочет потребить "consume" событие,
   он вызовет :ref:`Control.accept_event() <class_Control_accept_event>` 
   и событие не будет больше распространяться.
3. Если до сих пор никто не использовал событие, вызывается коллбэк unhandled input
    (enable with
   :ref:`Node.set_process_unhandled_input() <class_Node_set_process_unhandled_input>` и override
   :ref:`Node._unhandled_input() <class_Node__unhandled_input>`). Если какая-либо функция потребляет 
   событие, она вызовет :ref:`SceneTree.set_input_as_handled() <class_SceneTree_set_input_as_handled>`, и
   событие перестает распространяться. Коллбэк unhandled input идеален для игровых событий в полно-экранном режиме,
   поскольку они не принимаются когда GUI активен.
4. Если никто не хотел события до сих пор, и :ref:`Camera <class_Camera>` назначен
   Viewport`у, будет испущен луч в physics world (в направлении луча от клика) будет испущен.
   если этот луч попадет в объект, он вызовет функцию
   :ref:`CollisionObject._input_event() <class_CollisionObject__input_event>` function in the relevant
   physics object (по-умолчанию этот коллбэк принимают тела (bodies), но не areas. 
   Это можно сконфигурировать через свойства :ref:`Area <class_Area>`).
5. Наконец, если событие так и не было обработано, оно передается в следующий
   Viewport в дереве, или будет проигнорировано.

Анатомия InputEvent
--------------------

:ref:`InputEvent <class_InputEvent>` это только базовый встроенный тип, он ничего не представляет
кроме того что содержит некоторую базовую информацию, такую как event ID
(который увеличивается для каждого следующего события), индекс устройства и т.п.

InputEvent имеет член "type". Назначая его, событие может стать разного типа.
Каждый тип InputEvent имеет различные свойства, соответствующие его роли.

Пример изменения типа события.

::

    # создаем событие
    var ev = InputEvent()
    # задаем тип индекс
    ev.type = InputEvent.MOUSE_BUTTON
    # button_index единственно доступный для этого типа
    ev.button_index = BUTTON_LEFT

Вот различные типы InputEvent, описанные в таблице:

+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| Событие                                                           | Type Index         | Описание                                |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEvent <class_InputEvent>`                              | NONE               | Пустое событие ввода                    |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventKey <class_InputEventKey>`                        | KEY                | Содержит значение scancode и unicode,   |
|                                                                   |                    | а также modifiers.                      |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventMouseButton <class_InputEventMouseButton>`        | MOUSE_BUTTON       | Содержит информацию о клике, такую как  |
|                                                                   |                    | кнопка, modifiers, и т.п.               |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventMouseMotion <class_InputEventMouseMotion>`        | MOUSE_MOTION       | Содержит информацию о перемещении,      |
|                                                                   |                    | относит. и абсолютн. позиции и скорость |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventJoystickMotion <class_InputEventJoystickMotion>`  | JOYSTICK_MOTION    | Информация об осях аналогового          |
|                                                                   |                    | Joystick/Joypad.                        |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventJoystickButton <class_InputEventJoystickButton>`  | JOYSTICK_BUTTON    | Информация о кнопках Joystick/Joypad    |
|                                                                   |                    |                                         |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventScreenTouch <class_InputEventScreenTouch>`        | SCREEN_TOUCH       | Contains multi-touch press/release      |
|                                                                   |                    | information. (only available on mobile  |
|                                                                   |                    | devices)                                |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventScreenDrag <class_InputEventScreenDrag>`          | SCREEN_DRAG        | Contains multi-touch drag information.  | 
|                                                                   |                    | (only available on mobile devices)      |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+
| :ref:`InputEventAction <class_InputEventAction>`                  | SCREEN_ACTION      | Contains a generic action. These events |
|                                                                   |                    | are often generated by the programmer   |
|                                                                   |                    | as feedback. (more on this below)       |
+-------------------------------------------------------------------+--------------------+-----------------------------------------+

Actions (действия)
------------------

InputEvent могут (или могут не-) представлять пред-установленное действие. 
Actions полезны потому, что они абстрактнее устройств ввода при программировании 
игровой логики. Это позволяет:

-  Работать одному и тому же коду на разных устройствах с разным вводом (e.g.,
   клавиатура на PC, Joypad на консоли).
-  Input to be reconfigured at run-time.

Actions можно создавать в меню Project Settings во вкладке Actions.
Читайте :ref:`doc_simple_2d_game-input_actions_setup` о том
как работает редактор действий.

Любое событие имеет методы :ref:`InputEvent.is_action() <class_InputEvent_is_action>`,
:ref:`InputEvent.is_pressed() <class_InputEvent_is_pressed>` и :ref:`InputEvent <class_InputEvent>`.

Альтернативно, it may be desired to supply the game back with an action
from the game code (a good example of this is detecting gestures).
SceneTree (derived from MainLoop) has a method for this:
:ref:`MainLoop.input_event() <class_MainLoop_input_event>`. You would normally use it like this:

::

    var ev = InputEvent()
    ev.type = InputEvent.ACTION
    # set as move_left, pressed
    ev.set_as_action("move_left", true) 
    # feedback
    get_tree().input_event(ev)

InputMap
--------

Customizing and re-mapping input from code is often desired. If your
whole workflow depends on actions, the :ref:`InputMap <class_InputMap>` singleton is
ideal for reassigning or creating different actions at run-time. This
singleton is not saved (must be modified manually) and its state is run
from the project settings (engine.cfg). So any dynamic system of this
type needs to store settings in the way the programmer best sees fit.
