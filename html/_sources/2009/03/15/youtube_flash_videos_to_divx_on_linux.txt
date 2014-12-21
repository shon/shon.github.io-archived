Youtube flash videos to DivX (on Linux)
#######################################
.. author:: Shekhar
.. tags:: mencoder, linux, youtube

This how I convert flash I usually use `Firefox VideoHelper Addon`_ to download youtube videos.  To play them on my Philips DVP5986K DVD player from USB drive, I need to convert it to DivX.

``mencoder /home/shon/Desktop/file-864260998.flv -ovc lavc -oac mp3lame -ffourcc DX50 -o out.avi``

.. _Firefox VideoHelper Addon: https://addons.mozilla.org/en-US/firefox/addon/3006

