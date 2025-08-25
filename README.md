import pygame
import moderngl
import array
import math
import sys

# Init pygame
pygame.init()
pygame.display.set_mode((800, 600), pygame.OPENGL | pygame.DOUBLEBUF)
clock = pygame.time.Clock()

# ModernGL context (linked to pygame's OpenGL window)
ctx = moderngl.create_context()

# Cube vertices (x, y, z)
vertices = array.array('f', [
    -1, -1, -1,
     1, -1, -1,
     1,  1, -1,
    -1,  1, -1,
    -1, -1,  1,
     1, -1,  1,
     1,  1,  1,
    -1,  1,  1,
])

# Indices for cube faces
indices = array.array('I', [
    0, 1, 2, 2, 3, 0,  # back
    4, 5, 6, 6, 7, 4,  # front
    0, 4, 7, 7, 3, 0,  # left
    1, 5, 6, 6, 2, 1,  # right
    3, 2, 6, 6, 7, 3,  # top
    0, 1, 5, 5, 4, 0,  # bottom
])

# Create buffers
vbo = ctx.buffer(vertices.tobytes())
ibo = ctx.buffer(indices.tobytes())

# Shaders
prog = ctx.program(
    vertex_shader='''
        #version 330
        in vec3 in_vert;
        uniform float angle;
        void main() {
            mat4 rot = mat4(
                cos(angle), 0.0, sin(angle), 0.0,
                0.0,        1.0, 0.0,        0.0,
               -sin(angle), 0.0, cos(angle), 0.0,
                0.0,        0.0, 0.0,        1.0
            );
            gl_Position = rot * vec4(in_vert * 0.5, 1.0);
        }
    ''',
    fragment_shader='''
        #version 330
        out vec4 fragColor;
        void main() {
            fragColor = vec4(0.2, 0.7, 1.0, 1.0);
        }
    '''
)

vao = ctx.vertex_array(prog, [(vbo, "3f", "in_vert")], ibo)

angle = 0.0

# Game loop
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    ctx.clear(0.1, 0.1, 0.1)
    prog["angle"].value = angle
    vao.render()
    angle += 0.02

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()
