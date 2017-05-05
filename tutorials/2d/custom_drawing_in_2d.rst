.. _doc_custom_drawing_in_2d:

Собственное рисование в 2D
==========================

Зачем?
------

Godot обладает узлами для отрисовки спрайтов, полигонов, частиц, и
всего остального. Для большинства ситуаций этого достаточно, но не всегда.
Если что-то из того что вам нужно не поддерживается, не расстраивайтесь
из-за того что нет узла делающего-то-что-вам-надо... 
Все можно нарисовать просто создав 2D node (будь это
:ref:`Control <class_Control>` или :ref:`Node2D <class_Node2D>`
based) и задав пользовательские команды рисования. 
Это *действительно* не сложно.

Но...
------

Пользовательский рисунок вручную в узле *реально* здорово. И вот почему:

-  Отрисовка форм или логики которая не обрабатывается узлами (пример: 
   создание узла который рисует круг, изображение со следом (типа хвост кометы),
   особые случаи анимированных полигонов, и т.п.).
-  Визуализации, которые не совместимы с узлами: (пример: доска тетриса).
   Пример тетриса использует собственную функцию отрисовки блоков.
-  Управление логикой рисования большого количества простых объектов
   (сотни и тысячи). Использование тысяч узлов далеко не так эффективно
   как отрисовка, но тысяча вызовов отрисовки это не много.
   Посмотрите пример демо "Shower of Bullets".
-  Создание собственного UI control. Есть много готовых элементов управления,
   Но можно легко столкнуться с необходимостью создания нового, нестандартного.

OK, как?
--------

Добавьте скрипт к любому :ref:`CanvasItem <class_CanvasItem>`
derived node, типа :ref:`Control <class_Control>` или
:ref:`Node2D <class_Node2D>`. Замените функцию _draw().

::

    extends Node2D

    func _draw():
        #ваши собственные команды отрисовки
        pass

Команды рисования описаны здесь :ref:`CanvasItem <class_CanvasItem>`.
Их много.

Обновление
----------

Функция _draw() вызывается лишь однажды, затем команды рисования
кэшируются и запоминаются, поэтому повторные вызовы не нужны.

Если нужно пере-рисовать потому что что-то изменилось,
просто вызовите :ref:`CanvasItem.update() <class_CanvasItem_update>`
в этом же узле и произойдет новый вызов _draw().

Вот немного более сложный пример. Текстурная переменная, которая будет
перерисовываться при изменении:

::

    extends Node2D

    export var texture setget _set_texture

    func _set_texture(value):
        #если текстурная переменная изменена извне,
        #вызываем этот коллбэк.
        texture=value #текстура была изменена
        update() #обновление этого узла

    func _draw():
        draw_texture(texture,Vector2())

Иногда бывает нужно перерисовывать в каждом кадре. Для этого, просто
вызывайте update() из коллбэка _process() , вот так:

::

    extends Node2D

    func _draw():
        #your draw commands here
        pass

    func _process(delta):
        update()

    func _ready():
        set_process(true)

Например: рисуем круговую арку
-------------------------------

Теперь мы будем использовать пользовательские функции рисования Godot Engine для отрисовки чего в Godot нет готовой функции. Как например, Godot имеет функцию draw_circle() которая рисует полный круг. Но, что если вам нужна только часть круга? Для этого мы напишем собственный код и функцию, и сами отрисуем все.

Функция дуги
^^^^^^^^^^^^


Дуга определяется своими параметрами опорного круга, то есть: позиция центра, и радиус. А также значение угла с которого она начинает рисоваться и угла на котором заканчивается. Вот те 4 параметра которые понадобяться нашей функции отрисовки. Мы также предоставим значение цвета, чтобы можно было рисовать дугу разным цветом.

