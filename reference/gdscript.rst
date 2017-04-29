.. _doc_gdscript:

GDScript
========

Введение
------------

*GDScript* это высоко-уровневый, динамически типизированный язык программирования используемый для создания контента. Он использует синтаксис похожий на
`Python <https://en.wikipedia.org/wiki/Python_%28programming_language%29>`_ 
(блоки отступа и большинство ключевых слов похоже). Его цель быть оптимизированным и тесно интегрированным с Godot Engine, позволяет гибкое создание контента.

История
~~~~~~~

Изначально, Godot разрабатывался для поддержки многих языков скриптинга
(эта возможность существует до сих пор). Тем не менее, только GDScript используется сейчас.
Вот небольшая история почему.

Поначалу, движок использовал язык `Lua <http://www.lua.org>`__.
Lua быстра, но создание биндингов на ООП (через fallbacks) было сложным
и медленным и требовало огромного количества кода.После некоторых экспериментов с
`Python <http://www.python.org>`__, его тоже оказалось сложно встроить.

Последним сторонним языком для поставляемых игр был `Squirrel <http://squirrel-lang.org>`__, 
но и от него пришлось отказаться.
На данный момент, стало очевидно что кастомный скриповый язык наиболее
оптимальный для учета архитектуры Godot:

-  Godot включает скрипты в узлы. Большинство языков не учитывают такого.
-  Godot использует несколько встроенных типов данных для 2D и 3D математики. 
   Языки скриптинга не делают такого, и вставляют их неэффективно.
-  Godot использует потоки для извлечения и инициализации данных из сети
   или с диска. Интерпретатор скриптов для большинства языков не умеют это.
-  Godot имеет модель управления памятью под ресурсы, большинство скриптовых языков 
   имеют свой собственный что приводит к дублированию и ошибкам.
-  Связывание кода всегда неэффективно со многими точками ошибок,
   неожиданными багами и сложностями поддержки.

Результатом этих соображений и стал *GDScript*. Язык и интерпретатор GDScript
оказался куда меньше чем код биндинга для Lua и Squirrel, 
при эквивалентной функциональности. Со временем, наличие встроенного языка
принесло множество преимуществ.

Пример GDScript
~~~~~~~~~~~~~~~~~~~

Некоторые люди лучше понимают просто взглянув на синтаксис, 
вот простые примеры того как выглядит GDScript.

::

    # файл это class!

    # наследование

    extends BaseClass

    # переменные члены

    var a = 5 
    var s = "Hello"
    var arr = [1, 2, 3]
    var dict = {"key":"value", 2:3}

    # константы

    const answer = 42
    const thename = "Charly"

    # enums (Godot 2.2+)

    enum {UNIT_NEUTRAL, UNIT_ENEMY, UNIT_ALLY}
    enum Named {THING_1, THING_2, ANOTHER_THING = -1}

    # встроенные типы vector types

    var v2 = Vector2(1, 2)
    var v3 = Vector3(1, 2, 3)

    # функция

    func some_function(param1, param2):
        var local_var = 5

        if param1 < local_var:
            print(param1)
        elif param2 > 5:
            print(param2)
        else:
            print("fail!")

        for i in range(20):
            print(i)

        while(param2 != 0):
            param2 -= 1

        var local_var2 = param1+3
        return local_var2


    # внутренний class

    class Something:
        var a = 10

    # конструктор

    func _init():
        print("constructed!")
        var lv = Something.new()
        print(lv.a)

Если у вас есть предыдущий опыт со статически типизированными языками такими как
C, C++, или C# но вы не сталкивались с динамически-типизированными языками, прочтите 
этот туториал: :ref:`doc_gdscript_more_efficiently`.

Язык
--------

Далее, обзор-введение в GDScript. Детали, такие как 
методы доступные для массивов и других объектов, могут быть найдены
в описании их классов. 

Идентификаторы
~~~~~~~~~~~

Любая строка ограничена алфавитными символами (от ``a`` до
``z`` и ``A`` до ``Z``), цифрами (от ``0`` до ``9``) и ``_`` квалифицируется
как идентификатор. Кроме того, идентификатор не может начинаться с цифры.
Идентификаторы регистро-зависимы (``foo`` отличается от ``FOO``).

Ключевые слова
~~~~~~~~

Вот список ключевых слов поддерживаемых языком. Поскольку ключевые слова
являются зарезервированными словами (tokens), они не могут использоваться
как идентификаторы.

