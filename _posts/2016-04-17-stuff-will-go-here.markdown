---
layout: post
title: "Stuff will go here"
date: 2016-04-17 23:52:54 -0700
tags: miscpost,useless
----------------------
Soon I'll be writing about my programming experiences here. I'm just too tired to do that right now.

Here is a test code snippet:

{% highlight scala %}
def loadShaderFile(file: String, glShaderType: Int): Int = {
    val contents = Source.fromFile(file).getLines().mkString("\n")
    val shaderId = GL20.glCreateShader(glShaderType)

    GL20.glShaderSource(shaderId, contents)
    GL20.glCompileShader(shaderId)

    if (GL20.glGetShaderi(shaderId, GL20.GL_COMPILE_STATUS) != GL11.GL_TRUE) {
      throw new OpenGLException(GL20.glGetShaderInfoLog(shaderId))
    }
    GLUtil.checkGLError()

    shaderId
  }
{% endhighlight %}

Anyways, good night world!
