.. _doc_project_organization:

Структура проекта
====================

Введение
------------

Это руководство по простому рабочему процессу организации проектов.
Поскольку Godot не накладывает ограничений на структуру файлов проекта
определиться с формой организации проекта бывает не просто.
Начинать использование движка может быть небольшим вызовом. 
Поэтому мы опишем примерный путь, который можно принять или не принять.

А также упомянем о системах контроля версий.

Организация
------------

Другие игровые движки часто работают, имея базу данных активов, где вы можете
просматривать картинки, модели, звуки, и т.д.. Godot больше ориентирован на сцены
поскольку большую часть времени активы группируются внутри сцен или просто существуют
в виде файлов, но ссылаются на сцены.

Импортирование и папка игры
-----------------------

Очень часто необходимо использовать импорт активо в Godot. Поскольку
source assets для импорта также распознаются движком как resources,
это может стать проблеммой если и то и то лежит в папке проекта,
потому что во время экспорта, экспортер будет видеть и экспортирует и то и то.

Решение в том, чтобы разместить папку проекта внтри другой папки
(the actual project folder). Это позволяет разделить game
assets от source assets, и использовать систему контроля версий 
(такую как svn или git). Например:

::

    myproject/art/models/house.max
    myproject/art/models/sometexture.png
    myproject/sound/door_open.wav
    myproject/sound/door_close.wav
    myproject/translations/sheet.csv

Тогда также, сама игра, в этом случае, внутри папки game/ :

::

    myproject/game/engine.cfg
    myproject/game/scenes/house/house.scn
    myproject/game/scenes/house/sometexture.tex
    myproject/game/sound/door_open.smp
    myproject/game/sound/door_close.smp
    myproject/game/translations/sheet.en.xl
    myproject/game/translations/sheet.es.xl

Следуя этому макету, можно многое сделать:

-  Весь проект все еще находится в папке (myproject/).
-  Экспорт проекта не будет экспортировать файлы .wav и .png, которые были импортированы.
-  myproject/ можно поместить прямо в VCS (типа svn или git) под
   версионный контроль, отслеживаться будут и игровые и исходные активы.
-  Если над проектом работает команда, активы могут быть заново импортированы
    другими участниками проекта, поскольку Godot отслеживет исходные активы
   используя относительные пути.

Организация сцены
------------------

Внутри папки игры, частый вопрос iScene organizations как организовать сцены
в файловой системе. Некоторые разработчики пытаются использовать asset-type based
организацию что в конечном итоге приводит к беспорядку, 
поэтому лучше организовывать сцены по тому как игра работает а не по активам. 

Например:

Если вы организуете свой проект на основе типа активов, он будет выглядеть так::

::

    game/engine.cfg
    game/scenes/scene1.scn
    game/scenes/scene2.scn
    game/textures/texturea.png
    game/textures/another.tex
    game/sounds/sound1.smp
    game/sounds/sound2.wav
    game/music/music1.ogg

И обычно это плохая идея. Как только прокт разрастется, все станет неуправляемым.
Трудно определить что кому принадлежит.

Лучше использовать game-context based организацию,
например так:

::

    game/engine.cfg
    game/scenes/house/house.scn
    game/scenes/house/texture.tex
    game/scenes/valley/canyon.scn
    game/scenes/valley/rock.scn
    game/scenes/valley/rock.tex
    game/scenes/common/tree.scn
    game/scenes/common/tree.tex
    game/player/player.scn
    game/player/player.gd
    game/npc/theking.scn
    game/npc/theking.gd
    game/gui/main_screen/main_sceen.scn
    game/gui/options/options.scn

Эта модель позволяет проектам расти и при этом оставаться управляемыми.
Обратите внимание что все основано на частях игры которые можно назвать
или описать, как экран настроек или долина. 
Поскольку в Godot все делается в сценах, и все что может быть названо
или описано может быть сценой, этот подход удобен и прост.

Файлы кэша
-----------

Godot использует скрытый файл ".fscache" в корневой папке проекта.
On it, это кэш файлов проекта и используется чтобы быстро узнавать
о том что они изменились Отметьте **not commit this file** в git или svn, поскольку он
содержит локальную информацию и может сконфузить другой экземпляр редактора
на другом компьютере.
