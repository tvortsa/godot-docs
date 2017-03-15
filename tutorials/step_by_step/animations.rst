.. _doc_animations:

Анимации
==========

Введение
------------

Этот урок раскрывает то как все в Godot анимируется. 
Система анимации Godot очень мощная и гибкая.

Для начала, давайте просто используем сцену из предыдущего урока
(:ref:`doc_splash_screen`).
Задача в том чтобы, добавить ей простую анимацию.Вот копия на всякий случай:
:download:`robisplash.zip </files/robisplash.zip>`.

Создание анимации
----------------------

Прежде всего, добавим узел :ref:`AnimationPlayer <class_AnimationPlayer>`
в сцену как потомок "bg" (root node):

.. image:: /img/animplayer.png

Когда выбран узел этого типа, появляется панель animation editor:

.. image:: /img/animpanel.png

Нажмите кнопку new animation и назовите анимацию "intro".

.. image:: /img/animnew.png

После того как анимация была создана, отредактируем ее, нажав кнопку "edit":

.. image:: /img/animedit.png

Редактирование анимации
---------------------

При нажатии кнопки "edit" происходит несколько вещей, во первых
над панелью animation появляется animation editor.
(В Godot 2.x, эту кнопку удалили, вмсто нее кликайте 'animation' toggle справа внизу)

.. image:: /img/animeditor.png

Во вторых, самое главное, это то что property editor входит в режим
"animation editing". В этом режиме, возле каждого свойства в property editor
появляется иконка ключа. Это значит что, в Godot, *любое
свойство любого объекта* может быть анимировано:

.. image:: /img/propertykeys.png

Появление лого
----------------------

Лого будет появляться сверху экрана. После выделения animation player,
editor panel остается видимой пока не будет спрятана вручную
(или узел animation будет удален). Пользуясь этим,
выделите узел "logo" и перейдите к свойству "pos" , переместите его вверх
в позицию: 114,-400.

Once in this position, press the key button next to the property:

.. image:: /img/keypress.png

As the track is new, a dialog will appear asking to create it. Confirm
it!

.. image:: /img/addtrack.png

Ключевой кадр будет добавлен в рдактор animation player:

.. image:: /img/keyadded.png

Передвиньте курсор редвктора в конец, кликнув здесь:

.. image:: /img/move_cursor.png

Измените положение logo в 114,0 и снова добавьте ключевой кадр. С двуня ключами
произойдет анимация.

.. image:: /img/animation.png

Нажмите Play на animation panel произойдет спуск логотипа. Для проверки
запуска этого при загрузке сцены, кнопкой autoplay button можно пометить анимацию
чтобы анимация проигрывалась при старте сцены:

.. image:: /img/autoplay.png

And finally, when running the scene, the animation should look like
this:

.. image:: /img/out.gif
