---
layout: post
title: Tikz and Videos
tags: [Tikz, Latex, Animation, Video]
feature-img: "assets/img/tikz/map.jpg"
---

I am a really big fan of **Vector graphics** and I exhaustively use [Tikz](https://en.wikipedia.org/wiki/PGF/TikZ) to draw such.
However, creating videos and cool animations with Tikz and Latex in general can be a bit messy.
So I have come up with a simple but effective hack to make generating videos with Latex more easy.

The key idea is to have two separated Latex files: One that is manually created and stays the same over the whole video, and one that is procedurally generated for each frame.
The manual one is created as you would do usually write your Latex image. However, all variables that are dynamic throughout the video (e.g. color, position, opacity, etc.) are defined by placeholders. And, guess what, the second Latex file (the procedurally generated one) then fills all the placeholder for each frame.

# Drawing a frame
My starting point is the manually designed Latex file, that is based on the following scheme:
{% highlight tex %}
\documentclass[border=0cm,convert={outext=.png}]{standalone}
% Documentclass to directly create a PNG-files when invoking 'pdflatex'

\usepackage{xcolor}
\usepackage{tikz}

% Load dynamic variables
\input{dynamic.tex}

\begin{document}
\begin{tikzpicture}

% My Latex code
\node (start) at (0,0) [draw,fill=\mycolorsin] {A};
\node (start) at (2,0) [draw,fill=\mycolorcos] {B};

\end{tikzpicture}
\end{document}
{% endhighlight %}
In this case a two nodes are drawn, which colors (*\mycolor*) should be animated.

![Latex]({{ site.baseurl }}/assets/img/tikz/frame_000.png)

# Filling the placeholders

In the next step I will create a simple *python* script that generates the *dynamic.tex* file and fills all the dynamic placeholder variables.
Moreover, the program invokes the Latex compiler to generate a PNG file out of the code and stores it in a directory.

{% highlight python %}
import numpy as np
import time
import os
import subprocess
import shutil

# Create directory for the frames
if not os.path.exists('sequence'):
    os.makedirs('sequence')

# Example data for animation in range [0,100]
t = np.linspace(0,2*np.pi,100)
sin_seq = 50.0*np.sin(t)+50.0
cos_seq = 50.0*np.cos(t)+50.0

# Loop over each frame
for i in range(100):

    # Create file with for dynamic variables
    dynamic_file = open('dynamic.tex','w')
    dynamic_file.write('\\newcommand{\\mycolorsin}{green!'+str(int(round(sin_seq[i])))+'!white}\n')
    dynamic_file.write('\\newcommand{\\mycolorcos}{red!'+str(int(round(cos_seq[i])))+'!white}\n')
    dynamic_file.close()

    # Create png with pdflatex
    os.system('pdflatex -shell-escape -interaction=nonstopmode example.tex')

    # Move frame into directory with frames
    shutil.move('example.png', 'sequence/frame_'+str(i).zfill(3)+'.png')
{% endhighlight %}

Finally, all I have to do is to animate the frames with [ImageMagick](https://en.wikipedia.org/wiki/ImageMagick)
{% highlight bash %}
convert -loop 0 -delay 2 sequence/*.png animation.gif
{% endhighlight %}

![Latex]({{ site.baseurl }}/assets/img/tikz/animation.gif)

# Remarks
- Instead of using *\newcommand* for each placeholder, it might be more convinient to use *tikzset* or *tikzstyle* ([See here](https://tex.stackexchange.com/questions/52372/should-tikzset-or-tikzstyle-be-used-to-define-tikz-styles)) to define the dynamic variables
- If you want to create a video instead of an animation (mp4 instead of gif), you can easily do this with [ffmpeg](https://en.wikipedia.org/wiki/FFmpeg)
- Technically the gif animation is not a vector graphics anymore. However by tuning the *pdf-to-png* conversion density parameter, one can create an animation with an arbitrary high resolution. 
