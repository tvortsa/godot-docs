.. _doc_introduction_to_3d:

Введение в 3D
==================

Создание 3D игры может быть сложным. Эта дополнительная Z координата
делает то что было просто в 2D играх - сложнее.
Чтобы помочь в этом переходе, Godot использует схожие API и для 2D и для 3D.
Большинство узлов одинаковы и присутствуют и в 2D и в 3D версиях.
Фактически стоит проверить урок по 3D platformer , или 3D kinematic character tutorials,
которые почти идентичны их 2D вариантам.

В 3D, немного более сложная чем в 2D, так что сперва посмотрите
:ref:`doc_vector_math` в wiki (статья специально создана для
разработчиков игр, не математико или инженеров) Поможет проложить путь в
эффективную разработку 3D игр.

Spatial node
~~~~~~~~~~~~

:ref:`Node2D <class_Node2D>` базовый узел для 2D.
:ref:`Control <class_Control>` базовый узел для всего в GUI.
Следуя этой логике, 3D engine использует :ref:`Spatial <class_Spatial>`
node для всего в 3D.

.. image:: /img/tuto_3d1.png

Spatial nodes обладает локальными трансформциями, которые относятся
к родительскому узлу (до тех пор, пока родительский узел тоже является **или наследует** 
типа Spatial). Эти трансформации доступны как 4x3
:ref:`Transform <class_Transform>`, или как 3 :ref:`Vector3 <class_Vector3>`
члена представляющие расположение, эйлеровское вращение (x,y и z углы) и
масштабирование.

.. image:: /img/tuto_3d2.png

3D content
~~~~~~~~~~

В отличие от 2D, где загрузка содержимого изображения и рисования проста,
3D немного сложнее. Контент должен быть создан в специальной программе
3D редакторе (обычно называемый DCCs) и экспортировани в формат обмена
чтобы быть импортированным в Godot (3D форматы не стандартизованы как
форматы изображений).

DCC-создание моделей
------------------

Есть два пайплайна для импорта моделей 3D в Godot. Первый и наиболее используемый
через импортер :ref:`doc_importing_3d_scenes` который позволяет импортировать
сцену целиком (так же, как они выглядят в DCC), включая анимацию,
скелетные оснастки, blend shapes, и т.п.

Второй способ - через импортер :ref:`doc_importing_3d_meshes`. 
позволяет просто импортировать каркасы в .OBJ файлах,
которые затем могут быть помещены в узел :ref:`MeshInstance <class_MeshInstance>`
nдля отображения.

Генерация геометрии
------------------

Можно создавать и кастомную геометрию используя ресурс
:ref:`Mesh <class_Mesh>` напрямую, просто создавать массивы
и используя функцию :ref:`Mesh.add_surface() <class_Mesh_add_surface>`.
Также доступен вспомогательный класс, :ref:`SurfaceTool <class_SurfaceTool>`,
который обеспечивает более прямолинейный API и помощники для индексирования,
создания нормалей, касательных и т.п.

В любом случае, этот метод предназначен для создания статической геометрии 
(модели, которые не будут обновляться часто), поскольку создание массивов вершин и
и представления их в 3D API требуют значительных ресурсов.

Быстрая геометрия
------------------

Если, нужна простая часто изменяемая геометрия, Godot предоставляет
специяльный узел :ref:`ImmediateGeometry <class_ImmediateGeometry>`
в формате стиля OpenGL 1.x immediate-mode API для создания точек,
линий, треугольников, и т.п.

2D в 3D
--------

While Godot packs a powerful 2D engine, многие типы игр используют 2D в
3D окружении. Используя фиксированную камеру (ортогональную или
перспективную) которая не вращатся, такие узлы как
:ref:`Sprite3D <class_Sprite3D>` и
:ref:`AnimatedSprite3D <class_AnimatedSprite3D>`
могут быть использованы для создания 2D игр используя преимущество 
смешения с 3D фоном, более реалистичным параллаксом, освещением/тенями эффектами, и т.п.

Недостатком является, дополнительная сложность и потеря производительности
в сравнении с plain 2D, а также потеря возможности работы с пикселями.

Окружение
~~~~~~~~~~~

Помимо редактирования сцены, часто необходимо редактирование окружения.
Godot предлагает узел :ref:`WorldEnvironment <class_WorldEnvironment>`
который позволяет изменять цвет фона, режим (as in, put a
skybox), и применять несколько типов встроенных эффектов пост-обработки.
Окружение также может быть переопределено в Camera.

