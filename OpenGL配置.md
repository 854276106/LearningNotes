### c/c++

* 常规
  * 附加包含目录：$(SolutionDir)Dependencies\GLEW\include;$(SolutionDir)Dependencies\GLFW\include;$(SolutionDir)Dependencies\glm;
* 预处理器
  * 预处理命令：GLEW_STATIC

### 链接器

* 常规

  * 附加库目录：$(SolutionDir)Dependencies\GLEW\lib\Release\x64;$(SolutionDir)Dependencies\GLFW\lib-vc2019

* 输入

  * 附加依赖项：glew32s.lib;glfw3.lib;opengl32.lib;User32.lib;Gdi32.lib;Shell32.lib;

    