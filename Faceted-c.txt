#include <GLFW/glfw3.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

// For text rendering
#define STB_EASY_FONT_IMPLEMENTATION
#include "stb_easy_font.h"

// Define M_PI if not available
#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif

// Window dimensions
const int WIDTH = 800;
const int HEIGHT = 600;

// Light position
GLfloat light_position[] = {5.0, 5.0, 5.0, 0.0};

// Light components
GLfloat light_ambient[] = {0.2, 0.2, 0.2, 1.0};
GLfloat light_diffuse[] = {1.0, 1.0, 1.0, 1.0};
GLfloat light_specular[] = {1.0, 1.0, 1.0, 1.0};

// Material properties
GLfloat mat_ambient[] = {0.7, 0.7, 0.7, 1.0};
GLfloat mat_diffuse[] = {0.8, 0.8, 0.8, 1.0};
GLfloat mat_specular[] = {1.0, 1.0, 1.0, 1.0};
GLfloat mat_shininess[] = {100.0};

// Rotation variables
static GLfloat rotX = 0.0;
static GLfloat rotY = 0.0;

// Error callback
void errorCallback(int error, const char *description)
{
  fprintf(stderr, "GLFW Error: %s\n", description);
}

// Function to calculate the normal vector for a triangle
void calculateNormal(GLfloat *v1, GLfloat *v2, GLfloat *v3, GLfloat *normal)
{
  GLfloat vector1[3], vector2[3];

  // Calculate two vectors from the three points
  vector1[0] = v2[0] - v1[0];
  vector1[1] = v2[1] - v1[1];
  vector1[2] = v2[2] - v1[2];

  vector2[0] = v3[0] - v1[0];
  vector2[1] = v3[1] - v1[1];
  vector2[2] = v3[2] - v1[2];

  // Calculate the cross product to get the normal vector
  normal[0] = vector1[1] * vector2[2] - vector1[2] * vector2[1];
  normal[1] = vector1[2] * vector2[0] - vector1[0] * vector2[2];
  normal[2] = vector1[0] * vector2[1] - vector1[1] * vector2[0];

  // Normalize the normal vector
  float length = sqrt(normal[0] * normal[0] +
                      normal[1] * normal[1] +
                      normal[2] * normal[2]);

  if (length != 0.0)
  {
    normal[0] /= length;
    normal[1] /= length;
    normal[2] /= length;
  }
}

// Draw a cube with faceted shading
void drawFacetedCube()
{
  // Vertices of a cube
  GLfloat vertices[][3] = {
      {-1.0, -1.0, -1.0}, {1.0, -1.0, -1.0}, {1.0, 1.0, -1.0}, {-1.0, 1.0, -1.0}, {-1.0, -1.0, 1.0}, {1.0, -1.0, 1.0}, {1.0, 1.0, 1.0}, {-1.0, 1.0, 1.0}};

  // Define the 6 faces of the cube, each with 2 triangles
  GLint faces[][3] = {
      {0, 1, 2}, {0, 2, 3}, // Front face
      {4, 5, 6},
      {4, 6, 7}, // Back face
      {0, 3, 7},
      {0, 7, 4}, // Left face
      {1, 5, 6},
      {1, 6, 2}, // Right face
      {3, 2, 6},
      {3, 6, 7}, // Top face
      {0, 4, 5},
      {0, 5, 1} // Bottom face
  };

  // Materials with different colors for each face
  GLfloat materials[][4] = {
      {1.0, 0.0, 0.0, 1.0}, // Red
      {0.0, 1.0, 0.0, 1.0}, // Green
      {0.0, 0.0, 1.0, 1.0}, // Blue
      {1.0, 1.0, 0.0, 1.0}, // Yellow
      {1.0, 0.0, 1.0, 1.0}, // Magenta
      {0.0, 1.0, 1.0, 1.0}  // Cyan
  };

  // Draw all 12 triangles (6 faces, 2 triangles each)
  for (int i = 0; i < 12; i++)
  {
    GLfloat normal[3];
    int faceIndex = i / 2; // Determine which face we're drawing

    // Calculate the normal for this triangle
    calculateNormal(vertices[faces[i][0]],
                    vertices[faces[i][1]],
                    vertices[faces[i][2]],
                    normal);

    // Set the material color for this face
    GLfloat *color = materials[faceIndex];
    GLfloat mat_diffuse[] = {color[0], color[1], color[2], color[3]};
    glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse);

    // Begin drawing a triangle
    glBegin(GL_TRIANGLES);

    // For faceted shading, we set the normal once for the entire face
    glNormal3fv(normal);

    // Draw each vertex of the triangle
    glVertex3fv(vertices[faces[i][0]]);
    glVertex3fv(vertices[faces[i][1]]);
    glVertex3fv(vertices[faces[i][2]]);

    glEnd();
  }
}

