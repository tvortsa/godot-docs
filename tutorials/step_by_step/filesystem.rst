.. _doc_filesystem:

Файловая система
==========

Введение
------------

Файловые системы это еще одна горячая тема в разработке движков. Файловая система
управляет хранением активов, и доступом к ним.
Хорошо спроектированные файловые системы также позволяют нескольким разработикам
сообща редактировать файлы ресурсов и активы при совместной работе.

Начальные версии Godot движка (and previous iterations before it was
named Godot) used a database. Assets were stored in it and assigned an
ID. Other approaches were tried as well, such as local databases, files with
metadata, etc. In the end the simple approach won and now Godot stores
all assets as files in the file system.

Реализация
--------------

Файловая система хранит ресурсы на диске. Все, от скрипта до сцены или 
PNG изображения это ресурсы с точки зрения движка. Если ресурс содержит свойства
которые ссылаются на другие ресурсы на диске пути к этим ресурсам также включаются.
If a resource has sub-resources that are built-in, the resource is
saved in a single file together with all the bundled sub-resources. For
example, a font resource is often bundled together with the font textures.

In general the Godot file system avoids using metadata files. The reason for
this is simple, existing asset managers and VCSs are simply much better than
anything we can implement, so Godot tries the best to play along with SVN,
Git, Mercurial, Perforce, etc.

Example of a file system contents:

::

    /engine.cfg
    /enemy/enemy.scn
    /enemy/enemy.gd
    /enemy/enemysprite.png
    /player/player.gd
    
engine.cfg
----------

The engine.cfg file is the project description file, and it is always found at
the root of the project. In fact its location defines where the root is. This
is the first file that Godot looks for when opening a project.

This file contains the project configuration in plain text, using the win.ini
format. Even an empty engine.cfg can function as a basic definition of a blank
project.

символ - разделитель
-------------------

Godot поддерживает только ``/`` в качестве разделителя пути. This is done for
portability reasons. All operating systems support this, even Windows,
so a path such as ``c:\project\engine.cfg`` needs to be typed as
``c:/project/engine.cfg``.

Resource path
-------------

When accessing resources, using the host OS file system layout can be
cumbersome and non-portable. To solve this problem, the special path
``res://`` was created.

The path ``res://`` will always point at the project root (where
engine.cfg is located, so in fact ``res://engine.cfg`` is always
valid).

This file system is read-write only when running the project locally from
the editor. When exported or when running on different devices (such as
phones or consoles, or running from DVD), the file system will become
read-only and writing will no longer be permitted.

User path
---------

Writing to disk is often still needed for various tasks such as saving game
state or downloading content packs. To this end, the engine ensures that there is a
special path ``user://`` that is always writable.

Host file system
---------------

Alternatively host file system paths can also be used, but this is not recommended
for a released product as these paths are not guaranteed to work on all platforms.
However, using host file system paths can be very useful when writing development
tools in Godot!

Drawbacks
---------

There are some drawbacks to this simple file system design. The first issue is that
moving assets around (renaming them or moving them from one path to another inside
the project) will break existing references to these assets. These references will
have to be re-defined to point at the new asset location.

The second is that under Windows and OSX file and path names are case insensitive.
If a developer working in a case insensitive host file system saves an asset as "myfile.PNG",
but then references it as "myfile.png", it will work just fine on their platorm, but not
on other platforms, such as Linux, Android, etc. This may also apply to exported binaries,
which use a compressed package to store all files.

It is recommended that your team clearly defines a naming convention for files when
working with Godot! One simple fool-proof convention is to only allow lowercase
file and path names.