+------------+---------------------------------------------------------------------------------------------------------------+
|  Ключевое слово   | Описание                                                                                                   |
+============+===============================================================================================================+
| if         | см `if/else/elif`_.                                                                                          |
+------------+---------------------------------------------------------------------------------------------------------------+
| elif       | см `if/else/elif`_.                                                                                          |
+------------+---------------------------------------------------------------------------------------------------------------+
| else       | см `if/else/elif`_.                                                                                          |
+------------+---------------------------------------------------------------------------------------------------------------+
| for        | см for_.                                                                                                     |
+------------+---------------------------------------------------------------------------------------------------------------+
| do         | Reserved for future implementation of do...while loops.                                                       |
+------------+---------------------------------------------------------------------------------------------------------------+
| while      | см while_.                                                                                                   |
+------------+---------------------------------------------------------------------------------------------------------------+
| match      | см match_.                                                                                                   |
+------------+---------------------------------------------------------------------------------------------------------------+
| switch     | зарезервировано на будущее.                                                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| case       | зарезервировано на будущее.                                                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| break      | сейчас существует реализация для циклов ``for`` или ``while``.                                                 |
+------------+---------------------------------------------------------------------------------------------------------------+
| continue   | немедленный переход к следующей итерации ``for`` или ``while`` цикла.                                     |
+------------+---------------------------------------------------------------------------------------------------------------+
| pass       | Used where a statement is required syntactically but execution of code is undesired, e.g. in empty functions. |
+------------+---------------------------------------------------------------------------------------------------------------+
| return     | возврат значений из функции.                                                                              |
+------------+---------------------------------------------------------------------------------------------------------------+
| class      | объявляет class.                                                                                              |
+------------+---------------------------------------------------------------------------------------------------------------+
| extends    | говорит что class расширяет текуший class.Также проверяет расширяет ли переменная это данный класс.     |
+------------+---------------------------------------------------------------------------------------------------------------+
| tool       | выполняет script в редакторе.                                                                            |
+------------+---------------------------------------------------------------------------------------------------------------+
| signal     | объявляет signal.                                                                                             |
+------------+---------------------------------------------------------------------------------------------------------------+
| func       | Defines a function.                                                                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| static     | объявляет статическую функцию. Static member variables are not allowed.                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| const      | объявляет константу.                                                                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| enum       | объявляет enum. (Godot 2.2+)                                                                                 |
+------------+---------------------------------------------------------------------------------------------------------------+
| var        | объявляет переменную.                                                                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| onready    | Инициализирует переменную когда Node к которому прикреплен скрипт присоединяется и его потомки становятся частью дерева сцены scene tree.   |
+------------+---------------------------------------------------------------------------------------------------------------+
| export     | Сохраняет переменную вместе с ресурсом, к которому она присоединена, и делает её видимой и изменяемой в редакторе.  |
+------------+---------------------------------------------------------------------------------------------------------------+
| setget     | задает сеттер и геттер функции для переменной.                                                           |
+------------+---------------------------------------------------------------------------------------------------------------+
| breakpoint | помощник в редакторе для точек останова отладчика.                                                                       |
+------------+---------------------------------------------------------------------------------------------------------------+

Операторы
~~~~~~~~~

Далее - список поддерживаемых операторов и их приоритет
(TODO, change означает что это было сделано для отражения операторов Python)

+---------------------------------------------------------------+-----------------------------------------+
| **Operator**                                                  | **Description**                         |
+---------------------------------------------------------------+-----------------------------------------+
| ``x[index]``                                                  | Subscription, Высший приоритет          |
+---------------------------------------------------------------+-----------------------------------------+
| ``x.attribute``                                               | Attribute Reference                     |
+---------------------------------------------------------------+-----------------------------------------+
| ``extends``                                                   | Instance Type Checker                   |
+---------------------------------------------------------------+-----------------------------------------+
| ``~``                                                         | Bitwise NOT                             |
+---------------------------------------------------------------+-----------------------------------------+
| ``-x``                                                        | Negative                                |
+---------------------------------------------------------------+-----------------------------------------+
| ``*`` ``/`` ``%``                                             | Умножение / деление / Remainder         |
+---------------------------------------------------------------+-----------------------------------------+
| ``+`` ``-``                                                   | сложение / вычитание                    |
+---------------------------------------------------------------+-----------------------------------------+
| ``<<`` ``>>``                                                 | Битовый сдвиг                            |
+---------------------------------------------------------------+-----------------------------------------+
| ``&``                                                         | побитовое AND                             |
+---------------------------------------------------------------+-----------------------------------------+
| ``^``                                                         | побитовое XOR                             |
+---------------------------------------------------------------+-----------------------------------------+
| ``|``                                                         | побитовое OR                            |
+---------------------------------------------------------------+-----------------------------------------+
| ``<`` ``>`` ``==`` ``!=`` ``>=`` ``<=``                       | сравнение                               |
+---------------------------------------------------------------+-----------------------------------------+
| ``in``                                                        | Content Test                            |
+---------------------------------------------------------------+-----------------------------------------+
| ``!`` ``not``                                                 | Boolean NOT                             |
+---------------------------------------------------------------+-----------------------------------------+
| ``and`` ``&&``                                                | Boolean AND                             |
+---------------------------------------------------------------+-----------------------------------------+
| ``or`` ``||``                                                 | Boolean OR                              |
+---------------------------------------------------------------+-----------------------------------------+
| ``if x else``                                                 | Ternary if/else (Godot 2.2+)            |
+---------------------------------------------------------------+-----------------------------------------+
| ``=`` ``+=`` ``-=`` ``*=`` ``/=`` ``%=`` ``&=`` ``|=``        | присваивание, низший приоритет          |
+---------------------------------------------------------------+-----------------------------------------+

Литералы
~~~~~~~~

