.. _doc_scripting_continued:

Scripting (продолжение)
=====================

Процессинг
----------

Некоторые действия в Godot выполняют обратными вызовами или виртуальными функциями,
поэтому нет необходимости проверять написание кода запускаемого все время. 
Кроме того, многоу может быть сделано с помощью анимаций игроков.

Однако, все еще остается распротраненным ситуация когда нужен скрипт
выполняющийся в каждом кадре. Есть два типа процессинга, idle процессинг
и fixed процессинг.

Idle процессинг активируется с помощью функции
:ref:`Node.set_process() <class_Node_set_process>`
После активации, обратный вызов:ref:`Node._process() <class_Node__process>`
будет вызываться в каждом кадре. Пример:

::

    func _ready():
        set_process(true)

    func _process(delta):
        # do something...

Параметр delta описывает время, прошедшее (в секундах, как значение 
с плавающей точкой) с момента предыдущего вызова to _process().
Fixed процессинг похож, но он необходим только для синхронизации с физическим движком.

Простой способ протестировать это - создать сцену с единственным узлом Label,
со следующим скриптом:

::

    extends Label

    var accum=0

    func _ready():
        set_process(true)

    func _process(delta):
        accum += delta
        set_text(str(accum))

Мы увидим счетчик инкрементирующий в каждом кадре.

Группы
------

Узлы можно добавлять в группы (сколько угодно для каждого узла).
это простая но полезная функция для организации больших сцен. 
Есть два способа сделать это, первый из UI, по кнопке Groups в панели Node:

.. image:: /img/groups_in_nodes.PNG

А второй - в коде. Один из полезных примеров - 
помечать сцены которые будут врагами.

::

    func _ready():
        add_to_group("enemies")

Так, если игрок проникнет на секрентую базу,
все враги могут быть уведомлены звуком сирены, с помощью:
:ref:`SceneTree.call_group() <class_SceneTree_call_group>`:

::

    func _on_discovered():
        get_tree().call_group(0, "guards", "player_was_discovered")

Этот код вызывает функцию "player_was_discovered" каждого члена группы "guards".

Опционально, можно получить полный список узлов "guards" вызовом:

:ref:`SceneTree.get_nodes_in_group() <class_SceneTree_get_nodes_in_group>`:

::

    var guards = get_tree().get_nodes_in_group("guards")

Больше будет добавлено позже
:ref:`SceneTree <class_SceneTree>`


Уведомления
-------------

Godot имеет систему уведомлений. Она обычно не нуждается в скриптинге,
поскольку довольно низкоуровневая и и для большинства из них
предусмотрены виртуальные функции. Просто хорошо знать что они есть.
Просто
:ref:`Object._notification() <class_Object__notification>`
 добавьте функцию в ваш скрипт:

::

    func _notification(what):
        if (what == NOTIFICATION_READY):
            print("This is the same as overriding _ready()...")
        elif (what == NOTIFICATION_PROCESS):     
            var delta = get_process_time()
            print("This is the same as overriding _process()...")

The documentation of each class in the :ref:`Class Reference <toc-class-ref>`
shows the notifications it can receive. However, for most cases GDScript
provides simpler overrideable functions.

Overrideable functions
----------------------

Nodes provide many useful overrideable functions, which are described as
follows:

::

    func _enter_tree():
        # When the node enters the _Scene Tree_, it becomes active 
        # and  this function is called. Children nodes have not entered 
        # the active scene yet. In general, it's better to use _ready() 
        # for most cases.
        pass

    func _ready():
        # This function is called after _enter_tree, but it ensures 
        # that all children nodes have also entered the _Scene Tree_, 
        # and became active.
        pass 

    func _exit_tree():
        # When the node exits the _Scene Tree_, this function is called. 
        # Children nodes have all exited the _Scene Tree_ at this point 
        # and all became inactive.
        pass

    func _process(delta):
        # When set_process() is enabled, this function is called every frame.
        pass

    func _fixed_process(delta):
        # When set_fixed_process() is enabled, this is called every physics 
        # frame.
        pass

    func _paused():
        # Called when game is paused. After this call, the node will not receive 
        # any more process callbacks.
        pass

    func _unpaused():
        # Called when game is unpaused.
        pass

As mentioned before, it's best to use these functions.

Creating nodes
--------------

To create a node from code, just call the .new() method (like for any
other class based datatype). Example:

::

    var s
    func _ready():
        s = Sprite.new() # create a new sprite!
        add_child(s) # add it as a child of this node

To delete a node, be it inside or outside the scene, free() must be
used:

::

    func _someaction():
        s.free() # immediately removes the node from the scene and frees it

When a node is freed, it also frees all its children nodes. Because of
this, manually deleting nodes is much simpler than it appears. Just free
the base node and everything else in the sub-tree goes away with it.

However, it might happen very often that we want to delete a node that
is currently "blocked", because it is emitting a signal or calling a
function. This will result in crashing the game. Running Godot
in the debugger often will catch this case and warn you about it.

The safest way to delete a node is by using
:ref:`Node.queue_free() <class_Node_queue_free>`.
This erases the node safely during idle.

::

    func _someaction():
        s.queue_free() # remove the node and delete it while nothing is happening

Instancing scenes
-----------------

Instancing a scene from code is pretty easy and done in two steps. The
first one is to load the scene from disk.

::

    var scene = load("res://myscene.scn") # will load when the script is instanced

Preloading it can be more convenient sometimes, as it happens at parse
time.

::

    var scene = preload("res://myscene.scn") # will load when parsing the script

But 'scene' is not yet a node containing subnodes. It's packed in a
special resource called :ref:`PackedScene <class_PackedScene>`.
To create the actual node, the function
:ref:`PackedScene.instance() <class_PackedScene_instance>`
must be called. This will return the tree of nodes that can be added to
the active scene:

::

    var node = scene.instance()
    add_child(node)

The advantage of this two-step process is that a packed scene may be
kept loaded and ready to use, so it can be used to create as many
instances as desired. This is especially useful to quickly instance
several enemies, bullets, etc. in the active scene.
