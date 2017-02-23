.. _doc_instancing_continued:

Инстанцирование (продолжение)
======================

Резюме
-----

Мнстанцирование имеет множество удобных применений. На первый взгляд, с ним мы имеем:

-  Возможность подразделять сцену и упрощать управление ею.
-  Более гибкую альтернативу префабам (и гораздо более мощную,
   учитывая что экземпляры работают на многих уровнях).
-  Способ разработки более сложных игровых процессов даже UI
   (UI элементы в Godot тоже узлы).

Язык дизайна
---------------

Но подлинная мощь инстанцирования сцен в том, что это является прекрасным
языком дизайна. Это существенное отличие Godot от большинства других движков.
Весь движок был разработан с нуля вокруг этой концепции.

Создавая игры в Godot, рекомендуется отбросить другие паттерны проектирования
такие как MVC или диаграммы Сущность-Отношения и начать думать об игре
в более естественном ключе. Начните представлять визуальные элементы в игре, 
которые могут быть представлены не только программистом а кем угодно.

Вот например как можно представить простой шутер:

.. image:: /img/shooter_instancing.png

С такой диаграммой довольно просто представить почти любой тип игр.
Просто записывайте те элементы которые приходят в голову,
а затем стрелки которые определяют принадлежность.

Когда такая схема существует, создание игры это создание сцены
для каждого узла в диаграмме, и использование инстанцирования 
(кодом или редактором) для представления связей.

Большую часть времени программирования игр (и вообще программ) тратится
на разработку архитектуры fitting game components to that
architecture. Designing based on scenes replaces that and makes
development much faster and more straightforward, allowing to
concentrate on the game itself. Scene/Instancing based design is
extremely efficient at saving a large part of that work, since most of
the components designed map directly to a scene. This way, none or
little architectural code is needed.

The following is a more complex example, an open-world type of game with
lots of assets and parts that interact:

.. image:: /img/openworld_instancing.png

Make some rooms with furniture, then connect them. Make a house later,
and use those rooms as the interior.

The house can be part of a citadel, which has many houses. Finally the
citadel can be put on the world map terrain. Add also guards and other
NPCs to the citadel by previously creating their scenes.

With Godot, games can grow as quickly as desired, as all it needs is
to create and instance more scenes. The editor UI is designed to be
operated by non programmers too, so a typical team development process
can involve 3D or 2D artists, level designers, game designers, animators,
etc all working with the editor interface.

Information overload!
---------------------

Do not worry too much, the important part of this tutorial is to create
awareness on how scenes and instancing are used in real life. The best
way to understand all this is to make some games.

Everything will become very obvious when put to practice, so please do
not scratch your head and go on to the next tutorial!
