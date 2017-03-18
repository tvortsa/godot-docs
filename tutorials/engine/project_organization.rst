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

Then also, the game itself is, in this case, inside a game/ folder:

::

    myproject/game/engine.cfg
    myproject/game/scenes/house/house.scn
    myproject/game/scenes/house/sometexture.tex
    myproject/game/sound/door_open.smp
    myproject/game/sound/door_close.smp
    myproject/game/translations/sheet.en.xl
    myproject/game/translations/sheet.es.xl

Following this layout, many things can be done:

-  The whole project is still inside a folder (myproject/).
-  Exporting the project will not export the .wav and .png files which
   were imported.
-  myproject/ can be put directly inside a VCS (like svn or git) for
   version control, both game and source assets are kept track of.
-  If a team is working on the project, assets can be re-imported by
   other project members, because Godot keeps track of source assets
   using relative paths.

Scene organization
------------------

Inside the game folder, a question that often arises is how to organize
the scenes in the filesystem. Many developers try asset-type based
organization and end up having a mess after a while, so the best answer
is probably to organize them based on how the game works and not based
on asset type. Here are some examples.

If you were organizing your project based on asset type, it would look
like this:

::

    game/engine.cfg
    game/scenes/scene1.scn
    game/scenes/scene2.scn
    game/textures/texturea.png
    game/textures/another.tex
    game/sounds/sound1.smp
    game/sounds/sound2.wav
    game/music/music1.ogg

Which is generally a bad idea. When a project starts growing beyond a
certain point, this becomes unmanageable. It's really difficult to tell
what belongs to what.

It's generally a better idea to use game-context based organization,
something like this:

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

This model or similar models allows projects to grow to really large
sizes and still be completely manageable. Notice that everything is
based on parts of the game that can be named or described, like the
settings screen or the valley. Since everything in Godot is done with
scenes, and everything that can be named or described can be a scene,
this workflow is very smooth and easygoing.

Cache files
-----------

Godot uses a hidden file called ".fscache" at the root of the project.
On it, it caches project files and is used to quickly know when one is
modified. Make sure to **not commit this file** to git or svn, as it
contains local information and might confuse another editor instance in
another computer.
