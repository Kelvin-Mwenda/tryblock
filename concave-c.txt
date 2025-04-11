#include <GLFW/glfw3.h>
#include "stb_easy_font.h"
#include <stdlib.h>
#include <stdio.h>

// Polygon vertices
float polygon[][2] = {
    {100.0f, 200.0f},
    {100.0f, 100.0f},
    {300.0f, 100.0f},
    {200.0f, 0.0f}};
int numVertices = 4;

int width = 400, height = 300;

void drawPolygon()
{
  // Draw polygon outline in black
  glColor3f(0.0f, 0.0f, 0.0f); // Black
  glLineWidth(2.0f);

  // Polygon outline
  glBegin(GL_LINE_LOOP);
  for (int i = 0; i < numVertices; i++)
  {
    glVertex2f(polygon[i][0], polygon[i][1]);
  }
  glEnd();

  // Draw vertices as points
  glPointSize(6.0f);
  glBegin(GL_POINTS);
  for (int i = 0; i < numVertices; i++)
  {
    glVertex2f(polygon[i][0], polygon[i][1]);
  }
  glEnd();
}

void framebuffer_size_callback(GLFWwindow *window, int w, int h)
{
  glViewport(0, 0, w, h);
  glMatrixMode(GL_PROJECTION);
  glLoadIdentity();
  glOrtho(0.0, width, 0.0, height, -1.0, 1.0);
  glMatrixMode(GL_MODELVIEW);
  glLoadIdentity();
}

int main()
{
  // Initialize GLFW
  if (!glfwInit())
  {
    fprintf(stderr, "Failed to initialize GLFW\n");
    return -1;
  }

  // Create a windowed mode window and its OpenGL context
  GLFWwindow *window = glfwCreateWindow(width, height, "Concave Polygon", NULL, NULL);
  if (!window)
  {
    fprintf(stderr, "Failed to create GLFW window\n");
    glfwTerminate();
    return -1;
  }

  // Make the window's context current
  glfwMakeContextCurrent(window);
  glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

  // Set up the viewport and projection
  glViewport(0, 0, width, height);
  glMatrixMode(GL_PROJECTION);
  glLoadIdentity();
  glOrtho(0.0, width, 0.0, height, -1.0, 1.0);
  glMatrixMode(GL_MODELVIEW);
  glLoadIdentity();

  // Set background color
  glClearColor(1.0f, 1.0f, 1.0f, 1.0f); // White

  // Main loop
  while (!glfwWindowShouldClose(window))
  {
    // Clear the framebuffer
    glClear(GL_COLOR_BUFFER_BIT);

    // Draw our polygon
    drawPolygon();

    // Swap front and back buffers
    glfwSwapBuffers(window);

    // Poll for and process events
    glfwPollEvents();
  }

  glfwTerminate();
  return 0;
}