.. _doc_resources:

Ресурсы
=========

Узлы и ресурсы
-------------------

До сих пор, :ref:`Nodes <class_Node>`
был наиболее важным типом данных в Godot, поскольку большинство типов поведений
и фич движка реализованы через него. Но есть и другой тип данных, который не менее важен:
:ref:`Resource <class_Resource>`.

Если *Nodes* фокусируется на поведениях, таких как отрисовка спрайтов, 
3D моделей, физике, GUI контролах, и т.п.,

**Resources** это просто **data containers**. Это означает что они не делают
никаких действий и не обрабатывают каких либо данных. Resources просто
содержат данные.

Примеры resources 
:ref:`Texture <class_Texture>`,
:ref:`Script <class_Script>`,
:ref:`Mesh <class_Mesh>`,
:ref:`Animation <class_Animation>`,
:ref:`Sample <class_Sample>`,
:ref:`AudioStream <class_AudioStream>`,
:ref:`Font <class_Font>`,
:ref:`Translation <class_Translation>`,
и др.

Когда Godot сохраняет или загружает (с диска) сцену (.scn или .xml), изображения
(png, jpg), скрипта (.gd) или почти все, является файлом считается ресурсом.

Когда resource загружается с диска, **он всегда загружается один раз**. Что означает,
если есть копия этого ресурса уже загруженная в память,
при попытке загрузить ресурс будет всегда возвращаться уже загруженная копия.
это соответствует тому, что ресурсы это просто контейнеры данных,
поэтому нет необходимости дублировать их.

Обычно, каждый объект в Godot (Node, Resource, или что-то еще) может экспортировать свойства,
pсвойства могут быть разных типов (string,
integer, Vector2, и т.п) и один из этих типов может быть ресурс. Это означает что
и nodes и ресурсы могут содержать ресурсы как свойства.
Проиллюстрируем это:

.. image:: /img/nodes_resources.png

External или built-in
--------------------

Ресурсные свойства могут ссылаться на ресурсы двумя способами,
*external* (на диске) или **built-in**.

To be more specific, here's a :ref:`Texture <class_Texture>`
in a :ref:`Sprite <class_Sprite>` node:

.. image:: /img/spriteprop.png

Pressing the ">" button on the right side of the preview allows to
view and edit the resources properties. One of the properties (path)
shows where it comes from. In this case, it comes from a png image.

.. image:: /img/resourcerobi.png

When the resource comes from a file, it is considered an *external*
resource. If the path property is erased (or it never had a path to
begin with), it is considered a built-in resource.

For example, if the path \`"res://robi.png"\` is erased from the "path"
property in the above example, and then the scene is saved, the resource
will be saved inside the .scn scene file, no longer referencing the
external "robi.png". However, even if saved as built-in, and even though
the scene can be instanced multiple times, the resource will always
be loaded only once. That means, different Robi robot scenes instanced
at the same time will still share the same image.

Loading resources from code
---------------------------

Loading resources from code is easy. There are two ways to do it. The
first is to use load(), like this:

::

    func _ready():
            var res = load("res://robi.png") # resource is loaded when line is executed
            get_node("sprite").set_texture(res)

The second way is more optimal, but only works with a string constant
parameter, because it loads the resource at compile-time.

::

    func _ready():
            var res = preload("res://robi.png") # resource is loaded at compile time
            get_node("sprite").set_texture(res)

Loading scenes
--------------

Scenes are also resources, but there is a catch. Scenes saved to disk
are resources of type :ref:`PackedScene <class_PackedScene>`,
this means that the scene is packed inside a resource.

To obtain an instance of the scene, the method
:ref:`PackedScene.instance() <class_PackedScene_instance>`
must be used.

::

    func _on_shoot():
            var bullet = preload("res://bullet.scn").instance()
            add_child(bullet)                  

This method creates the nodes in hierarchy, configures them (sets all
the properties) and returns the root node of the scene, which can be
added to any other node.

The approach has several advantages. As the
:ref:`PackedScene.instance() <class_PackedScene_instance>`
function is pretty fast, adding extra content to the scene can be done
efficiently. New enemies, bullets, effects, etc can be added or
removed quickly, without having to load them again from disk each
time. It is important to remember that, as always, images, meshes, etc
are all shared between the scene instances.

Freeing resources
-----------------

Resource extends from :ref:`Reference <class_Reference>`.
As such, when a resource is no longer in use, it will automatically free
itself. Since, in most cases, Resources are contained in Nodes, scripts
or other resources, when a node is removed or freed, all the children
resources are freed too.

Scripting
---------

Like any object in Godot, not just nodes, resources can be scripted,
too. However, there isn't generally much of an advantage, as resources
are just data containers.
