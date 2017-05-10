.. _doc_particle_systems_2d:

Системы частиц (2D)
=====================

Введение
--------

В поставке идет простая (но достаточно гибкую для большинства применений) 
система частиц. Частицы используют для симуляции комплексных физических
эффектов таких как искры, огонь, волшебная пыль, дым, mist, и т.п.

Идея в том что "частица" испускаются за определенный промежуток времени
с заданной продолжительностью жизни. Во время жизни, каждая частица обладает 
одинаковым базовым поведением. Каждую частицу делает уникальной и обеспечивает
более органичный вид - "случайность" ассоциированная с каждым параметром. 
В сущности, создание системы частиц означает установку базовых физических
параметров и затем добавление им случайности.

Particles2D
~~~~~~~~~~~

Системы частиц добавляются в сцену узлом
:ref:`Particles2D <class_Particles2D>`. Они включены по-умолчанию
И запускают эмиссию частиц в виде падающих белых точек (под действием гравитации).
Это разумная отправная точка для последующих настроек тоак как вам надо.

.. image:: /img/particles1.png

Текстура
~~~~~~~~

Системы частиц используют одну текстуру (в будущем это возможно будет
расширено анимацией через spritesheet). Текстуру задают через соответствующее свойство:

.. image:: /img/particles2.png

Физические переменные
---------------------

Перед тем как начать разговор о глобальных переменных для систем частиц,
давайте сначала посмотрим, что произойдет, когда физические параметры будут изменены.

Direction - направление
-----------------------

Это базовый угол направления эмиссии. По - умолчанию равен 0 (вниз):

.. image:: /img/paranim1.gif

Изменяя этот угол, изменяется направление эмиссии но гравитация
продолжит свое воздействие:

.. image:: /img/paranim2.gif

Этот параметр полезен потому что, вращая узел, гравитация тоже будет вращаться.
Изменение направления держит их отдельно.

Spread-распространение
----------------------

Spread это угол в пределах которого частицы будут случайно испускаться.
Увеличение spread увеличивает этот угол. Spread равное 180 будет испускать
частицы во всех направлениях.

.. image:: /img/paranim3.gif

Линейная скорость
-----------------

Linear velocity это скорость с которой испускаются частицы (в
пикселей в секунду). Скорость впоследствии может изменяться 
гравитацией или accelerations (как описано ниже).

.. image:: /img/paranim4.gif

Скорость вращения
-----------------

Spin velocity это скорость с которой частицы вращаются вокруг своего центра
(в градусах/чекунду).

.. image:: /img/paranim5.gif

Орбитальная скорость
--------------------

Orbit velocity используется для того, чтобы частицы вращались вокруг их центра.

.. image:: /img/paranim6.gif

Направление и сила гравитации
-----------------------------

Gravity имеет направление и силу. Гравитация применяется к каждой
живущей на данный момент частице.

.. image:: /img/paranim7.gif

Радиальное ускорение
--------------------

Если это ускорение положительное, частицы ускоряются от их центра. 
Если отрицательное, они поглощаются ею.

.. image:: /img/paranim8.gif

Тангенциальное ускорение
------------------------

Это ускорение использует tangent vector к центру. Совместно с
radial acceleration может давать хороший эффект.

.. image:: /img/paranim9.gif

Damping-демпфирование
---------------------

Damping применяет трение к частицам, заставляя их останавливаться. Это
особенно полезно для искр и взрывов, которые обычно имеют высокую
линейную скоростьy и затем останавливаются по мере падения.

.. image:: /img/paranim10.gif

Начальный угол
--------------

Задает начальный угол частиц (в градусах). Этот параметр
в основном полезно рандомизировать.

.. image:: /img/paranim11.gif

Начальный и финальный размер
----------------------------

Determines the initial and final scales of the particle.

.. image:: /img/paranim12.gif

Цвет фазы
---------

Particles can use up to 4 color phases. Each color phase can include
transparency.

Phases must provide an offset value from 0 to 1, and always in
ascending order. For example, a color will begin at offset 0 and end
in offset 1, but 4 colors might use different offsets, such as 0, 0.2,
0.8 and 1.0 for the different phases:

.. image:: /img/particlecolorphases.png

Will result in:

.. image:: /img/paranim13.gif

Глобальные параметры
--------------------

These parameters affect the behavior of the entire system.

Время жизни
-----------

The time in seconds that every particle will stay alive. When lifetime
ends, a new particle is created to replace it.

Lifetime: 0.5

.. image:: /img/paranim14.gif

Lifetime: 4.0

.. image:: /img/paranim15.gif

Timescale
---------

It happens often that the effect achieved is perfect, except too fast or
too slow. Timescale helps adjust the overall speed.

Timescale everything 2x:

.. image:: /img/paranim16.gif

Preprocess
----------

Particle systems begin with 0 particles emitted, then start emitting.
This can be an inconvenience when just loading a scene and systems like
a torch, mist, etc begin emitting the moment you enter. Preprocess is
used to let the system process a given amount of seconds before it is
actually shown the first time.

Emit timeout
------------

This variable will switch emission off after given amount of seconds
being on. When zero, itś disabled.

Offset
------

Allows to move the emission center away from the center

Half extents
------------

Makes the center (by default 1 pixel) wider, to the size in pixels
desired. Particles will emit randomly inside this area.

.. image:: /img/paranim17.gif

It is also possible to set an emission mask by using this value. Check
the "Particles" menu on the 2D scene editor viewport and select your
favorite texture. Opaque pixels will be used as potential emission
location, while transparent ones will be ignored:

.. image:: /img/paranim19.gif

Local space
-----------

By default this option is on, and it means that the space that particles
are emitted to is contained within the node. If the node is moved, all
particles are moved with it:

.. image:: /img/paranim20.gif

If disabled, particles will emit to global space, meaning that if the
node is moved, the emissor is moved too:

.. image:: /img/paranim21.gif

Explosiveness
-------------

If lifetime is 1 and there are 10 particles, it means every particle
will be emitted every 0.1 seconds. The explosiveness parameter changes
this, and forces particles to be emitted all together. Ranges are:

-  0: Emit all particles together.
-  1: Emit particles at equal interval.

Values in the middle are also allowed. This feature is useful for
creating explosions or sudden bursts of particles:

.. image:: /img/paranim18.gif

Randomness
----------

All physics parameters can be randomized. Random variables go from 0 to
1. the formula to randomize a parameter is:

::

    initial_value = param_value + param_value*randomness
