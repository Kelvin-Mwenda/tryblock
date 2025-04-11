#include <GLFW/glfw3.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

// Define M_PI if not available
#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif

// Window dimensions
const int WIDTH = 800;
const int HEIGHT = 600;

// Light position
GLfloat light_position[] = {5.0, 5.0, 5.0, 1.0};

// Light components
GLfloat light_ambient[] = {0.2, 0.2, 0.2, 1.0};
GLfloat light_diffuse[] = {1.0, 1.0, 1.0, 1.0};
GLfloat light_specular[] = {1.0, 1.0, 1.0, 1.0};

// Material properties
GLfloat mat_ambient[] = {0.2, 0.2, 0.2, 1.0};
GLfloat mat_diffuse[] = {0.8, 0.8, 0.8, 1.0};
GLfloat mat_specular[] = {1.0, 1.0, 1.0, 1.0};
GLfloat mat_shininess[] = {50.0};

// Rotation variables
static GLfloat rotX = 0.0;
static GLfloat rotY = 0.0;

// Error callback
void errorCallback(int error, const char *description)
{
  fprintf(stderr, "GLFW Error: %s\n", description);
}

// Custom perspective function
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

// Custom lookAt function
void lookAt(float eyeX, float eyeY, float eyeZ,
            float centerX, float centerY, float centerZ,
            float upX, float upY, float upZ)
{
  float forward[3] = {centerX - eyeX, centerY - eyeY, centerZ - eyeZ};
  float up[3] = {upX, upY, upZ};
  float right[3];

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
      right[0], up[0], -forward[0], 0.0f,
      right[1], up[1], -forward[1], 0.0f,
      right[2], up[2], -forward[2], 0.0f,
      0.0f, 0.0f, 0.0f, 1.0f};

  glMultMatrixf(matrix);

  // Apply translation
  glTranslatef(-eyeX, -eyeY, -eyeZ);
}

// Draw a sphere with Phong shading
void drawPhongSphere()
{
  int slices = 40;
  int stacks = 40;
  float radius = 1.0;

  for (int i = 0; i < stacks; i++)
  {
    float theta1 = i * M_PI / stacks;
    float theta2 = (i + 1) * M_PI / stacks;

    glBegin(GL_TRIANGLE_STRIP);
    for (int j = 0; j <= slices; j++)
    {
      float phi = j * 2 * M_PI / slices;

      float x1 = radius * sin(theta1) * cos(phi);
      float y1 = radius * cos(theta1);
      float z1 = radius * sin(theta1) * sin(phi);

      float x2 = radius * sin(theta2) * cos(phi);
      float y2 = radius * cos(theta2);
      float z2 = radius * sin(theta2) * sin(phi);

      glNormal3f(x1, y1, z1);
      glVertex3f(x1, y1, z1);

      glNormal3f(x2, y2, z2);
      glVertex3f(x2, y2, z2);
    }
    glEnd();
  }
}

// Set up projection matrix
void setupProjection(GLFWwindow *window)
{
  int width, height;
  glfwGetFramebufferSize(window, &width, &height);

  glViewport(0, 0, width, height);

  glMatrixMode(GL_PROJECTION);
  glLoadIdentity();

  float aspect = (float)width / (float)height;
  perspective(45.0f, aspect, 0.1f, 100.0f);

  glMatrixMode(GL_MODELVIEW);
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
  glClearColor(0.0, 0.0, 0.0, 0.0);

  glEnable(GL_DEPTH_TEST);
  glEnable(GL_LIGHTING);
  glEnable(GL_LIGHT0);

  glLightfv(GL_LIGHT0, GL_POSITION, light_position);
  glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient);
  glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse);
  glLightfv(GL_LIGHT0, GL_SPECULAR, light_specular);

  glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient);
  glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse);
  glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular);
  glMaterialfv(GL_FRONT, GL_SHININESS, mat_shininess);
}

int main()
{
  glfwSetErrorCallback(errorCallback);

  if (!glfwInit())
  {
    fprintf(stderr, "Failed to initialize GLFW\n");
    return -1;
  }

  GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "Phong Shading Example", NULL, NULL);
  if (!window)
  {
    fprintf(stderr, "Failed to open GLFW window\n");
    glfwTerminate();
    return -1;
  }

  glfwMakeContextCurrent(window);
  glfwSetKeyCallback(window, keyCallback);

  init();

  while (!glfwWindowShouldClose(window))
  {
    rotY += 0.1;
    if (rotY > 360.0)
      rotY -= 360.0;

    setupProjection(window);

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glLoadIdentity();

    lookAt(0.0, 0.0, 5.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0);

    glRotatef(rotX, 1.0, 0.0, 0.0);
    glRotatef(rotY, 0.0, 1.0, 0.0);

    drawPhongSphere();

    glfwSwapBuffers(window);
    glfwPollEvents();
  }

  glfwTerminate();
  return 0;
}