+--------------------------+--------------------------------+
| **Literal**              | **Type**                       |
+--------------------------+--------------------------------+
| ``45``                   | десятичное целое               |
+--------------------------+--------------------------------+
| ``0x8F51``               | шестнадцатеричное (hex) целое  |
+--------------------------+--------------------------------+
| ``3.14``, ``58.1e-10``   | с плавающей точкой дробное (real)   |
+--------------------------+--------------------------------+
| ``"Hello"``, ``"Hi"``    | Строковое                      |
+--------------------------+--------------------------------+
| ``"""Hello, Dude"""``    | Многострочное строковое        |
+--------------------------+--------------------------------+
| ``@"Node/Label"``        | NodePath или StringName        |
+--------------------------+--------------------------------+

Комментарии
~~~~~~~~

Все начинающееся с ``#`` до конца строки игнорируется и является
комментарием.

::

    # Это комментарий

..  Раскомментируй меня если/когда https://github.com/godotengine/godot/issues/1320 будет пофиксен
    
    Многострочные комментарии можно делать с помощью """ (тройных кавычек) в
    начале и в конце блока текста.
    
    ::
    
        """ Все в этих строках
        является
        комментарием """

Встроенные типы
--------------

Базовые встроенные типы
~~~~~~~~~~~~~~~~~~~~

Переменной в GDScript может быть присвоено значение нескольких типов.

null
^^^^

``null`` пустой тип данных который не содержит никакой информации
и не может быть назначено никакой другое значение. 

bool
^^^^

тип Boolean может быть только ``true`` или ``false``.

int
^^^

целочисленный тип данных может содержать целые числа, (положительные и отрицательные).

float
^^^^^

Используется для хранения чисел с плавающей точкой (real numbers).

:ref:`String <class_String>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

набор символов в формате `Unicode  <https://en.wikipedia.org/wiki/Unicode>`_. Строки могут содержать
`стандартные C escape sequences <https://en.wikipedia.org/wiki/Escape_sequences_in_C>`_.
GDScript поддерживает :ref:`format strings aka printf functionality
<doc_gdscript_printf>`.

встроенный типы Vector
~~~~~~~~~~~~~~~~~~~~~

:ref:`Vector2 <class_Vector2>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2D вектор содержит поля  ``x`` и ``y`` . Альтернативно к полям можно
обратиться как ``width`` и ``height`` для читабельности. А также можно
обратиться как к массиву.

:ref:`Rect2 <class_Rect2>`
^^^^^^^^^^^^^^^^^^^^^^^^^^

2D прямоугольник имеет два векторных поля : ``pos`` и ``size``.
Альтернативно содержит поле ``end`` с ``pos+size``.

:ref:`Vector3 <class_Vector3>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

3D вектор содержит поля ``x``, ``y`` и ``z`` . Также доступны как массив.

:ref:`Matrix32 <class_Matrix32>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

3x2 матрица используемая для 2D трансформаций.

:ref:`Plane <class_Plane>`
^^^^^^^^^^^^^^^^^^^^^^^^^^

3D плоскость в нормализованной форме содержит векторное поле ``normal`` 
и скалярное значение дистанции ``d``.

:ref:`Quat <class_Quat>`
^^^^^^^^^^^^^^^^^^^^^^^^

Quaternion это тип данных для представления 3D вращения. Полезно
для интерполяции вращений.

:ref:`AABB <class_AABB>`
^^^^^^^^^^^^^^^^^^^^^^^^

Axis Aligned bounding box - габаритный бокс выровненный по осям
(или 3D box) содержит 2 векторных поля: ``pos``
и ``size``. Альтернативно содержит поле ``end`` и
``pos+size``. Как алиас типа, ``Rect3`` can be used
interchangeably.

:ref:`Matrix3 <class_Matrix3>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

3x3 матрица для 3D вращения и масштабирования. Содержит 3 векторных поля
(``x``, ``y`` и ``z``) доступных также как массив 3D
векторов.

:ref:`Transform <class_Transform>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

3D Transform содержит поле Matrix3 ``basis`` и поле Vector3 
``origin``.

Встроенные типы в Engine
~~~~~~~~~~~~~~~~~~~~~

:ref:`Color <class_Color>`
^^^^^^^^^^^^^^^^^^^^^^^^^^

тип данных Color содержит поля ``r``, ``g``, ``b``, и ``a`` . Доступные также как ``h``, ``s``, и ``v`` для hue/saturation/value.

:ref:`Image <class_Image>`
^^^^^^^^^^^^^^^^^^^^^^^^^^

Содержит кастомный формат 2D изображения и позволяет прямой доступ
к пикселям.

:ref:`NodePath <class_NodePath>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Скомпилированный путь к узлу, используется в основном в системе сцен. 
Может быть легко присвоен строке или из строки (String).

:ref:`RID <class_RID>`
^^^^^^^^^^^^^^^^^^^^^^

Resource ID (RID). Servers use generic RIDs to reference opaque data.

:ref:`Object <class_Object>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Базовый класс для всего что не является встроенным типом.

:ref:`InputEvent <class_InputEvent>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

События от устройств ввода содержат в очень компактной форме в виде объектов
InputEvent.В связи с тем что они могут быть приняты в больших количествах от
кадра к кадру они оптимизированы в собственном типе данных.

Встроенные типы контейнеров
~~~~~~~~~~~~~~~~~~~~~~~~

:ref:`Array <class_Array>`
^^^^^^^^^^^^^^^^^^^^^^^^^^

Общая последовательность объектов произвольных типов, включая другие массивы или словари (смотрите ниже). 
Массив может изменять размер динамически. Индексация элементов массива начинается с индекса ``0``.
Начиная с Godot 2.1, индексация может быть отрицательной как в Python, для отсчета с конца.

::

    var arr=[]
    arr=[1, 2, 3]
    var b = arr[1]            # это 2
    var c = arr[arr.size()-1] # это 3
    var d = arr[-1]           # как и предыдущая строка но короче
    arr[0] = "Hi!"            # заменяет значение 1 на "Hi"
    arr.append(4)             # теперь массив ["Hi", 2, 3, 4]

GDScript массивы расположены в памяти линейно для скорости. Очень большие массивы
(больше 10 тыс. элементов) могут приводить к фрагментации памяти.
Для этого сущестуют спец. типы массивов. 
Которые принимают только значения одного типа. Они позволяют избежать фрагментации
памяти и занимают меньше памяти но они атомарны и имеют тенденцию медленнее стартовать
чем стандартные массивы. Они рекомендуются только для очень больших массивов данных: 

- :ref:`ByteArray <class_ByteArray>`: Массив байтов (целые от 0 до 255).
- :ref:`IntArray <class_IntArray>`: Массив целых.
- :ref:`FloatArray <class_FloatArray>`: Массив дробных с плав. точкой.
- :ref:`StringArray <class_StringArray>`: Массив строк.
- :ref:`Vector2Array <class_Vector2Array>`: Массив объектов :ref:`Vector2 <class_Vector2>` .
- :ref:`Vector3Array <class_Vector3Array>`: Массив объектов :ref:`Vector3 <class_Vector3>` .
- :ref:`ColorArray <class_ColorArray>`: Массив объектов :ref:`Color <class_Color>` .

:ref:`Dictionary <class_Dictionary>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ассоциативный контейнер который содержит значения ссылающиеся по уникальным ключам.

::

    var d={4:5, "a key":"a value", 28:[1,2,3]}
    d["Hi!"] = 0
    var d = {
        22         : "Value",
        "somekey"  : 2,
        "otherkey" : [2,3,4],
        "morekey"  : "Hello"
    }

Lua-стиль табличного синтаксиса также поддерживается. Lua-использует ``=`` вместо ``:`` 
и не использует кавычки для пометки строковых ключей (немного сокращая писанину). 
Заметьте однако что как любой GDScript идентификатор, ключи записанные в такой форме
не могут начинаться с цифры

::

    var d = {
        test22 = "Value", 
        somekey = 2,
        otherkey = [2,3,4],
        morekey = "Hello"
    }

Добавить ключ к существующему словарю можно обращаясь к нему как к существующему
ключу и присваивая ему значение::

    var d = {} # создает пустой словарь
    d.Waiting = 14 # добавляет строку "Waiting" как ключ и назначает ему значение 14 
    d[4] = "hello" # добавляет целое `4` как ключ и назначает ему строку "hello" как значение
    d["Godot"] = 3.01 # добавляет строку "Godot" как ключ и назначает значение 3.01 

Data
----

Переменные
~~~~~~~~~

Переменные могут существовать как члены класса или локально в функциях. Они создаются
ключевым словом ``var`` и, опционально, могут получать значения при объявлении.

::

    var a  # по-умолчанию тип данных - null  
    var b = 5
    var c = 3.8
    var d = b + c  # переменные всегда инициализируются по-порядку

Константы
~~~~~~~~~

Constants are similar to variables, but must be constants or constant
expressions должны инициализироваться при объявлении. 

::

    const a = 5
    const b = Vector2(20, 20)
    const c = 10 + 20 # константное выражение
    const d = Vector2(20, 30).x  # constant expression: 20
    const e = [1, 2, 3, 4][0]  # constant expression: 1
    const f = sin(20)  # sin() может использоваться в константных выражениях
    const g = x + 20  # invalid; это НЕ константное выражение!
    
Enums
^^^^^

*Note, только в Godot 2.2 и выше.*

Enums основан на константах, и очень полезны, если вы хотите назначить
последовательные целые числа некоторой константе.

Если вы передаете имя в enum, оно также поместит все значения в
постоянный словарь этого имени.

::

    enum {TILE_BRICK, TILE_FLOOR, TILE_SPIKE, TILE_TELEPORT}
    # То же что и:
    const TILE_BRICK = 0
    const TILE_FLOOR = 1
    const TILE_SPIKE = 2
    const TILE_TELEPORT = 3

    enum State {STATE_IDLE, STATE_JUMP = 5, STATE_SHOOT}
    # То же что и:
    const STATE_IDLE = 0
    const STATE_JUMP = 5
    const STATE_SHOOT = 6
    const State = {STATE_IDLE = 0, STATE_JUMP = 5, STATE_SHOOT = 6}


Функции
~~~~~~~~~

Функции всегда принадлежат классам `class <Classes_>`_. 
Приоритет области действия для поиска переменной: 
локальная → член класса → глобальная. Переменная ``self`` всегда доступна
и представляется как опция для доступа к членам класса, но не всегда требуется
(and should *not* be sent as the function's first argument, unlike Python).

::

    func myfunction(a, b):
        print(a)
        print(b)
        return a + b  # return опционален; без него вернется null 

Функция может ``return`` в любом местеt. Дефолтное возвращаемое значение ``null``.

Referencing Functions
^^^^^^^^^^^^^^^^^^^^^

Для вызова функций в *base class* (i.e. one ``extend``-ed в вашем текущем классе),
предваряйте ``.`` перед именем функции:

::

    .basefunc(args)

В отличие от Python, функции *НЕ* объекты первого класса в GDScript.
Это значит что они не могут быть сохранены в переменных variables, 
переданы как аргумент другим функциям или озвращены из других функций.
Это все из соображений производительности.

Для обращения к функции в рантайме, (e.g. to store it in a variable, or
pass it to another function as an argument) one нужно использовать ``call`` или
``funcref`` хелперы::
   
    # Вызов функции по-имени в один шаг
    mynode.call("myfunction", args)  

    # Сохранение обращения к функции 
    var myfunc = funcref(mynode, "myfunction")
    
    # Вызов сохраненного обращения к функции 
    myfunc.call_func(args)


Помните что дефолтные функции такие как  ``_init``, и большинство
нотификаций таких как``_enter_tree``, ``_exit_tree``, ``_process``,
``_fixed_process``, etc. вызываются во всех базовых классах автоматически.
Так что вызывать их явно нужно только если вы их overloading каким-то образом. 


Статические функции
^^^^^^^^^^^^^^^^

Функция может быть объявлена статичной. Статическая функция не имеет доступа к членам класса
или ``self``. Это полезно в основном для создания библиотек хелперов функций:

::

    static func sum2(a, b):
        return a + b


Операторы управления потоком выполнения
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Операторы стандартны и могут быть присваиваться, вызовами функции, структурами
управления потоком, и т.п.. ``;`` как разделитель операторов полностью опционален.

if/else/elif
^^^^^^^^^^^^

Простые условия создаются с использованием синтаксиса ``if``/``else``/``elif``.
Скобки вокруг условий допускаются, но не обязательны. Учитывая характер отступов 
на основе табуляции, ``elif`` можно использовать вместо
``else``/``if`` для поддержания уровня вложенности.

::

    if [expression]:
        statement(s)
    elif [expression]:
        statement(s)
    else:
        statement(s)

Короткие операторы могут быть написаны в той же строке, что и условие::

    if (1 + 1 == 2): return 2 + 2
    else:
        var x = 3 + 3
        return x

Иногда вам может понадобиться назначить другое начальное значение, 
основанное на логическом выражении. В таком случае удобно тернарное-if выражение
(Godot 2.2+)::

    var x = [true-value] if [expression] else [false-value]
    y += 3 if y < 10 else -1

while
^^^^^

Простые циклы создают с помощью синтаксиса ``while``. Циклы можно прерывать
оператором ``break`` и продолжать оператором ``continue``:

::

    while [выражение]:
        оператор(ы)

for
^^^

Для итераций в диапазоне, таком как массив или таблица, используют цикл *for*.
При обходе массива, текущий элемент массива сохраняется в переменной цикла.
При итерации по словарю, в переменной цикла сохраняется *index* .

::

    for x in [5, 7, 11]:
        statement  # цикл повторится три раза со значениями x как 5, затем 7 и наконец 11

    var dict = {"a":0, "b":1, "c":2}
    for i in dict:
        print(dict[i])  # цикл предоставляет ключи в произвольном порядке; может быть напечатано 0, 1, 2, или 2, 0, 1, и т.п...

    for i in range(3):
        statement  # похоже на [0, 1, 2] но не выделяет массив

    for i in range(1,3):
        statement  # похоже на [1, 2] но не выделяет массив

    for i in range(2,8,2):
        statement  # похоже на [2, 4, 6] но не выделяет массив

    for c in "Hello":
        print(c)   # итерация по всем символам в String, печатает каждый символ на новой строке
match
^^^^^

Оператор ``match`` используют для ветвления потока выполнения программы.
Эквивалентно оператору ``switch`` из большинства других языков но обладает некоторым доп. функционалом.

Базовый синтаксис:
::
    
    match [expression]:
        [pattern](s): [block]
        [pattern](s): [block]
        [pattern](s): [block]


**Crash-course для тех кто знаком с оператором switch**:

1) replace ``switch`` with ``match``
2) remove ``case``
3) remove any ``break``'s. If you don't want to ``break`` by default you can use ``continue`` for a fallthrough.
4) change ``default`` to a single underscore.


**Control flow**:

The patterns are matched from top to bottom.
If a pattern matches, the corresponding block will be executed. After that, the execution continues below the ``match`` statement.
If you want to have a fallthrough you can use ``continue`` to stop execution in the current block and check the ones below it.




There are 6 pattern types:

- constant pattern
    constant primitives, like numbers and strings ::
    
        match x:
            1:      print("We are number one!")
            2:      print("Two are better than one!")
            "test": print("Oh snap! It's a string!")


- variable pattern
    matches the contents of a variable/enum ::
    
        match typeof(x):
            TYPE_FLOAT:  print("float")
            TYPE_STRING: print("text")
            TYPE_ARRAY:  print("array")


- wildcard pattern
    This pattern matches everything. It's written as a single underscore.
    
    It can be used as the equivalent of the ``default`` in a ``switch`` statement in other languages. ::
    
        match x:
            1: print("it's one!")
            2: print("it's one times two!")
            _: print("it's not 1 or 2. I don't care tbh.")


- binding pattern
    A binding pattern introduces a new variable. Like the wildcard pattern, it matches everything - and also gives that value a name.
    It's especially useful in array and dictionary patterns. ::
        
        match x:
            1:           print("it's one!")
            2:           print("it's one times two!")
            var new_var: print("it's not 1 or 2, it's ", new_var)


- array pattern
    matches an array. Every single element of the array pattern is a pattern itself so you can nest them.
    
    The length of the array is tested first, it has to be the same size as the pattern, otherwise the pattern don't match.

    **Open-ended array**: An array can be bigger than the pattern by making the last subpattern ``..``
    
    Every subpattern has to be comma seperated. ::
    
        match x:
            []:
                print("empty array")
            [1, 3, "test", null]:
                print("very specific array")
            [var start, _, "test"]:
                print("first element is ", start, ", and the last is \"test\"")
            [42, ..]:
                print("open ended array")
    
- dictionary pattern
    Works in the same was as the array pattern. Every key has to be a constant pattern.

    The size of the dictionary is tested first, it has to be the same size as the pattern, otherwise the pattern don't match.

    **Open-ended dictionary**: A dictionary can be bigger than the pattern by making the last subpattern ``..``

    Every subpattern has to be comma seperated.

    If you don't specify a value, then only the existance of the key is checked.

    A value pattern is seperated from the key pattern with a ``:`` ::

        match x:
            {}:
                print("empty dict")
            {"name": "dennis"}:
                print("the name is dennis")
            {"name": "dennis", "age": var age}:
                print("dennis is ", age, " years old.")
            {"name", "age"}:
                print("has a name and an age, but it's not dennis :(")
            {"key": "godotisawesome", ..}:
                print("I only checked for one entry and ignored the rest")

Multipatterns:
    You can also specify multiple patterns seperated by a comma. These patterns aren't allowed to have any bindings in them. ::

        match x:
            1, 2, 3:
                print("it's 1 - 3")
            "sword", "splashpotion", "fist":
                print("yep, you've taken damage")



Классы
~~~~~~~

По умолчанию, тело файла скрипта это безымянный класс и извне к нему можно
обратиться только как к ресурсу или файлу. 
Синтаксис Class очень компактный и может содержать только переменные - члены
или функции. 
Статические функции допустимы, но не статические члены (это в духе потоко-безопасности,
скрипты могут инициализироваться в отдельных потоках без ведома пользователя).
Тким же образом, переменные члены (включая массивы и словари) инициализируются 
каждый раз при создании экземпляра.

Ниже примеры файла класса. 

::

    # сохранен как файл с именем myclass.gd

    var a = 5

    func print_value_of_a():
        print(a)

наследование
^^^^^^^^^^^

Класс (сохраненный как файлe) может наследовать от 

- Глобального класса
- Другого файла-класса 
- Внутреннего класса внутри другого файла-класса. 

Множественного наследования нет. 

Наследование использует ключевое слово ``extends`` :

::

    # Наследование/расширение глобально доступного класса
    extends SomeClass 
    
    # Наследование/расширение именованного класса-файла
    extends "somefile.gd" 
    
    # Наследование/расширение внутреннего класса из другого класса
    extends "somefile.gd".SomeInnerClass


To check if a given instance inherits from a given class 
the ``extends`` keyword can be used as an operator instead:

::

    # Cache the enemy class
    const enemy_class = preload("enemy.gd")

    # [...]

    # use 'extends' to check inheritance
    if (entity extends enemy_class):
        entity.apply_damage()

Конструктор класса
^^^^^^^^^^^^^^^^^

Конструктор класса, вызывающий инстанцирование класса, называется ``_init``. 
Как упоминалось ранее конструкторы родительского класса вызываются автоматически
при наследовании класса. Поэтому не нужно вызывать ``._init()`` явно.

Если родительский конструктор принимает аргументы, они передаются так:

::

    func _init(args).(parent_args):
       pass

Внутренние классы
^^^^^^^^^^^^^

Файл-класс может содержать внутренние классы. Внутренние классы объявляют
ключевым словом ``class`` . Они инстанцируются используя функцию ``ClassName.new()`` .

::

    # внутри файла класса

    # An inner class in this class file
    class SomeInnerClass:
        var a = 5
        func print_value_of_a():
            print(a)

    # This is the constructor of the class file's main class
    func _init():
        var c = SomeInnerClass.new() 
        c.print_value_of_a()

Классы как ресурсы
^^^^^^^^^^^^^^^^^^^^

Classes храняться как файлы доступные как :ref:`resources <class_GDScript>`. 
Они должны быть загружены с диска чтобы быть доступными для других классов.
Это достигается с помощью функций ``load`` или ``preload`` (см. ниже). 
Инстанцирование загруженного класса осуществляется функцией ``new`` on on the class object::

    # Load the class resource when calling load()
    var MyClass = load("myclass.gd")

    # Preload the class only once at compile time
    var MyClass2 = preload("myclass.gd")

    func _init():
        var a = MyClass.new()
        a.somefunction()

Exports
~~~~~~~

Class members can be exported. This means their value gets saved along
with the resource (e.g. the :ref:`scene <class_PackedScene>`) they're attached
to. They will also be available for editing in the property editor. Exporting
is done by using the ``export`` keyword::

    extends Button

    export var number = 5  # значение будет сохранено и видимо в редакторе свойств

An exported variable must be initialized to a constant expression or have an
export hint in the form of an argument to the export keyword (see below).

One of the fundamental benefits of exporting member variables is to have
them visible and editable in the editor. This way artists and game designers
can modify values that later influence how the program runs. For this, a
special export syntax is provided.

::

    # If the exported value assigns a constant or constant expression, 
    # the type will be inferred and used in the editor

    export var number = 5

    # Export can take a basic data type as an argument which will be 
    # used in the editor

    export(int) var number

    # Export can also take a resource type to use as a hint

    export(Texture) var character_face

    # Integers and strings hint enumerated values

    # Editor will enumerate as 0, 1 and 2
    export(int, "Warrior", "Magician", "Thief") var character_class   
    # Editor will enumerate with string names 
    export(String, "Rebecca", "Mary", "Leah") var character_name 

    # Strings as paths

    # String is a path to a file
    export(String, FILE) var f  
    # String is a path to a directory
    export(String, DIR) var f  
    # String is a path to a file, custom filter provided as hint
    export(String, FILE, "*.txt") var f  

    # Using paths in the global filesystem is also possible, 
    # but only in tool scripts (see further below)

    # String is a path to a PNG file in the global filesystem
    export(String, FILE, GLOBAL, "*.png") var tool_image 
    # String is a path to a directory in the global filesystem
    export(String, DIR, GLOBAL) var tool_dir

    # The MULTILINE setting tells the editor to show a large input 
    # field for editing over multiple lines
    export(String, MULTILINE) var text

    # Limiting editor input ranges

    # Allow integer values from 0 to 20
    export(int, 20) var i  
    # Allow integer values from -10 to 20 
    export(int, -10, 20) var j 
    # Allow floats from -10 to 20, with a step of 0.2
    export(float, -10, 20, 0.2) var k 
    # Allow values y = exp(x) where y varies betwee 100 and 1000 
    # while snapping to steps of 20. The editor will present a 
    # slider for easily editing the value. 
    export(float, EXP, 100, 1000, 20) var l 

    # Floats with easing hint

    # Display a visual representation of the ease() function 
    # when editing
    export(float, EASE) var transition_speed 

    # Colors

    # Color given as Red-Green-Blue value
    export(Color, RGB) var col  # Color is RGB
    # Color given as Red-Green-Blue-Alpha value
    export(Color, RGBA) var col  # Color is RGBA
   
    # another node in the scene can be exported too
    
    export(NodePath) var node

It must be noted that even if the script is not being run while at the
editor, the exported properties are still editable (see below for
"tool").

Экспорт битовых флагов
^^^^^^^^^^^^^^^^^^^

Integers used as bit flags can store multiple ``true``/``false`` (boolean)
values in one property. By using the export hint ``int, FLAGS``, they
can be set from the editor:

::

    # Individually edit the bits of an integer
    export(int, FLAGS) var spell_elements = ELEMENT_WIND | ELEMENT_WATER 

Restricting the flags to a certain number of named flags is also
possible. The syntax is very similar to the enumeration syntax:

::

    # Set any of the given flags from the editor
    export(int, FLAGS, "Fire", "Water", "Earth", "Wind") var spell_elements = 0 

In this example, ``Fire`` has value 1, ``Water`` has value 2, ``Earth``
has value 4 and ``Wind`` corresponds to value 8. Usually, constants
should be defined accordingly (e.g. ``const ELEMENT_WIND = 8`` and so
on).

Using bit flags requires some understanding of bitwise operations. If in
doubt, boolean variables should be exported instead.

Экспорт массивов
^^^^^^^^^^^^^^^^

Exporting arrays works but with an important caveat: While regular
arrays are created local to every class instance, exported arrays are *shared*
between all instances. This means that editing them in one instance will
cause them to change in all other instances. Exported arrays can have
initializers, but they must be constant expressions.

::

    # Exported array, shared between all instances.
    # Default value must be a constant expression.

    export var a=[1,2,3]

    # Typed arrays also work, only initialized empty:

    export var vector3s = Vector3Array()
    export var strings = StringArray()

    # Regular array, created local for every instance.
    # Default value can include run-time values, but can't
    # be exported.

    var b = [a,2,3]


Сеттеры/геттеры
~~~~~~~~~~~~~~~

It is often useful to know when a class' member variable changes for 
whatever reason. It may also be desired to encapsulate its access in some way. 

For this, GDScript provides a *setter/getter* syntax using the ``setget`` keyword. 
It is used directly after a variable definition:

::

    var variable = value setget setterfunc, getterfunc

Whenever the value of ``variable`` is modified by an *external* source 
(i.e. not from local usage in the class), the *setter* function (``setterfunc`` above)
will be called. This happens *before* the value is changed. The *setter* must decide what to do 
with the new value. Vice-versa, when ``variable`` is accessed, the *getter* function 
(``getterfunc`` above) must ``return`` the desired value. Below is an example: 


::

    var myvar setget myvar_set,myvar_get

    func myvar_set(newvalue):
        myvar=newvalue

    func myvar_get():
        return myvar # getter must return a value

Either of the *setter* or *getter* functions can be omitted:

::

    # Only a setter
    var myvar = 5 setget myvar_set
    # Only a getter (note the comma)
    var myvar = 5 setget ,myvar_get

Get/Setters are especially useful when exporting variables to editor in tool
scripts or plugins, for validating input.

As said *local* access will *not* trigger the setter and getter. Here is an 
illustration of this: 

::

    func _init():
        # Does not trigger setter/getter
        myinteger=5
        print(myinteger)
        
        # Does trigger setter/getter
        self.myinteger=5
        print(self.myinteger)

режим Tool 
~~~~~~~~~

Scripts, by default, don't run inside the editor and only the exported
properties can be changed. In some cases it is desired that they do run
inside the editor (as long as they don't execute game code or manually
avoid doing so). For this, the ``tool`` keyword exists and must be
placed at the top of the file:

::

    tool
    extends Button

    func _ready():
        print("Hello")

Управление памятью
~~~~~~~~~~~~~~~~~

If a class inherits from :ref:`class_Reference`, then instances will be
freed when no longer in use. No garbage collector exists, just simple
reference counting. By default, all classes that don't define
inheritance extend **Reference**. If this is not desired, then a class
must inherit :ref:`class_Object` manually and must call instance.free(). To
avoid reference cycles that can't be freed, a ``weakref`` function is
provided for creating weak references.


Сигналы
~~~~~~~

It is often desired to send a notification that something happened in an
instance. GDScript supports creation of built-in Godot signals.
Declaring a signal in GDScript is easy using the `signal` keyword. 

::

    # No arguments
    signal your_signal_name
    # With arguments
    signal your_signal_name_with_args(a,b)

These signals, just like regular signals, can be connected in the editor
or from code. Just take the instance of a class where the signal was
declared and connect it to the method of another instance:

::

    func _callback_no_args():
        print("Got callback!")

    func _callback_args(a,b):
        print("Got callback with args! a: ",a," and b: ",b)

    func _at_some_func():
        instance.connect("your_signal_name",self,"_callback_no_args")
        instance.connect("your_signal_name_with_args",self,"_callback_args")

It is also possible to bind arguments to a signal that lacks them with
your custom values:

::

    func _at_some_func():
        instance.connect("your_signal_name",self,"_callback_args",[22,"hello"])

This is very useful when a signal from many objects is connected to a
single callback and the sender must be identified:

::

    func _button_pressed(which):
        print("Button was pressed: ",which.get_name())

    func _ready():
        for b in get_node("buttons").get_children():
            b.connect("pressed",self,"_button_pressed",[b])

Finally, emitting a custom signal is done by using the
Object.emit_signal method:

::

    func _at_some_func():
        emit_signal("your_signal_name")
        emit_signal("your_signal_name_with_args",55,128)
        someinstance.emit_signal("somesignal")

Coroutines
~~~~~~~~~~

GDScript offers support for `coroutines <https://en.wikipedia.org/wiki/Coroutine>`_ 
via the ``yield`` built-in function. Calling ``yield()`` will
immediately return from the current function, with the current frozen
state of the same function as the return value. Calling ``resume`` on
this resulting object will continue execution and return whatever the
function returns. Once resumed the state object becomes invalid. Here is
an example:

::

    func myfunc():

       print("hello")
       yield()
       print("world")

    func _ready():

        var y = myfunc()
        # Function state saved in 'y'
        print("my dear")
        y.resume()
        # 'y' resumed and is now an invalid state

Will print:

::

    hello
    my dear
    world

It is also possible to pass values between yield() and resume(), for
example:

::

    func myfunc():

       print("hello")
       print( yield() )
       return "cheers!"

    func _ready():

        var y = myfunc()
        # Function state saved in 'y'
        print( y.resume("world") )
        # 'y' resumed and is now an invalid state

Will print:

::

    hello
    world
    cheers!

Coroutines & signals
^^^^^^^^^^^^^^^^^^^^

The real strength of using ``yield`` is when combined with signals.
``yield`` can accept two parameters, an object and a signal. When the
signal is received, execution will recommence. Here are some examples:

::

    # Resume execution the next frame
    yield( get_tree(), "idle_frame" )

    # Resume execution when animation is done playing:
    yield( get_node("AnimationPlayer"), "finished" )

    # Wait 5 seconds, then resume execution (Godot 2.2+)
    yield( get_tree().create_timer(5.0), "timeout" )

ключевое слово Onready 
~~~~~~~~~~~~~~~

When using nodes, it's very common to desire to keep references to parts
of the scene in a variable. As scenes are only warranted to be
configured when entering the active scene tree, the sub-nodes can only
be obtained when a call to Node._ready() is made.

::

    var mylabel

    func _ready():
        mylabel = get_node("MyLabel")

This can get a little cumbersome, specially when nodes and external
references pile up. For this, GDScript has the ``onready`` keyword, that
defers initialization of a member variable until _ready is called. It
can replace the above code with a single line:

::

    onready var mylabel = get_node("MyLabel")
