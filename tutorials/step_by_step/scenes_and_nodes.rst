.. _doc_scenes_and_nodes:

Сцены и узлы
================

Введение
------------

.. image:: /img/chef.png

Представьте что вы больше не разработчик. Вы шеф-повар. 
Сейчас, вместо создания игр, вы создаете новые деликатесы.

Итак, как повар создает рецепты? Рецепты делятся на два типа вещей:
ингридиенты и инструкции по их приготовлению.
Таким образом любой, следуя рецепту может создать удивительное творение.

Создание игр в Godot очень похоже. Использование движка как кухни.
В этой кухне, *nodes* (узлы) как холодильник наолненный свежими ингридиентами.

Существует много типов узлов, некоторые отображают изображение, другие воспроизводят звук,
а третьи - 3D модели, и т.д. .

Узлы
-----

Но давайте начнем с основ. Узел это базовый элемент для создания игры,
он обладает следующими характеристиками:

-  Имя.
-  редактируемые свойства.
-  может принимать callback для обработки каждый кадр.
-  может быть расширен (получить больше функций).
-  может быть добавлен к другому узлу как потомок.

.. image:: /img/tree.png

Последнее крайне важно. Узлы могут иметь другие узлы в качестве потомков.
Узлы тогда можно представить как **дерево**.

В Godot, the ability to arrange nodes in this way creates a powerful
tool for organizing projects. Since different nodes have different
functions, combining them allows for creation of more complex functions.

This is probably not clear yet and makes little sense, but everything
will click a few sections ahead. The most important fact to remember for
now is that nodes exist and can be arranged this way.

Сцены
------

.. image:: /img/scene_tree_example.png

Now that the concept of nodes has been defined, the next logical
step is to explain what a Scene is.

A scene is composed of a group of nodes organized hierarchically (in
tree fashion). It has the following properties:

-  Сцена всегда имеет только один корневой узел.
-  Сцены можно сохранить на диск и загрузить с диска.
-  Сцены могут быть *инстанцированы* (об этом чуть позже).
-  Запустить игру значит запустить сцену.
-  В проекте может быть много сцен, но для их запуска, должна быть выбрана 
   для загрузки первой.

В сущности, редактор Godot это **редактор сцены**. Он имеет много инструментов
для редактирования 2D и 3D сцен и интерфейса пользователя, но редактор
основан на концепции редактирования сцены и узлов ее составляющих.

Создание нового проекта
----------------------

Теория скучна, перейдем к практике. Следуя давней традиции туториалов,
первый проект назовем Hello World. Для чего используем редактор.

Когда godot запускается вне проекта, появляется Project Manager.
Что помогает управляться с проектами.

.. image:: /img/project_manager.png

Для создания нового проекта, используем опцию "New Project" . 
Выберите путь для проекта и задайте имя проекта:

.. image:: /img/create_new_project.png

Редактор
------

Когда проект "New Project" создан, следующий шаг - открыть его. 
Его откроет Godot редактор. Вот как выглядит только что открытый редактор

opened:

.. image:: /img/empty_editor.png

Как мы упоминали, создание игр в Godot похоже на готовку, 
так что давайте откроем холодильник и добавим свежих нодов в проект.
Начнем с Hello World! Нажмем кнопку "New Node" (выглядит как символ "плюс"):

.. image:: /img/newnode_button.png

Откроется окно Create Node, появится длинный список узлов доступных для создания:

.. image:: /img/node_classes.png

Выберите здесь, узел "Label" . Быстрее всего наверное в поиске:

.. image:: /img/node_search_label.png

И нконец, создайте Label! A lot happens when Create is pressed:

.. image:: /img/editor_with_label.png

First of all, the scene is changed to the 2D editor (because Label is
a 2D Node type), and the Label appears, selected, at the top left
corner of the viewport.

The node appears in the scene tree editor (box in the top left
corner), and the label properties appear in the Inspector (box on the
right side).

The next step will be to change the "Text" Property of the label, let's
change it to "Hello, World!":

.. image:: /img/hw.png

Ok, everything's ready to run the scene! Press the PLAY SCENE Button on
the top bar (or hit F6):

.. image:: /img/playscene.png

Aaaand... Oops.

.. image:: /img/neversaved.png

Scenes need to be saved to be run, so save the scene to something like
hello.scn in Scene -> Save:

.. image:: /img/save_scene.png

And here's when something funny happens. The file dialog is a special
file dialog, and only allows you to save inside the project. The project
root is "res://" which means "resource path". This means that files can
only be saved inside the project. For the future, when doing file
operations in Godot, remember that "res://" is the resource path, and no
matter the platform or install location, it is the way to locate where
resource files are from inside the game.

After saving the scene and pressing run scene again, the "Hello, World!"
demo should finally execute:

.. image:: /img/helloworld.png

Success!

.. _doc_scenes_and_nodes-configuring_the_project:

Configuring the project
-----------------------

Ok, It's time to do some configuration to the project. Right now, the
only way to run something is to execute the current scene. Projects,
however, have several scenes so one of them must be set as the main
scene. This scene is the one that will be loaded at the time the project
is run.

These settings are all stored in the engine.cfg file, which is a
plaintext file in win.ini format, for easy editing. There are dozens of
settings that can be changed in this file to alter how a project executes,
so to make matters simpler, a project setting dialog exists, which is
sort of a frontend to editing engine.cfg

To access that dialog, simply go to Scene -> Project Settings.

Once the window opens, the task will be to select a main scene. This can
be done easily by changing the application/main_scene property and
selecting 'hello.scn'.

.. image:: /img/main_scene.png

With this change, pressing the regular Play button (or F5) will run the
project, no matter which scene is being edited.

Going back to the project settings dialog. This dialog provides a lot
of options that can be added to engine.cfg, and shows their default
values. If the default value is ok, then there isn't any need to
change it.

When a value is changed, a tick is marked to the left of the name.
This means that the property will be saved to the engine.cfg file and
remembered.

As a side note, for future reference and a little out of context (this
is the first tutorial after all!), it is also possible to add custom
configuration options and read them in run-time using the
:ref:`Globals <class_Globals>` singleton.

To be continued...
------------------

This tutorial talks about "scenes and nodes", but so far there has been
only *one* scene and *one* node! Don't worry, the next tutorial will
deal with that...
