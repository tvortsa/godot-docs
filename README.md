# Документация Godot Engine

Этот реппозиторий содержит исходники документации [Godot Engine](http://godotengine.org) в формате языка разметки reStructuredText (reST).

Что означает, что они должны быть разобраны с помощью построителя документации [Sphinx](http://sphinx-doc.org/) для сборки   HTML документации на [сайте Godot](http://docs.godotengine.org).

## Contributing changes

Though arguably less convenient to edit than a wiki, this git repository is meant to receive pull requests to always improve the documentation, add new pages, etc. Having direct access to the source files in a revision control system is a big plus to ensure the quality of our documentation.

### Редактирование существующих страниц

To edit an existing page, just locate its .rst source file and open it in your favourite text editor. You can then commit the changes, push them to your fork and make a pull request. **Note that the pages in `classes/`should not be edited here, they are automatically generated from Godot's [XML class reference](https://github.com/godotengine/godot/tree/master/doc/base).**

### Добавление новых страниц

Чтобы добавить новую страницу, создайте .rst файл с соответствующим именем в разделе в который хотите добавить страницу, т.е. `tutorials/3d/light_baking.rst`. Write its content like you would do for any other file, and make sure to define a reference name for Sphinx at the beginning of the file (check other files for the syntax), based on the file name with a "doc_" prefix (e.g. `.. _doc_light_baking:`).

You should then add your page to the relevant "toctree" (table of contents). By convention, the files used to define the various levels of toctree are prefixed with an underscore, so in the above example the file should be referenced in `tutorials/3d/_3d_graphics.rst`. Just add your new filename to the list on a new line, using a relative path and no extension, e.g. here `light_baking`.

### Sphinx и синтаксис reStructuredText

Посмотрите Sphinx's [reST Primer](http://www.sphinx-doc.org/en/stable/rest.html) и [official reference](http://docutils.sourceforge.net/rst.html) чтобы узнать о синтаксисе.

Sphinx использует специальные reST комментарии to do specific operations, like defining the table of contents (`:toctree:`) or cross-referencing pages. Посмотрите [official Sphinx documentation](http://www.sphinx-doc.org/en/stable/index.html) for more details, or see how things are done in existing pages and adapt it to your needs.

### Добавление картинок и вложений

Чтобы добавить изображение, поместите его в папку `img/` с соответствующим именем и включите его в свою страницу:
```rst
.. image:: /img/image_name.png
```

Похожим образом, вы можете включить вложения (как активы и материалы для туториалов тут) поместив их в папку `files/` , и используя эту инлайн разметку:
```rst
:download:`myfilename.zip </files/myfilename.zip>`
```

## Сборка в Sphinx

Для сборки HTML сайта (или любой другой формат поддерживаемый Sphinx, типа PDF, EPUB или LaTeX), вам нужно установить [Sphinx](http://sphinx-doc.org/) >= 1.3 а так же (для HTML) тему [readthedocs.org theme](https://github.com/snide/sphinx_rtd_theme). Только Python 3 flavour was tested, though the Python 2 versions might work too.

Those tools are best installed using [pip](https://pip.pypa.io), Python's module installer. The Python 3 version might be provided (on Linux distros) as `pip3` or `python3-pip`. You can then run:

```sh
pip3 install sphinx
pip3 install sphinx_rtd_theme
```

Вы можете затем собрать HTML документацию из корневой папки этого реппозитория командой:

```sh
make html
```

Компиляция может занять время в зависимости от того как много файлов в папке `classes/` .  
Вы можете протестировать изменения открыв `_build/html/index.html` в браузере.

### Сборка в Sphinx под Windows

В Windows, вам также потребуется: 
* Загрузить установщик Python [здесь](https://www.python.org/downloads/).
* Install Python. Don't forget to check the "Add Python to PATH" box.
* Use the above `pip` commands.

Building is still done at the root folder of this repository, but with this command line instead:
```sh
sphinx-build -b html ./ _build
```

### Сборка в Sphinx и virtualenv

If you want your Sphinx installation scoped to the project, you can install it using virtualenv.
Execute this from the root folder of this repository:

```sh
virtualenv --system-site-packages env/
. env/bin/activate
pip3 install sphinx
pip3 install sphinx_rtd_theme
```

Then do `make html` like above.

## Лицензии

At the exception of the `classes/` folder, all the content of this repository is licensed under the Creative Commons Attribution 3.0 Unported license ([CC BY 3.0](https://creativecommons.org/licenses/by/3.0/)) and is to be attributed to "Juan Linietsky, Ariel Manzur and the Godot community".

The files in the `classes/` folder are derived from [Godot's main source repository](https://github.com/godotengine/godot) and are distributed under the MIT license, with the same authors as above.
