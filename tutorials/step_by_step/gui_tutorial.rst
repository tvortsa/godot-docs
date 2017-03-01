.. _doc_gui_tutorial:

GUI туториал
============

Введение
~~~~~~~~~~~~

Больше всего программисты не любят программировать GUI - графич.
интерфейс пользователя. Это скучно, утомительно и невызывающе. 

Наиболее неприятные аспекты:

-  Пиксельное выравнивание UI элементов - сложно (особенно выдерживать
   все таким как задумал дизайнер).
-  UI постоянно меняется из-за соображений юзабилити, тестирования и т.п..
-  Зоопарк разрешений экранов.
-  Анимирование компонентов UI.

GUI программирование лидирует среди причин выгорания программистов.
Разрабатывая Godot мы попробовали несколько техник и философий,
разработки UI таких как немедленный режим , контейнеры, якори, скриптинг etc.
Главным образом для снижения стресса программистов работающих с GUI.

В результате, подсистема UI в Godot это еффективная система соединяющая
несколько различных подходов. Кривая обучения немного круче других,
но разработчики могут создавать сложные интерфейсы очень быстро,
разделяя один и тот же набор инструментов с дизайнерами и аниматорами.

Элементы управления
~~~~~~~

Базовый узел для UI элементов это :ref:`Control <class_Control>`
(иногда называемый "Widget" или "Box" в других тулсетах). Каждый узел
предоставляющий функционал интерфейса наследует от него.

Когда контролы помещают в дерево сцены как потомок другого контрола,
его координаты (положение, размер) всегда относительно родителя.
Это закладывает основу для редактирования сложных пользовательских
интерфейсов быстро и визуально.

Ввод и отрисовка
~~~~~~~~~~~~~~~~~

Контролы принимают события ввода посредством
:ref:`Control._input_event() <class_Control__input_event>`
коллбэка. Только один контрол, который в фокусе, принимает события
клавиатуры/джойпада (см
:ref:`Control.set_focus_mode() <class_Control_set_focus_mode>`
и :ref:`Control.grab_focus() <class_Control_grab_focus>`).

События движения мыши принимаются контролом прямо с указателя мыши.
Когда контрол принимает события нажатия кнопок мыши, все
последующие события перемещения принимаются контролом который был нажат
пока кнопка не будет отпущена, даже если указатель выйдет за пределы контрола.

Как всякий класс наследующий от :ref:`CanvasItem <class_CanvasItem>`
(Control does), и :ref:`CanvasItem._draw() <class_CanvasItem__draw>`
коллбэк будет принимать вначале и каждый раз когда контрол должен быть
перерисован (программист должен вызвать
:ref:`CanvasItem.update() <class_CanvasItem_update>`
поставить в очередь CanvasItem для перерисовки). Если контрол невидим
(yet another CanvasItem property), контрол не принимает никакой ввод.

В общем хотя, программист не должен иметь дело с отрисовкой и
событиями ввода непосредственно при построении UI (это удобно при 
создании собственных контролов). Вместо этого, controls выделяют различные виды 
сигналов с контекстной информацией когда происходит действие. Например
 :ref:`Button <class_Button>` испускает
сигнал "pressed" при нажатии, :ref:`Slider <class_Slider>` будет испускать
"value_changed" при перетаскивании, etc.

Мини-урок по собственным контролам
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Перед тем ка углубляться, создание собственного контрола
даст полнее почувствовать как работают контролы, поскольку это
не так сложно как может показаться.

Кроме того, хоть Godot поставляется с множеством контролов, 
часто оказывается проще реализовать то что нужно в собственном.

Для начала, создайте сцену с единственным узлом. Тип нода "Control" и
имеет определенную область экрана в 2D редакторе, типа так:

.. image:: /img/singlecontrol.png

Добавьте скрипт к этому узлу
::

    extends Control

    var tapped=false

    func _draw():

        var r = Rect2( Vector2(), get_size() )
        if (tapped):
            draw_rect(r, Color(1,0,0) )
        else:
            draw_rect(r, Color(0,0,1) )

    func _input_event(ev):

        if (ev.type==InputEvent.MOUSE_BUTTON and ev.pressed):
            tapped=true
            update()

Then run the scene. When the rectangle is clicked/tapped, it will change
color from blue to red. That synergy between the events and the drawing
is pretty much how most controls work internally.

.. image:: /img/ctrl_normal.png

.. image:: /img/ctrl_tapped.png

UI complexity
~~~~~~~~~~~~~

As mentioned before, Godot includes dozens of controls ready for use
in a user interface. Such controls are divided in two categories. The
first is a small set of controls that work well for creating most game
user interfaces. The second (and most controls are of this type) are
meant for complex user interfaces and uniform skinning through styles. A
description is presented as follows to help understand which one should
be used in which case.

Simplified UI controls
~~~~~~~~~~~~~~~~~~~~~~

This set of controls is enough for most games, where complex
interactions or ways to present information are not necessary. They can
be skinned easily with regular textures.

-  :ref:`Label <class_Label>`: Node used for showing text.
-  :ref:`TextureFrame <class_TextureFrame>`: Displays a single texture,
   which can be scaled or kept fixed.
-  :ref:`TextureButton <class_TextureButton>`: Displays a simple textured
   button, states such as pressed, hover, disabled, etc. can be set.
-  :ref:`TextureProgress <class_TextureProgress>`: Displays a single
   textured progress bar.

Additionally, re-positioning of controls is most efficiently done with
anchors in this case (see the :ref:`doc_size_and_anchors` tutorial for more
information).

In any case, it will happen often that even for simple games, more
complex UI behaviors are required. An example of this is a scrolling
list of elements (for a high score table, for example), which needs a
:ref:`ScrollContainer <class_ScrollContainer>`
and a :ref:`VBoxContainer <class_VBoxContainer>`.
These kind of more advanced controls can be mixed with the regular ones
seamlessly (they are all controls after all).

Complex UI controls
~~~~~~~~~~~~~~~~~~~

The rest of the controls (and there are dozens of them!) are meant for
another set of scenarios, most commonly:

-  Games that require complex UIs, such as PC RPGs, MMOs, strategy,
   sims, etc.
-  Creating custom development tools to speed up content creation.
-  Creating Godot Editor Plugins, to extend the engine functionality.

Re-positioning controls for these kind of interfaces is more commonly
done with containers (see the :ref:`doc_size_and_anchors` tutorial for more
information).
