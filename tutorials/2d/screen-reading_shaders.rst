.. _doc_screen-reading_shaders:

Screen-reading шейдеры
======================

Введение
~~~~~~~~

Очень часто нужно сделать шейдер который считывает с того-же экрана, в
который и записывает. 3D API типа OpenGL или DirectX делают это довольно
сложно из-за внутренних аппаратных ограничений. GPU экстремально паралелен,
так что чтение и запись вызывают все виды проблемм с кэшем и когерентностью.
В результате, даже самые современные аппаратные средства не поддерживают это 
должным образом.

Обходной путь - сделать копию экрана, или части экрана,
бэк-буффером и производить чтение из него при отрисовке. Godot предоставляет
несколько инструментов упрощающих этот процесс!

инструкция шейдера TexScreen
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Godot :ref:`doc_shading_language` обладает специальной инструкцией, "texscreen",
она принимает в качестве параметра UV экрана и возвращает vec3 RGB с цветом. 
Специальный встроенный varying: SCREEN_UV можно использовать для получения UV для текущего
фрагмента. В результате, это просто 2D fragment shader:

::

    COLOR=vec4( texscreen(SCREEN_UV), 1.0 );

результат приводит к невидимому объекту, поскольку отображает
просто то что лежит за ним.
Тот же шейдер, который использует визуальный редактор, выглядит следующим образом:

.. image:: /img/texscreen_visual_shader.png

TexScreen example
~~~~~~~~~~~~~~~~~

Texscreen инструкция может быть использована для множества вещей. Есть специальное демо
для *Screen Space Shaders*, которое вы можете скачать и поразобраться.
Один шейдер просто добавляет яркость, контраст и насыщенность:

::

    uniform float brightness = 1.0; 
    uniform float contrast = 1.0;
    uniform float saturation = 1.0;

    vec3 c = texscreen(SCREEN_UV);

    c.rgb = mix(vec3(0.0), c.rgb, brightness);
    c.rgb = mix(vec3(0.5), c.rgb, contrast);
    c.rgb = mix(vec3(dot(vec3(1.0), c.rgb)*0.33333), c.rgb, saturation);

    COLOR.rgb = c;

За кулисами
~~~~~~~~~~~

While this seems magical, it's not. The Texscreen instruction, when
first found in a node that is about to be drawn, does a full-screen
copy to a back-buffer. Subsequent nodes that use texscreen() in
shaders will not have the screen copied for them, because this ends up
being very inefficient.

As a result, if shaders that use texscreen() overlap, the second one
will not use the result of the first one, resulting in unexpected
visuals:

.. image:: /img/texscreen_demo1.png

In the above image, the second sphere (top right) is using the same
source for texscreen() as the first one below, so the first one
"disappears", or is not visible.

To correct this, a
:ref:`BackBufferCopy <class_BackBufferCopy>`
node can be instanced between both spheres. BackBufferCopy can work by
either specifying a screen region or the whole screen:

.. image:: /img/texscreen_bbc.png

With correct back-buffer copying, the two spheres blend correctly:

.. image:: /img/texscreen_demo2.png

Back-buffer logic
~~~~~~~~~~~~~~~~~

So, to make it clearer, here's how the backbuffer copying logic works in
Godot:

-  If a node uses the texscreen(), the entire screen is copied to the
   back buffer before drawing that node. This only happens the first
   time, subsequent nodes do not trigger this.
-  If a BackBufferCopy node was processed before the situation in the
   point above (even if texscreen() was not used), this behavior
   described in the point above does not happen. In other words,
   automatic copying of the entire screen only happens if texscreen() is
   used in a node for the first time and no BackBufferCopy node (not
   disabled) was found before in tree-order.
-  BackBufferCopy can copy either the entire screen or a region. If set
   to only a region (not the whole screen) and your shader uses pixels
   not in the region copied, the result of that read is undefined
   (most likely garbage from previous frames). In other words, it's
   possible to use BackBufferCopy to copy back a region of the screen
   and then use texscreen() on a different region. Avoid this behavior!