// Define our own perspective function (since we can't use gluPerspective)
void perspective(float fovy, float aspect, float near, float far)
{
  float f = 1.0f / tanf(fovy * 0.5f * M_PI / 180.0f);
  float nf = 1.0f / (near - far);

  float matrix[16] = {0};
  matrix[0] = f / aspect;
  matrix[5] = f;
  matrix[10] = (far + near) * nf;
  matrix[11] = -1.0f;
  matrix[14] = 2.0f * far * near * nf;

  glMultMatrixf(matrix);
}

// Define our own lookAt function (since we can't use gluLookAt)
void lookAt(float eyeX, float eyeY, float eyeZ,
            float centerX, float centerY, float centerZ,
            float upX, float upY, float upZ)
{

  // Create forward (z), right (x), and up (y) vectors
  float forward[3] = {eyeX - centerX, eyeY - centerY, eyeZ - centerZ};
  float right[3];
  float up[3] = {upX, upY, upZ};

  // Normalize forward vector
  float length = sqrtf(forward[0] * forward[0] +
                       forward[1] * forward[1] +
                       forward[2] * forward[2]);
  forward[0] /= length;
  forward[1] /= length;
  forward[2] /= length;

  // Calculate right vector as forward x up
  right[0] = up[1] * forward[2] - up[2] * forward[1];
  right[1] = up[2] * forward[0] - up[0] * forward[2];
  right[2] = up[0] * forward[1] - up[1] * forward[0];

  // Normalize right vector
  length = sqrtf(right[0] * right[0] + right[1] * right[1] + right[2] * right[2]);
  right[0] /= length;
  right[1] /= length;
  right[2] /= length;

  // Recalculate up vector as right x forward
  up[0] = forward[1] * right[2] - forward[2] * right[1];
  up[1] = forward[2] * right[0] - forward[0] * right[2];
  up[2] = forward[0] * right[1] - forward[1] * right[0];

  // Create rotation matrix
  float matrix[16] = {
      right[0], up[0], forward[0], 0.0f,
      right[1], up[1], forward[1], 0.0f,
      right[2], up[2], forward[2], 0.0f,
      0.0f, 0.0f, 0.0f, 1.0f};

  // Apply rotation
  glMultMatrixf(matrix);

  // Apply translation
  glTranslatef(-eyeX, -eyeY, -eyeZ);
}

// Set up projection matrix
void setupProjection(GLFWwindow *window)
{
  int width, height;
  glfwGetFramebufferSize(window, &width, &height);

  glViewport(0, 0, width, height);

  // Set perspective projection
  glMatrixMode(GL_PROJECTION);
  glLoadIdentity();

  // Calculate aspect ratio
  float aspect = (float)width / (float)height;

  // Use our own perspective function
  perspective(45.0f, aspect, 0.1f, 100.0f);

  // Switch back to modelview matrix
  glMatrixMode(GL_MODELVIEW);
}

