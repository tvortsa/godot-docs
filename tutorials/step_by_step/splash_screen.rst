.. _doc_splash_screen:

Splash screen
=============

Tutorial
--------

Это простой урок представляющий базовую идею функционирования
подсистемы GUI. Его цель создать действительно простой, статический
splash screen.

Ниже приведен файл с используемыми активами. Вы можете его просто положить в папку проекта
- не нужно его импортировать:

:download:`robisplash_assets.zip </files/robisplash_assets.zip>`.

Настройки
----------

Задайте разрешение экрана 800x450 в Project Settings, и настройте новую сцену так:

.. image:: /img/robisplashscene.png

.. image:: /img/robisplashpreview.png

Узлы "background" и "logo" имеют тип :ref:`TextureFrame <class_TextureFrame>`.
Есть специальное свойство для установки текстуры которая должна отображаться,
просто загрузите соответствующий файл.

.. image:: /img/texframe.png

Узел "start" это :ref:`TextureButton <class_TextureButton>`.
У него есть несколько изображений для разных состояний,Нов этом примере
мы используем только normal и pressed:

.. image:: /img/texbutton.png

Наконец, узел "copyright" это :ref:`Label <class_Label>`.
Собственный шрифт можно указать для labels изменяя следующее свойство:

.. image:: /img/label.png

В качестве примечания, шрифт был импортирован из TTF, см :ref:`doc_importing_fonts`.
