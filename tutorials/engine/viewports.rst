.. _doc_viewports:

Вьюпорты
=========

Введение
------------

В Godot небольшая но очень полезная функция - вьюпорты. Вьюпорты
кк подразумевает название, прямоугольники в которых отрисовывается мир.
Они имеют три основных назначения, но могут гибко адаптироваться к гораздо большему. 
Все это делают через узел :ref:`Viewport <class_Viewport>`.

.. image:: /img/viewportnode.png

Основными видами использования являются:

-  **Scene Root**: root активной сцены всегда Viewport.
   Это то, что отображает сцены, созданные пользователем. (Вы должны 
   уже знать об этом из предыдущих уроков!)
-  **Sub-Viewports**: These can be created when a Viewport is a child of
   a :ref:`Control <class_Control>`.
-  **Render Targets**: Вьюпорты могут быть установлены в "RenderTarget" режиме.
   это значит что видовое окно невидимо напрямую, но к его содержимому
   можно обратиться через :ref:`Texture <class_Texture>`.

Ввод
-----

Вьюпорты are also responsible of delivering properly adjusted and
scaled событий ввода ко всем его дочерним узлам. И root viewport
и sub-viewports делают это автоматически, но render targets нет.
Поэтому, пользователи должны делать это вручную через функцию
:ref:`Viewport.input() <class_Viewport_input>` если нужно.

Слушатель
--------

Godot поддерживает 3D звук (и в 2D и в 3D узлах), больше об этом
мы узнаем в следующих уроках (one day..). Чтобы слышать такой тип звука,
вьюпорт должен быть включен как слушатель (для 2D или 3D).
Если вы используете custom viewport для отображения вашего мира, не забудьте
включить это!

Камеры (2D и 3D)
-----------------

При использовании 2D или 3D :ref:`Camera <class_Camera>` /
:ref:`Camera2D <class_Camera2D>`, камеры всегда отображают ближайший
родительский вьюпорт (восходящий к root). Например, в след. иерархии:

-  Viewport

   -  Camera

Камера отобразится в родительском видовом экране, но в следующем:

-  Camera

   -  Viewport

Нет (или может отображаться в корневом окне просмотра, если это под-сцена).

В окне просмотра может быть только одна активная камера, так что,
если их больше чем одна, убедитесь что у желаемой камеры активировано
свойство "current" или сделать камеру текущей вызовом:

::

    camera.make_current()

Масштаб и растяжение
------------------

У вьюпортов есть свойства "rect". X и Y часто не используют (только
root вьюпорт реально их использует), а WIDTH и HEIGHT представляют
размер вьюпорта в пикселях. Для Sub-Viewports, эти значения
перекрываются тем что установлено в parent control, но для render targets
это задает их разрешение.

2D содержимое также можно масштабировать and make it believe the
viewport resolution is other than the one specified in the rect, 
вызовом:

::

    viewport.set_size_override(w,h) #кастомный размер для 2D
    viewport.set_size_override_stretch(true/false) #enable stretch для кастомного размера

Root вьюпорт использует это для параметров растяжения в настройках проекта.

Миры
------

Для 3D, вьюпорт будет содержать :ref:`World <class_World>`. 
Это базовая вселенная которая связывает физику и рендер.
Spatial-base узлы регистрируют с помощью World ближайшего вьюпорта.
По умолчанию, вновь созданные вьпорты не содержат World но
используют тот же что и родительский вьюпорт (root вьюпорт does contain one
though, which is the one objects are rendered to by default). A world can
be set in a viewport using the "world" property, and that will separate
all children nodes of that viewport from interacting with the parent
viewport world. This is specially useful in scenarios where, for
example, вам может понадобиться показать отдельного персонажа в 3D наложенного
поверх игры (как в Starcraft).

As a helper for situations where you want to create viewports that
display single objects and don't want to create a world, viewport has
the option to use its own World. Это очень пригодиться когда вы хотите
инстанцировать 3D персонаж или объект в 2D мире.

Для 2D, каждый Viewport всегда содержит свои собственные :ref:`World2D <class_World2D>`.
Этого достаточно в большинстве случаев, but in case sharing them may be desired, it
is possible to do so by calling the viewport API manually.

Захват содержимого экрана
-------

Можно запросить захват содержимого вьюпорта. Для root
viewport это фактически захват экрана. Это выполняют
следующим API:

::

    # queues a screen capture, will not happen immediately
    viewport.queue_screen_capture() 

После кадра или двух (check _process()), захват будет готов,
верните его используя:

::

    var capture = viewport.get_screen_capture()

Если возвращенное изображение пусто, захват все еще не произошел, 
подождите немного дольше, поскольку это API асинхронно.

Sub-viewport
------------

Если вьюпорт является потомком control, он станет активным
и покажет все, что есть внутри. Макет выглядит примерно так:

-  Control

   -  Viewport

Область вьюпорта полностью покрывает область родительского control.

.. image:: /img/subviewport.png

Render target
-------------

Чтобы настроить как render target, просто переключите свойство "render target" 
tвьюпорта. Заметьте, что все, что находится внутри, не будет отображаться
в редакторе сцен. Для отображения содержимого, нужно использовать render target
texture. Это можно запросить с помощью кода (например):

::

    var rtt = viewport.get_render_target_texture() 
    sprite.set_texture(rtt)

By default, re-rendering of the render target happens when the render
target texture has been drawn in a frame. If visible, it will be
rendered, otherwise it will not. This behavior can be changed to manual
rendering (once), or always render, no matter if visible or not.

A few classes are created to make this easier in most common cases
inside the editor:

-  :ref:`ViewportSprite <class_ViewportSprite>` (for 2D).
-  ViewportQuad (for 3D).
-  ViewportFrame (for GUI).

*TODO: Review the doc, ViewportQuad and ViewportFrame don't exist in 2.0.*

Make sure to check the viewport demos! Viewport folder in the demos
archive available to download, or
https://github.com/godotengine/godot-demo-projects/tree/master/viewport