3D вьюпорт
~~~~~~~~~~~

Редактирование 3D сцен происходит во вкладке 3D. Эту вкладку можно
выбрать вручную, но она также будет автоматически активирована при
выборе Spatial узла.

.. image:: /img/tuto_3d3.png

Дефолтная навигация в 3D сцене похожа на ту что в Blender (aiming to
have some sort of consistency in the free software pipeline..), but
options are included to customize mouse buttons and behavior to be
similar to other tools in Editor Settings:

.. image:: /img/tuto_3d4.png

Система координат
-----------------

Godot использует `metric <http://en.wikipedia.org/wiki/Metric_system>`__
для всего. 3D Физика и другие области настроены на это, так что
испоьзовать другие системы обычно плохая идея (только если вы не
уверены в том что знаете что делаете).

Работая с 3D активами, всегда лучше работать в корректном масштабе
(установите ваш DCC в метры). Godot allows scaling post-import and,
while this works in most cases, in rare situations it may introduce
floating point precision issues (and thus, glitches or artifacts) in
delicate areas such as rendering or physics. So, make sure your artists
always work in the right scale!

Координата Y используется как направление "вверх", though for most objects that need
alignment (like lights, cameras, capsule collider, vehicle, etc.), the Z
axis is used as a "pointing towards" direction. Грубо говоря:

-  **X** это всторону
-  **Y** это верх/низ
-  **Z** это пере/назад

Space и гизмо манипуляций
-----------------------------

Перемещение объектов в 3D виде осуществляется с помощью гизмо трансформаций.
Каждая ось представлена цветом: Красный, Зеленый, Голубой соответствует X,Y,Z.
respectively. This convention applies to the grid and other gizmos too
(а также к языку шейдинга, для компонентов Vector3,Color, и т.п.).

.. image:: /img/tuto_3d5.png

Некоторые полезные хоткеи:

-  Для привязки вращения или перемещения, нажмите "s" во время вращения/перемещения.
-  Для центрирования вида по объекту, нажмите "f".

Меню View
---------

Опции вида управляются в меню "[ view ]". Обратите внимание на
это маленькое меню внутри окна поскольку оно часто упускается из виду!

.. image:: /img/tuto_3d6.png

Дефолтное освещение
----------------

3D вид имеет некоторые параметры освещения по умолчанию:

-  Есть направленный свет, который делает объекты видимыми во время
   редактирования включенный по-умолчанию. Он отключается при запуске игры.
-  There is subtle default environment light to avoid places not reached
   by the light to remain visible. Он также отключается при запуске игры
   (и когда дефолтный свет отключают).

Его можно отключить опцией "Default Light":

.. image:: /img/tuto_3d8.png

Настройка этого (и другие дефолтные view опции) таже доступны через
меню settings:

.. image:: /img/tuto_3d7.png

Которое открывает это окно, Позволяя настраивать цвет и
дефолтное направление света:

.. image:: /img/tuto_3d9.png

Камеры
-------

No matter how many objects are placed in 3D space, nothing will be
displayed unless a :ref:`Camera <class_Camera>` is
also added to the scene. Cameras can either work in orthogonal or
perspective projections:

.. image:: /img/tuto_3d10.png

Cameras are associated and only display to a parent or grand-parent
viewport. Since the root of the scene tree is a viewport, cameras will
display on it by default, but if sub-viewports (either as render target
or picture-in-picture) are desired, they need their own children cameras
to display.

.. image:: /img/tuto_3d11.png

When dealing with multiple cameras, the following rules are followed for
each viewport:

-  If no cameras are present in the scene tree, the first one that
   enters it will become the active camera. Further cameras entering the
   scene will be ignored (unless they are set as *current*).
-  If a camera has the "*current*" property set, it will be used
   regardless of any other camera in the scene. If the property is set,
   it will become active, replacing the previous camera.
-  If an active camera leaves the scene tree, the first camera in
   tree-order will take its place.

Lights
------

There is no limitation on the number of lights nor of types of lights in
Godot. As many as desired can be added (as long as performance allows). Shadow
maps are, however, limited. The more they are used, the less the quality
overall.

It is possible to use :ref:`doc_light_baking`, to avoid using large amount of
real-time lights and improve performance.