// Render text using stb_easy_font
void drawText(GLFWwindow *window, const char *text, int x, int y)
{
  int fb_width, fb_height;
  glfwGetFramebufferSize(window, &fb_width, &fb_height);

  // Save current state
  glPushMatrix();
  glPushAttrib(GL_ALL_ATTRIB_BITS);

  // Setup orthographic projection for text rendering
  glMatrixMode(GL_PROJECTION);
  glLoadIdentity();
  glOrtho(0, fb_width, fb_height, 0, -1, 1);

  glMatrixMode(GL_MODELVIEW);
  glLoadIdentity();

  // Disable lighting for text
  glDisable(GL_LIGHTING);
  glDisable(GL_DEPTH_TEST);

  // Set text color to white
  glColor3f(1.0f, 1.0f, 1.0f);

  // Buffer for vertex data
  static char buffer[99999];
  int num_quads;

  // Generate text quads (cast text to char *)
  num_quads = stb_easy_font_print(x, y, (char *)text, NULL, buffer, sizeof(buffer));

  // Draw text
  glEnableClientState(GL_VERTEX_ARRAY);
  glVertexPointer(2, GL_FLOAT, 16, buffer);
  glDrawArrays(GL_QUADS, 0, num_quads * 4);
  glDisableClientState(GL_VERTEX_ARRAY);

  // Restore previous state
  glPopAttrib();
  glPopMatrix();
}

// Key callback function
void keyCallback(GLFWwindow *window, int key, int scancode, int action, int mods)
{
  if (action == GLFW_PRESS || action == GLFW_REPEAT)
  {
    switch (key)
    {
    case GLFW_KEY_ESCAPE:
      glfwSetWindowShouldClose(window, GLFW_TRUE);
      break;
    case GLFW_KEY_W:
      rotX += 1.0;
      break;
    case GLFW_KEY_S:
      rotX -= 1.0;
      break;
    case GLFW_KEY_A:
      rotY -= 1.0;
      break;
    case GLFW_KEY_D:
      rotY += 1.0;
      break;
    }
  }
}

// Initialize OpenGL settings
void init()
{
  // Set background color to black
  glClearColor(0.0, 0.0, 0.0, 0.0);

  // Enable depth testing
  glEnable(GL_DEPTH_TEST);

  // Enable lighting
  glEnable(GL_LIGHTING);
  glEnable(GL_LIGHT0);

  // Set light properties
  glLightfv(GL_LIGHT0, GL_POSITION, light_position);
  glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient);
  glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse);
  glLightfv(GL_LIGHT0, GL_SPECULAR, light_specular);

  // Set material properties
  glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient);
  glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular);
  glMaterialfv(GL_FRONT, GL_SHININESS, mat_shininess);

  // Enable color tracking
  glEnable(GL_COLOR_MATERIAL);
  glColorMaterial(GL_FRONT, GL_DIFFUSE);
}

int main()
{
  // Set error callback
  glfwSetErrorCallback(errorCallback);

  // Initialize GLFW
  if (!glfwInit())
  {
    fprintf(stderr, "Failed to initialize GLFW\n");
    return -1;
  }

  // Create a window and its OpenGL context
  GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "Faceted Shading Example", NULL, NULL);
  if (!window)
  {
    fprintf(stderr, "Failed to open GLFW window\n");
    glfwTerminate();
    return -1;
  }

  // Make the window's context current
  glfwMakeContextCurrent(window);

  // Set up key callback
  glfwSetKeyCallback(window, keyCallback);

  // Initialize OpenGL settings
  init();

  // Main loop
  while (!glfwWindowShouldClose(window))
  {
    // Apply automatic rotation
    rotY += 0.1;
    if (rotY > 360.0)
      rotY -= 360.0;

    // Set up projection
    setupProjection(window);

    // Clear the screen
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    // Reset transformations
    glLoadIdentity();

    // Set up the camera
    lookAt(0.0, 0.0, 5.0,  // Eye position
           0.0, 0.0, 0.0,  // Look at position
           0.0, 1.0, 0.0); // Up vector

    // Apply rotations
    glRotatef(rotX, 1.0, 0.0, 0.0);
    glRotatef(rotY, 0.0, 1.0, 0.0);

    // Draw the faceted cube
    drawFacetedCube();

    // Draw title text
    drawText(window, "FACETED SHADING EXAMPLE", 10, 20);
    drawText(window, "Use WASD to rotate the cube", 10, 40);

    // Swap buffers
    glfwSwapBuffers(window);

    // Poll for events
    glfwPollEvents();
  }

  // Clean up
  glfwTerminate();
  return 0;
}
