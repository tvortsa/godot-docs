.. _doc_singletons_autoload:

Singletons (AutoLoad)
=====================

Введение
------------

Синглтоны сцен очень полезны, обслуживая частые ситуации когда вам нужно
сохранить информацию при переходе между сценами.

Несмотря на то что система сцен очень мощная, у нее есть некоторые недостатки:

-  Нет общего места для хранения информации (такой как предметы пользователя и т.п.)
   нужной в разных сценах.
-  Хотя та сцена которая загружает и выгружает другие сцены как своих потомков может
   сохранять информацию общую для них, нельзя ожидать что их самостоятельное 
   использование будет работать корректно.
-  Конечно информацию можно сохранять на диск \`user://\` и эта информация может быть
   загружена сценами которым она нужна, но непрерывное сохранение 
   и загрузка данных во время смены сцен громоздка и может быть медленной.

Однако до сих пор существует необходимость создания в Godot частей сцен которые:

-  Загружаются всегда, независимо от того какая сцена открыта в редакторе
-  Могут хранить глобальные переменные, такие как информация об игроке, предметах, валюте
   и разделять эту информацию между сценами
-  Могут обрабатывать переключение между сценами и переходы между ними
-  Служить синглтоном, поскольку дизайн GDScript не поддерживает
   глобальные переменные по дизайну.

Auto-loading узлов и скриптов удовлетворяют этой потребности.

AutoLoad
--------

Вы можете использовать AutoLoad для загрузки сцены, или скрипта который наследует от Node
(node будет создан и для него будет установлен скрипт). 

To autoload a scene or script, select Scene > Project Settings from the menu and switch
to the AutoLoad tab. Each entry in the list requires a name, which is used as the name
of the node, and the node is always added to the root viewport before any other scenes 
are loaded.

.. image:: /img/singleton.png

This means that any node can access a singleton named "playervariables" with:

::

   var player_vars = get_node("/root/playervariables")

Custom scene switcher
---------------------

This short tutorial will explain how to make a scene switcher using
autoload. For simple scene switching, the
:ref:`SceneTree.change_scene() <class_SceneTree_change_scene>`
method suffices (described in :ref:`doc_scene_tree`), so this method is for
more complex behavior when switching between scenes.

First download the template from here:
:download:`autoload.zip </files/autoload.zip>`, then open it.

Two scenes are present, scene_a.scn and scene_b.scn on an otherwise
empty project. Each are identical and contain a button connected to a
callback for switching to the other scene. When the project runs, it
starts in scene_a.scn. However, this currently does nothing and pressing the
button does not work.

global.gd
---------

First of all, create a global.gd script. The easy way to create a
resource from scratch is from the new resource button in the inspector tab:

.. image:: /img/newscript.png

Save the script as `global.gd`:

.. image:: /img/saveasscript.png

The script should open in the script editor. The next step is to add
it to AutoLoad list. Select Scene > Project Settings from the menu,
switch to the AutoLoad tab and add a new entry with name "global" that
points to this file:

.. image:: /img/addglobal.png

Now, whenever you run any of your scenes, the script is always loaded.

Returning to our script, the current scene needs to be fetched in the 
`_ready()` function. Both the current scene and `global.gd` are children of
root, but the autoloaded nodes are always first. This means that the
last child of root is always the loaded scene.

Note: Make sure that global.gd extends Node, otherwise it won't be
loaded!

::

    extends Node

    var current_scene = null

    func _ready():
            var root = get_tree().get_root()
            current_scene = root.get_child( root.get_child_count() -1 )

Next up is the function for changing the scene. This function frees the
current scene and replaces it with the requested one.

::

    func goto_scene(path):

        # This function will usually be called from a signal callback,
        # or some other function from the running scene.
        # Deleting the current scene at this point might be
        # a bad idea, because it may be inside of a callback or function of it.
        # The worst case will be a crash or unexpected behavior.

        # The way around this is deferring the load to a later time, when
        # it is ensured that no code from the current scene is running:

        call_deferred("_deferred_goto_scene",path)


    func _deferred_goto_scene(path):

        # Immediately free the current scene,
        # there is no risk here.    
        current_scene.free()

        # Load new scene
        var s = ResourceLoader.load(path)

        # Instance the new scene
        current_scene = s.instance()

        # Add it to the active scene, as child of root
        get_tree().get_root().add_child(current_scene)

        # optional, to make it compatible with the SceneTree.change_scene() API
        get_tree().set_current_scene( current_scene )

As mentioned in the comments above, we really want to avoid the
situation of having the current scene being deleted while being used
(code from functions of it being run), so using
:ref:`Object.call_deferred() <class_Object_call_deferred>`
is desired at this point. The result is that execution of the commands
in the second function will happen at a later time when no code from
the current scene is running.

Finally, all that is left is to fill the empty functions in scene_a.gd
and scene_b.gd:

::

    #add to scene_a.gd

    func _on_goto_scene_pressed():
            get_node("/root/global").goto_scene("res://scene_b.scn")

and

::

    #add to scene_b.gd

    func _on_goto_scene_pressed():
            get_node("/root/global").goto_scene("res://scene_a.scn")

Now if you run the project, you can switch between both scenes by pressing
the button!

To load scenes with a progress bar, check out the next tutorial,
:ref:`doc_background_loading`
