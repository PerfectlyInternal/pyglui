pyglui is a poorly-documented mess of a library developed by Pupil Labs. This document is (hopefully) a way for us to understand and use this library for plugin development.

## Creating a menu for a plugin ##
Here's an example for how to create a menu with a slider. This is available as part of the example plugin, but is poorly explained there.
```
def init_ui(self):
                # init the ui and give the menu a name
                self.add_menu()
                self.menu.label = "Example Plugin Menu"

                # making a slider
		# the name *must* be the same as the variable it controls
		# i have no idea how or why it does that
                self.menu.append(ui.Slider("example_variable", self, min=1))
```

## Drawing things on the screen ##
Assuming you're drawing on the world camera's screen, you don't need to change GL contexts or anything of the sort. If you're drawing on the eye cam windows, good luck, because I've no clue how to do that. (perhaps digging thru some code/examples will reveal the secret?) All the following code should be put into the gl_display() method of the plugin. All of the following methods are found in `pyglui.cygl.utils`.

# Drawing points #
Works for either 2D or 3D, but 3D will behave... strangely?
```
draw_points(points, size=20, color=RGBA(1.,0.5,0.5,.5), sharpness=0.8)
```

# Drawing a circle #
Only works in 2D, would be nice if it worked in 3D
```
draw_circle(center_position=(0,0), radius=20, stroke_width=2, color=RGBA(1.,0.5,0.5,0.5), sharpness=0.8):
```

# Drawing a rounded rectangle #
Only works in 2D (why would you even want this in 3D?)
```
draw_rounded_rect(origin, size, corner_radius, color=RGBA(1.,0.5,0.5,.5), sharpness=0.8)
```

# Drawing lines #
Just like points, works in either 3D or 2D, but 3D seems broken?
```
draw_polyline(verts, thickness=1, color=RGBA(1.,0.5,0.5,.5), line_type=GL_LINE_STRIP)
```

## Drawing in 3D ##
Drawing in 3D requires a little more work than in 2D, since you need to set up the camera. This is where things get interesting, because pyglui does not directly support such concepts as "drawing stuff in 3D" despite having handling for 3D points and lines. Fortunately, PyOpenGL supplies all the necessary calls to do this, which should be used in the same gl_display method as before. If you already know how to use OpenGL 1 and 2, this is nothing new.

This bit of code resets the projection matrix, which is responsible for the camera's FOV and such. Replacing `GL_PROJECTION` with `GL_MODELVIEW` resets the model view matrix instead, which is responsible for the camera's position and rotation. The `glPushMatrix()` call allows you to take advantage of the GL matrix stack to preserve whatever was there before and put it back later with `glPopMatrix()`.
```
glMatrixMode(GL_PROJECTION)
glPushMatrix()
glLoadIdentity()
```

This, when used after setting the matrix mode to projection, will set the camera to be orthographic with the corresponding corner positions and clipping planes
```
glOrtho(left, right, bottom, top, nearPlane, farPlane)
```

This will set up a perspective camera in the same manner as above. FOV is vertical, and ratio is the aspect ratio used to determine the x FOV.
```
gluPerspective(fov, ratio, nearPlane, farPlane)
```

This will move the camera to a certain point, and point it at another point. Call this when the current matrix is the model view matrix. The eye XYZ is the position of the camera, the targ(et) XYZ is the point to look at and up XYZ is the direction to point the top of the screen at.
```
gluLookAt(eyeX, eyeY, eyeZ, targX, targY, targZ, upX, upY, upZ) 
```
For example, to put the camera at 0, 0, 0 and point it at 10, 0, 0 with "up" being positive Y,
```
gluLookAt(0, 0, 0, 10, 0, 0, 0, 1, 0)
```

Call these after you're done drawing to reset the camera to whatever it was before. This prevents problems later, probably.
```
glMatrixMode(GL_PROJECTION)
glPopMatrix()
glMatrixMode(GL_MODELVIEW)
glPopMatrix()
```

## How does Pupil Capture report gaze position? ##
Gaze position is reported via the event system, which plugins can access via the `recent_events()` callback. Long story short, it gives you XYZ coords of the gaze position, relative to the world camera. Unfortunately it is unknown what units this uses, or what the assumed FOV of the world camera is. This is to be determined via experimentation at a later date (whenever kenny gets the DG working, hopefully) UPDATE: We've roughly determined the FOV of the camera to be 40 degrees vertically. Check the source code for the calibration assistant for more details.
