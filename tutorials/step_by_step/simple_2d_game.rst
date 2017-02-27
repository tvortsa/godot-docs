.. _doc_simple_2d_game:

Простая 2D игра
==============

Pong
~~~~

В этом уроке, мы создадим простую иру пинг-понг (Pong). Есть много
более сложных примеров в демо-комплекте движка, но этот
должен ввести вас в основы функционирования 2D Игр.

Начнем с запуска Godot Engine и создания нового проекта.

Активы
~~~~~~

Некоторые активы включены в этот туториал:
:download:`pong_assets.zip </files/pong_assets.zip>`. Распакуйте содержимое
в папку вашего проекта.

Настройка сцены
~~~~~~~~~~~

В память о старых временах, игра будет с разрешением 640x400 пикселей
Это можно сконфигурировать в Project Settings (см
:ref:`doc_scenes_and_nodes-configuring_the_project`) в Scene/Project
settings меню. Цвет фона по умолчанию оставим черный:

.. image:: /img/clearcolor.png

Создайте узел :ref:`class_Node2D` для корневого узла проекта. 
Node2D это базовый тип узлов для 2D движка. После этого добавим спрайты, 
(:ref:`class_Sprite` node) для левой и правой ракеток, разделителя и мяча.
Вы можете задать произвольные имена для каждого узла, и зададим текстуру 
для каждого спрайта в Inspector. 

.. image:: /img/pong_nodes.png

Задаем позиции узлов:
 - "left" node: (67, 183)
 - "right" node: (577, 187)
 - "separator" node: (320, 200)
 - "ball" node: (320, 188)


В итоге макет сцены должен выглядеть так (note: шар посредине!):

.. image:: /img/pong_layout.png


Сохраните сцену как "pong.tscn" и установите ее в качестве главной
сцены проекта.

.. _doc_simple_2d_game-input_actions_setup:

Настройка действий ввода
~~~~~~~~~~~~~~~~~~~

В видео-игры можно играть многими устройствами ввода: Keyboard, Joypad,
Mouse, Touchscreen (multitouch)... Godot может использовать их все.
Но, было бы интересно определить ввод как "Input Actions"
вместо аппаратных действий которыми вы могли бы управлять отдельно. 
Так любой метод ввода может быть использован: каждый будет требовать только
подключения пользователя к кнопкам игровых действий которые вы определили. 

Это Pong. Единственный значимый ввод это движение ракетки вверх-вниз.

Откройте свойства проекта снова (Scene/Project settings), но теперь перейдите
во вкладку

"Input Map".

В этой вкладке добавьте 4 действия:
``left_move_up``, ``left_move_down``, ``right_move_up``,
``right_move_down``.
Назначьте любую клавишу по вашему желанию. A/Z (для левого игрока) и Up/Down
(для правого) должно подойти для большинства ситуаций.

.. image:: /img/inputmap.png

Скрипт
~~~~~~

Создайте скрипт для корневого узла сцены и откройте его (как было показано в
 :ref:`doc_scripting-adding_a_script`). Этот скрипт наследует от Node2D:

::

    extends Node2D

    func _ready():
        pass
        
        
Перво наперво, надо определить некоторых участников для нашего скрипта
которые будут хранить полезные значения. Такие как разрешение экрана,
ракетку и начальное направление шарика.

::

    extends Node2D
    
    # Member variables
    var screen_size
    var pad_size
    var direction = Vector2(1.0, 0.0)

    func _ready():
        pass


Как вы помните, функция ``_ready()`` это первая функция вызываемая
(после ``_enter_tree()`` которая нам тут не нужна). В этой функции,
должно дыть сделано две вещи. Во первых включить процессинг (processing):
это делает функция ``set_process(true)``.
Вторая это инициализировать две наши переменные члены.

::

    extends Node2D

    # Переменные члены
    var screen_size
    var pad_size
    var direction = Vector2(1.0, 0.0)

    func _ready():
        screen_size = get_viewport_rect().size
        pad_size = get_node("left").get_texture().get_size()
        set_process(true)
        
Мы инициировали переменную ``pad_size`` получением одного из узлов ракетки
(левой), и получили ее размер текстуры.  ``screen_size`` это
инициализация с использованием ``get_viewport_rect()`` который возвращает объект Rect
соответствующий окну игры, и мы сохраняем его размер.


