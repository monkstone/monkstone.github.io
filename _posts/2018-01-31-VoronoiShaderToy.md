---
layout: post
title: "Voronoi Shadertoy Sketch JRubyArt"
date: 2018-01-31 07:00:00
categories: jruby_art update
keywords: ruby-processing, JRubyArt, shadertoy, gpu
---
Thanks to the [PixelFlow library][pixgit] by [Thomas Diewald][diewald] here is a voronoi shadertoy example translated for [JRubyArt][jruby_art],  see more examples [here][jra] and [here][pro]

```ruby
load_library :PixelFlow
java_import 'java.nio.ByteBuffer'
java_import 'com.jogamp.opengl.GL2'
java_import 'com.thomasdiewald.pixelflow.java.DwPixelFlow'
java_import 'com.thomasdiewald.pixelflow.java.dwgl.DwGLTexture'
java_import 'com.thomasdiewald.pixelflow.java.imageprocessing.DwShadertoy'
# PixelFlow | Copyright (C) 2017 Thomas Diewald - www.thomasdiewald.com
# translated to JRubyArt by Martin Prout
# https://github.com/diwi/PixelFlow.git
#
# A Processing/Java library for high performance GPU-Computing. MIT
# License: https://opensource.org/licenses/MIT
#
# Shadertoy Demo:   https://www.shadertoy.com/view/ldl3W8
# Shadertoy Author: https://www.shadertoy.com/user/iq
attr_reader :context, :toy, :tex0

def settings
  size(1280, 720, P2D)
  smooth(0)
end

def setup
  sketch_title 'Warming Up'
  surface.set_resizable(true)
  @tex0 = DwGLTexture.new
  @context = DwPixelFlow.new(self)
  context.print
  context.printGL
  @toy = DwShadertoy.new(context, data_path('voronoi_distances.frag'))
  # create noise texture
  wh = 256
  bdata = []
  (0...wh * wh * 4).step(4) do
    bdata << rand(-125..125) # NB: java bytes are signed
    bdata << rand(-125..125)
    bdata << rand(-125..125)
    bdata << 125
  end
  bbuffer = Java::JavaNio::ByteBuffer.wrap(bdata.to_java(Java::byte))
  tex0.resize(
    context,
    GL2::GL_RGBA8,
    wh,
    wh,
    GL2::GL_RGBA,
    GL2::GL_UNSIGNED_BYTE,
    GL2::GL_LINEAR,
    GL2::GL_MIRRORED_REPEAT,
    4,
    1,
    bbuffer
  )
  frame_rate(60)
end

def draw
  toy.set_iMouse(mouse_x, height - 1 - mouse_y, mouse_x, height - 1 - mouse_y)
  toy.set_iChannel(0, tex0)
  toy.apply(self.g)
  title_format = 'Shadertoy Voronoi Distances | size: [%d, %d] frame_count: %d fps: %6.2f'
  surface.set_title(format(title_format, width, height, frame_count, frame_rate))
end

```

### A snapshot of a running sketch from atom

<img src="/assets/voronoi_distance.png" />

[pixgit]:https://github.com/diwi/PixelFlow
[diewald]:http://thomasdiewald.com/blog/
[jruby_art]:https://ruby-processing.github.io/JRubyArt/
[jra]:https://github.com/ruby-processing/JRubyArt-examples/tree/master/external_library/java/PixelFlow
[propane]:https://ruby-processing.github.io/propane/
[pro]:https://github.com/ruby-processing/propane-examples/tree/master/external_library/java/pixel_flow