В основе своей, рисование фигур на экране требует ее разложения на определенное количество точек выстроенных одна за другой. Чем больше точек требуется вашей фоме тем более гладкой она выглядит, но тем сложнее процессору ее вычислять. Обычно, если форма огромная (или в 3D, ближе к камере), тем больше точек она потребует без учета вида под углом. С другой стороны, если ваша форма маленькая (или в 3D,  далеко от камеры), вы можете сократить количество точек и удешевить отрисовку формы. Это называют уровнем детализации *Level of Detail (LoD)*. В нашем примере, мы будем просто использовать фиксированное количество точек, вне зависимости от радиуса.

::

    func draw_circle_arc( center, radius, angle_from, angle_to, color ):
        var nb_points = 32
        var points_arc = Vector2Array()
    
        for i in range(nb_points+1):
            var angle_point = angle_from + i*(angle_to-angle_from)/nb_points - 90
            var point = center + Vector2( cos(deg2rad(angle_point)), sin(deg2rad(angle_point)) ) * radius
            points_arc.push_back( point )
    
        for indexPoint in range(nb_points):
            draw_line(points_arc[indexPoint], points_arc[indexPoint+1], color)

Помните про точки на которые нам надо было разложить форму? Мы зададим их количество в переменной nb_points используя фиксированное значение - 32. Затем инициализирцем пустой массив Vector2Array, это просто массив типа Vector2.

Следующим шагом будет вычисление реальных координат этих 32 точек составляющих нашу дугу. Это выполняется в первом цикле for: мы проходим по всем точкам позиции которых хотим вычислить, плюс одна последняя точка. Сперва мы вычисляем угол каждой точки между стартовым и конечным углами. 

Причина, почему каждый угол уменьшается на 90° в том, что мы будем вычислять 2D позиции для каждого угла с помощью тригонометрии (ну, знаете, синус-косинус...). Но чтобы упростить, cos() и sin() используют радианы, не градусы. Угол в 0° (0 радиан) начинается на три часа, а мы хотим начать отсчет от ноль часов. Поэтому мы просто сокращаем каждый угол на 90° чтобы отсчет шел от ноля часов.

Фактическое положение точки, расположенной по кругу под углом 'angle' (в радианах) задается в Vector2(cos(angle), sin(angle)). Поскольку cos() и sin() возвращают значение между -1 и 1, позиция задается на круге с радиусом 1. Чтобы получить позицию на заданном опорном круге, радиус которого задан в 'radius', нужно просто умножить позицию на 'radius'. Наконец, нам нужно расположитьнаш опорный круг в позицию 'center', что достигается сложением его с нашим значением Vector2. Наконец, мы вставим точку в Vector2Array который объявили ранее.

Теперь нужно отрисовать наши точки. Как вы понимаете, мы не можем просто отрисовать 32 точки: нам нужно отрисовать и все, что между каждой из них. Мы могли бы вычислить каждую точку сами используя предыдущий способ, и отрисовать их по-одной, но это очень сложно и не эффективно (кроме случаев когда вам именно это и нужно). Итак, мы просто отрисуем прямые линии между каждой парой точек. Unless the radius of our support circle is very big, the length of each line between a pair of points will never be long enough to see them. If this happens, we simply would need to increase the number of points.

Отрисовка дуги на экране
^^^^^^^^^^^^^^^^^^^^^^^^
Теперь у нас есть функция которая рисует на экране: настало время вызвать ее в функции _draw().

::

    func _draw():
        var center = Vector2(200,200)
        var radius = 80
        var angle_from = 75
        var angle_to = 195
        var color = Color(1.0, 0.0, 0.0)
        draw_circle_arc( center, radius, angle_from, angle_to, color )

Result:

.. image:: /img/result_drawarc.png



Функция полигональной дуги
^^^^^^^^^^^^^^^^^^^^^^^^^^
Мы можем сделать еще один шаг и написать функцию, которая рисует гладкую часть диска, определяемую дугой, а не только ее форму. Метод точно такой же, как и ранее, за исключением того, что мы рисуем многоугольник вместо линий:

