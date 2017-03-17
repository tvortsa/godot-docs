.. _doc_scene_tree:

SceneTree
=========

Введение
------------

Тут вещи начинают становиться абстрактными, но без паники, 
на самом деле все не страшно.

В предыдущих уроках, все крутилось вокруг концепции узлов,
из них состоят сцены, и они становятся активны как только они входят в
 *scene tree*.

Фактически, система сцен даже не является ядром Godot, поскольку можно написать скрипт
(или код на C++) который будет непосредственно контактировать с серверами. 
Но создание игры потребует куда больше усилий.

MainLoop (главный цикл)
--------

Внутри Godot работает is as follows. Есть класс
:ref:`OS <class_OS>` ,
который является единственным экземпляром который выполняется вначале.
После него загружаются все драйверы, серверы, языки скриптинга, система сцен, и т.п.

По завершении инициализации, :ref:`OS <class_OS>` должен предоставить
:ref:`MainLoop <class_MainLoop>`
для запуска. До этого момента, все происходит внутри (можете посмотреть
файл main/main.cpp в исходниках если вас интересует что происходит внутри).

Пользовательское приложение, или игра, стратует в MainLoop. Этот класс имеет несколько методов,
для инициализации, idle (коллбэк синхронизации кадров), fixed
(коллбэк синхронизации физики), и input. Повторюсь, все это действительно
низкоуровневые вещи и при создании игр в Godot, написание собственного MainLoop не имеет смысла.

SceneTree
---------

Одним из способов разобраться с тем как работает Godot, в том что это 
высоко-уровневый игровой движок поверх низкоуровневого middleware.

Система сцен это game engine, а :ref:`OS <class_OS>`
и servers это низкоуровневые API.

В любом случае, система сцен предоставляет свой собственный main loop для OS,
:ref:`SceneTree <class_SceneTree>`.
Он автоматически инстанцируется и настраивается при запуске сцены, 
не требуя никакой дополнительной работы.

Важно знать что этот класс существует потому что он имеет несколько
важных использований:

-  Он содержит root :ref:`Viewport <class_Viewport>`, к которому 
   добавляется сцена как потомок при первом открытии, становясь частью *Scene Tree*
   (подробнее об этом)
-  Он содержит информацию о группах, и имеет средства для вызова всех узлов в группе,
   или их списка.
-  Он содержит некоторые функции глобального состояния, такие как режим установки паузы, 
   или процесс выхода.

Если узел является частью Scene Tree, синглтон
:ref:`SceneTree <class_SceneTree>`
можно получить простым вызовом
:ref:`Node.get_tree() <class_Node_get_tree>`.

Root viewport
-------------

root :ref:`Viewport <class_Viewport>`
всегда вверху сцены. From a node, он может быть получен двумя разными способами:

::

        get_tree().get_root() # доступ через scenemainloop
        get_node("/root") # доступ через абсолютный путь

Этот узел содержит основной main viewport, все что является потомком
:ref:`Viewport <class_Viewport>`
отрисовывается внутри него по-умолчанию, поэтому имеет смысл то что выше всех
узлов в иерархии всегда узел этого типа, иначе ничего не будет видно!

В то время как другие вьюпорты могут быть созданы в сцене (например для
сплит-скрина), только этот один никогда не создается пользователем.
Он создается автоматически внутри  SceneTree.

Scene tree
----------

When a node is connected, directly or indirectly, to the root
viewport, it becomes part of the *scene tree*.

This means that, as explained in previous tutorials, it will get the
_enter_tree() and _ready() callbacks (as well as _exit_tree()).

.. image:: /img/activescene.png

When nodes enter the *Scene Tree*, they become active. They get access
to everything they need to process, get input, display 2D and 3D,
notifications, play sound, groups, etc. When they are removed from the
*scene tree*, they lose access.

Tree order
----------

Most node operations in Godot, such as drawing 2D, processing or getting
notifications are done in tree order. This means that parents and
siblings with a smaller rank in the tree order will get notified before
the current node.

.. image:: /img/toptobottom.png

"Becoming active" by entering the *Scene Tree*
----------------------------------------------

#. A scene is loaded from disk or created by scripting.
#. The root node of that scene (only one root, remember?) is added as
   either a child of the "root" Viewport (from SceneTree), or to any
   child or grand-child of it.
#. Every node of the newly added scene, will receive the "enter_tree"
   notification ( _enter_tree() callback in GDScript) in top-to-bottom
   order.
#. An extra notification, "ready" ( _ready() callback in GDScript) is
   provided for convenience, when a node and all its children are
   inside the active scene.
#. When a scene (or part of it) is removed, they receive the "exit
   scene" notification ( _exit_tree() callback in GDScript) in
   bottom-to-top order

Changing current scene
----------------------

After a scene is loaded, it is often desired to change this scene for
another one. The simple way to do this is to use the
:ref:`SceneTree.change_scene() <class_SceneTree_change_scene>`
function:

::

    func _my_level_was_completed():
        get_tree().change_scene("res://levels/level2.scn")

This is a quick and useful way to switch scenes, but has the drawback
that the game will stall until the new scene is loaded and running. At
some point in your game, it may be desired to create proper loading
screens with progress bar, animated indicators or thread (background)
loading. This must be done manually using autoloads (see next chapter!)
and :ref:`doc_background_loading`.
