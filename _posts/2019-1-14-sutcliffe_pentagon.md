---
layout: post
title: "Sutcliffe Pentagon is it a thing?"
date: 2019-1-14 06:00:00
categories: jruby_art update
keywords: jruby_art, Vec2D, pentagon, recursion, fractal, sutcliffe
---

Recently suzy on the processing forum was asking how to create a Sutcliffe pentagon, this got me thinking is there such a thing.  So far the only reference I have found to it, was in an extract of [Matt Pearsons Book][book] Generative Art book. Anyway it looked interesting so I had go at coding it in JRubyArt (using some logic from a recursive pentagon sketch by C. Andrews on openprocessing). The sketch features a couple of nifty JRubyArt features including Vec2D (whose methods can be chained, _and just look like math_) DegLUT tables, and a handy way of creating an enumerable colour palette from an array of web strings. The other quite useful feature (missing in vanilla processing) is the ability to create a vertex directly from the vector.

### sutcliffe.rb
```ruby
# Sutcliffe Pentagons (a JRubyArt Sketch by Martin Prout)

load_library :color_group
attr_reader :strut_factor, :renderer, :cols
PALETTE = %w[#fff0a5 #ffd500 #594a00 #9999ff #000059].freeze

def setup
  sketch_title 'Recursive Pentagons'
  @strut_factor = 0.2
  @renderer = AppRender.new self # so we can send Vec2D :to_vertex
  group = ColorGroup.from_web_array(PALETTE.to_java(:string))
  @cols = group.colors
  background(cols.last)
  no_loop
end

def draw
  translate(width / 2, height / 2)
  angle = TWO_PI / 5
  radius = width / 2
  points = (0...5).map do |i|
    x = radius * cos(angle * i)
    y = radius * sin(angle * i)
    Vec2D.new(x, y)
  end
  fractal = Pentagon.new(points, 4)
  fractal.draw
  save_frame(data_path('sutcliffe.png'))
end

def settings
  size(800, 800)
end

# Here we include Processing::Proxy to mimic vanilla processing inner class
# access.
class Pentagon
  include Processing::Proxy
  attr_reader :points ,:branches, :level, :midpoints, :innerpoints

  def initialize(points, levels)
    @points = points
    @level = levels
    return if level.zero? # so called guard clause in ruby simplifies code
    # find the midpoints on each edge
    @midpoints = (0...5).map do |i|
      midpoint(points[i], points[(i + 1) % 5])
    end
    # find the inner points
    @innerpoints = (0...5).map do |i|
      opposite = points[(i + 3) % 5]
      x = midpoints[i].x + (opposite.x - midpoints[i].x) * strut_factor
      y = midpoints[i].y + (opposite.y - midpoints[i].y) * strut_factor
      Vec2D.new(x, y)
    end
    # Create the Pentagon objects representing the six inner
    # pentagons (`logic stolen from C. Andrews`)
    # the shape is very regular, so we can build the ring of five
    @branches = (0...5).map do |i|
      p = [
        midpoints[i],
        innerpoints[i],
        innerpoints[(i + 1) % 5],
        midpoints[(i + 1) % 5],
        points[(i + 1) % 5]
      ]
      Pentagon.new(p, level - 1)
    end
    # add the final innermost pentagon
    branches << Pentagon.new(innerpoints, level - 1)
  end
  # This is a simple helper function that takes in two points (as Vec2D) and
  # returns the midpoint between them as Vec2D.
  def midpoint(point1, point2)
    (point2 + point1) * 0.5
  end
  # This draws the fractal. If this is on level 0, we just draw the
  # pentagon formed by the points. When not level 0, iterate through the
  # six branches and tell them to draw themselves.
  def draw
    no_fill
    begin_shape
    stroke_weight level
    stroke cols[level]
    points.each do |point|
      point.to_vertex(renderer)
    end
    end_shape CLOSE
    return if level.zero?

    branches.each(&:draw)
  end
end

```

[book]:https://www.manning.com/books/generative-art

![sutcliffe.png]({{site.github.url}}/assets/sutcliffe.png)