::

    func draw_circle_arc_poly( center, radius, angle_from, angle_to, color ):
        var nb_points = 32
        var points_arc = Vector2Array()
        points_arc.push_back(center)
        var colors = ColorArray([color])
    
        for i in range(nb_points+1):
            var angle_point = angle_from + i*(angle_to-angle_from)/nb_points - 90
            points_arc.push_back(center + Vector2( cos( deg2rad(angle_point) ), sin( deg2rad(angle_point) ) ) * radius)
        draw_polygon(points_arc, colors)
        
        
.. image:: /img/result_drawarc_poly.png

Dynamic custom drawing
^^^^^^^^^^^^^^^^^^^^^^
Alright, we are now able to draw custom stuff on screen. However, it is very static: let's make this shape turn around the center. The solution to do this is simply to change the angle_from and angle_to values over time. For our example, we will simply increment them by 50. This increment value has to remain constant, else the rotation speed will change accordingly.

First, we have to make both angle_from and angle_to variables global at the top of our script. Also note that you can store them in other nodes and access them using get_node().

::

 extends Node2D

 var rotation_ang = 50
 var angle_from = 75
 var angle_to = 195



We make these values change in the _process(delta) function. To activate this function, we need to call set_process(true) in the _ready() function. 

We also increment our angle_from and angle_to values here. However, we must not forget to wrap() the resulting values between 0 and 360°! That is, if the angle is 361°, then it is actually 1°. If you don't wrap these values, the script will work correctly but angles values will grow bigger and bigger over time, until they reach the maximum integer value Godot can manage (2^31 - 1). When this happens, Godot may crash or produce unexpected behavior. Since Godot doesn't provide a wrap() function, we'll create it here, as it is relatively simple.

Finally, we must not forget to call the update() function, which automatically calls _draw(). This way, you can control when you want to refresh the frame.

::

 func _ready():
     set_process(true)
 
 func wrap(value, min_val, max_val):
     var f1 = value - min_val
     var f2 = max_val - min_val
     return fmod(f1, f2) + min_val

 func _process(delta):
     angle_from += rotation_ang
     angle_to += rotation_ang
     
     # we only wrap angles if both of them are bigger than 360
     if (angle_from > 360 && angle_to > 360):
         angle_from = wrap(angle_from, 0, 360)
         angle_to = wrap(angle_to, 0, 360)
     update()

Also, don't forget to modify the _draw() function to make use of these variables:
::

 func _draw():
	var center = Vector2(200,200)
	var radius = 80
	var color = Color(1.0, 0.0, 0.0)

	draw_circle_arc( center, radius, angle_from, angle_to, color )

Let's run!
It works, but the arc is rotating insanely fast! What's wrong?

The reason is that your GPU is actually displaying the frames as fast as he can. We need to "normalize" the drawing by this speed. To achieve, we have to make use of the 'delta' parameter of the _process() function. 'delta' contains the time elapsed between the two last rendered frames. It is generally small (about 0.0003 seconds, but this depends on your hardware). So, using 'delta' to control your drawing ensures your program to run at the same speed on every hardware.

In our case, we simply need to multiply our 'rotation_ang' variable by 'delta' in the _process() function. This way, our 2 angles will be increased by a much smaller value, which directly depends on the rendering speed.

::

 func _process(delta):
     angle_from += rotation_ang * delta
     angle_to += rotation_ang * delta
     
     # we only wrap angles if both of them are bigger than 360
     if (angle_from > 360 && angle_to > 360):
         angle_from = wrap(angle_from, 0, 360)
         angle_to = wrap(angle_to, 0, 360)
     update()

Let's run again! This time, the rotation displays fine!

Tools
-----

Drawing your own nodes might also be desired while running them in the
editor, to use as preview or visualization of some feature or
behavior.

Remember to just use the "tool" keyword at the top of the script
(check the :ref:`doc_gdscript` reference if you forgot what this does).