Теперь, нам нужно добавить еще несколько членов к нашему скрипту чтобы наш шар перемещался.

::

    extends Node2D

    # Переменные члены
    var screen_size
    var pad_size
    var direction = Vector2(1.0, 0.0)
    
    # Константа скорости ракетки (в пикселях/секунду)
    const INITIAL_BALL_SPEED = 80
    # Скорость мяча (тоже пикселей в секунду)
    var ball_speed = INITIAL_BALL_SPEED
    # Константа для скорости ракеток
    const PAD_SPEED = 150

    func _ready():
        screen_size = get_viewport_rect().size
        pad_size = get_node("left").get_texture().get_size()
        set_process(true)

    

Наконец, функция ``_process()`` . Весь код ниже содержится в этой функции.

Нам нужно инициализировать некоторые полезные для вычислений значения. 
Первая это положение шарика (from the node), второе это прямоугольник
(``Rect2``) для каждой ракетки. Эти прямоугольники будут использоваться
для анализа столкновений между мячом и ракетками. Спрайты по-умолчанию
центрируют свои текстуры, так что нужна небольшая подстройка ``pad_size / 2`` .

::

    func _process(delta):
        var ball_pos = get_node("ball").get_pos()
        var left_rect = Rect2( get_node("left").get_pos() - pad_size*0.5, pad_size )
        var right_rect = Rect2( get_node("right").get_pos() - pad_size*0.5, pad_size )

Теперь добавим немного движения мячу в функции ``_process()`` .
Поскольку положение мяча сохранено в переменной ``ball_pos``,
интегрирование проста:

::

        # Интегрирование нового положения мяча
        ball_pos += direction * ball_speed * delta

Эта строка кода вызывается каждую итерацию функции ``_process()``.
Это значит что положение мяча будет обновлено для каждого следующего кадра.

Now that the ball has a new position, we need to test if it
collides with anything, that is the window borders and the pads. First,
the floor and the roof:

::

        # Flip when touching roof or floor
        if ((ball_pos.y < 0 and direction.y < 0) or (ball_pos.y > screen_size.y and direction.y > 0)):
            direction.y = -direction.y

Second, the pads: if one of the pads is touched, we need to invert the
direction of the ball on the X axis so it goes back, and define a new
random Y direction using the ``randf()`` function. We also increase its
speed a little.

::

        # Flip, change direction and increase speed when touching pads
        if ((left_rect.has_point(ball_pos) and direction.x < 0) or (right_rect.has_point(ball_pos) and direction.x > 0)):
            direction.x = -direction.x
            direction.y = randf()*2.0 - 1
            direction = direction.normalized()
            ball_speed *= 1.1

Finally, if the ball went out of the screen, it's game over. That is, we test if
the X position of the ball is less than 0 or greater than the screen
width. If so, the game restarts:

::

        # Check gameover
        if (ball_pos.x < 0 or ball_pos.x > screen_size.x):
            ball_pos = screen_size*0.5
            ball_speed = INITIAL_BALL_SPEED
            direction = Vector2(-1, 0)

Once everything is done, the node is updated with the new position of
the ball, which was computed before:

::

        get_node("ball").set_pos(ball_pos)

Next, we allow the pads to move. We only update their position according
to player input. This is done using the Input class:

::

        # Move left pad
        var left_pos = get_node("left").get_pos()

        if (left_pos.y > 0 and Input.is_action_pressed("left_move_up")):
            left_pos.y += -PAD_SPEED * delta
        if (left_pos.y < screen_size.y and Input.is_action_pressed("left_move_down")):
            left_pos.y += PAD_SPEED * delta

        get_node("left").set_pos(left_pos)

        # Move right pad
        var right_pos = get_node("right").get_pos()

        if (right_pos.y > 0 and Input.is_action_pressed("right_move_up")):
            right_pos.y += -PAD_SPEED * delta
        if (right_pos.y < screen_size.y and Input.is_action_pressed("right_move_down")):
            right_pos.y += PAD_SPEED * delta

        get_node("right").set_pos(right_pos)
        
We use the four actions previously defined in the Input actions setup
section above. When the player activates the respective key, the
corresponding action is triggered. As soon as this happens, we simply
compute a new position for the pad in the desired direction and apply it
to the node.

That's it! A simple Pong was written with a few lines of code.
