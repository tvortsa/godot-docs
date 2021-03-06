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

Для автозагрузки сцены или скрипта, выберите Scene > Project Settings из меню и переключитесб на
вкладку AutoLoad. Каждой записи в списке требуется имя, которое используется как имя узла, 
и узел всегда добавляется к root viewport до загрузки любой другой сцены.

.. image:: /img/singleton.png

Это означает что любой узел всегда будет иметь доступ к синглтону с именем "playervariables" с:

::

   var player_vars = get_node("/root/playervariables")

Собственный переключатель сцен
---------------------

Это краткое руководство по тому как создать свой переключатель сцен используя
autoload. Для простого переключения сцен, достаточно метода 
:ref:`SceneTree.change_scene() <class_SceneTree_change_scene>`
 (описан в :ref:`doc_scene_tree`), так что этот метод предназначен
 для более сложного поведения при переключении между сценами.

Сперва загрузите шаблон отсюда:
:download:`autoload.zip </files/autoload.zip>`, then open it.

Там есть две сцены, scene_a.scn и scene_b.scn или пустой проект.
Они идентичны и каждая содержит кнопку, связанную с обратным вызовом
для переключения на другую сцену. При запуске проекта, стартует сцена
 scene_a.scn. Но, пока она ничего не делает и нажатие на кнопку
 ни к чему не приводит.

global.gd
---------

Прежде всего, создадим скрипт global.gd. Простой способ создания
ресурса с нуля - кнопкой нового ресурса на вкладке инспектора:

.. image:: /img/newscript.png

Сохраните скрипт как `global.gd`:

.. image:: /img/saveasscript.png

Скрипт должен открыться в script editor. Следующий шаг это добавление его
к списку AutoLoad. Выделите Scene > Project Settings из меню,
переключитесь на панель AutoLoad и добавьте новую запись с именем "global" 
указывающую на этот файл:

.. image:: /img/addglobal.png

Теперь, когда бы вы ни запустили любую из ваших сцен, скрипт всегда будет загружаться.

Вернемся к нашему скрипту, текущая сцена должна быть выбрана функцией 
`_ready()` . И текущая сцена и `global.gd` являются потомками
root, но узлы autoloaded всегда первее. Это значит что последний потомок
у root всегда является загруженной сценой.

Note: Убедитесь что global.gd расширяет Node, иначе он не будет загружен!

::

    extends Node

    var current_scene = null

    func _ready():
            var root = get_tree().get_root()
            current_scene = root.get_child( root.get_child_count() -1 )

Далее идет функция переключения сцены. Эта функция освобождает текущую сцену
и заменяет ее запрошенной.

::

    func goto_scene(path):

        # Эта функция обычно вызывается из обратного вызова сигнала,
        # или из какой-то другой функции из текущей сцены.
        # Удаление текущей сцены в этом месте может быть
        # плохой идеей, поскольку она может находится внутри обратного вызова или его функции.
        # В худшем случае произойдет крушение с неопределенными последствиями.

        # Выход из ситуации в отложенной загрузке, 
        # на момент когда будет гарантия того что не запущен никакой код из данной сцены:

        call_deferred("_deferred_goto_scene",path)


    func _deferred_goto_scene(path):

        # немедленное освобождение текущей сцены,
        # здесь нет риска.    
        current_scene.free()

        # Загружаем тщвую сцену
        var s = ResourceLoader.load(path)

        # Инстанцируем новую сцену
        current_scene = s.instance()

        # Добавляем ее к активной сцене, как потомк у root
        get_tree().get_root().add_child(current_scene)

        # опционально, делаем ее совместимой с API SceneTree.change_scene() 
        get_tree().set_current_scene( current_scene )

Как упомянуто в комментариях выше, мы хотим избежать ситуации
при которой удаляется сцена которая все еще используется
(код ее функций все еще выполняется), так что использование
:ref:`Object.call_deferred() <class_Object_call_deferred>`
здесь желательно.Результатом является то, что выполнение команд
Во второй функции произойдет в более позднее время, когда 
никакой код текущей сцены не запущен.

Наконец, осталось только заполнить пустые функции в scene_a.gd
и scene_b.gd:

::

    #добавить к scene_a.gd

    func _on_goto_scene_pressed():
            get_node("/root/global").goto_scene("res://scene_b.scn")

и

::

    #добавить в scene_b.gd

    func _on_goto_scene_pressed():
            get_node("/root/global").goto_scene("res://scene_a.scn")

Теперь если вы щапустите проект, то сможете переключаться между сценами
нажимая на кнопку!

Для загрузки сцен с прогресс-баром, посмотрите следующий урок,
:ref:`doc_background_loading`
