1. **¿Cuál es la diferencia entre Working Directory, Staging Area y Local Repository? Da un ejemplo de un archivo pasando por las tres.**

   La diferencia principal es el estado en el que está el archivo con el que estemos trabajando/modificando:
   - **Working Directory:** Aquí podemos encontrar el archivo que estamos creando/modificando, por ejemplo, si creamos un archivo `README.md` con el texto “hola”, ese archivo quedará en nuestro entorno de trabajo.
   - **Staging Area:** Es una zona intermedia entre nuestro entorno de trabajo y el repositorio local que tenemos (la versión final del proyecto). En nuestro caso podemos pasarlo a staging área mediante `git add README.md`.  
     Es una zona de preparación para seleccionar qué cambios irán en el próximo commit.
   - **Local Repository:** Es donde tenemos ya los cambios realizados en nuestro archivo sobre el proyecto principal, será donde tenemos la versión final del proyecto. En nuestro caso meteríamos el cambio de `README.md` realizando un commit sobre `main` con un comentario, por ejemplo `git commit -m "docs: se agrega README.md"`. Si estamos en otra rama tenemos que hacer un merge con `main` (nuestra rama principal).

2. **Si modificas un archivo pero no haces git add, ¿aparece ese cambio en tu próximo commit? Explica por qué.**

   No aparece porque no ha pasado todavía a la staging área.

3. **¿Por qué git status no mostraba las carpetas vacías que creaste en la Parte C? ¿Qué truco usamos para solucionarlo?**

   Porque git solo detecta cambios en los archivos, si tengo un directorio vacío no lo detecta. Para ello podemos agregarlos creando un archivo oculto `.gitkeep` dentro de los mismos -> `touch nuestro_directorio/.gitkeep`.

4. **Explica con tus palabras qué es HEAD.**

   Head es la última versión (el último commit) de la rama, funciona como un puntero o indicador que señala en qué punto o commit del historial estás.

5. **¿Qué diferencia hay entre crear una branch con git switch -c y crear una carpeta nueva con mkdir? ¿Cómo lo comprobamos en la Parte G?**

   Cuando creo una rama, creo una versión paralela de mi código (generalmente de main) que no es visible en el directorio del proyecto solo a nivel de versionado. Sin embargo al crear una carpeta nueva, si que puedo visualizarla.  
   por ejemplo utilizando `ls` sí veremos la carpeta creada con mkdir.

6. **Durante el conflicto de la Parte H, ¿qué representaba el contenido entre <<<<<<< HEAD y =======? ¿Y entre ======= y >>>>>>>?**

   El primer bloque representa el código del HEAD, es decir, del último commit que tiene conflicto en la rama que estoy trabajando. Y el segundo, la parte con la que choca cuando haces el merge, en este caso es la rama que estoy integrando. En definitiva,
   primero me dice el código de mi rama actual y después el código de la rama que estoy integrando y que genera el conflicto a resolver.  

7. **¿Por qué NO se debe hacer git commit --amend sobre un commit que ya se subió con git push?**

   Porque cambias el hash del último commit reescibiendo el historial y si otra persona ha hecho un pull del remoto con ese commit cuando vuelva a subirlo haciendo push provocará un conflicto. Rompiendo la sincronización del repositorio
   al resto de personas.

8. **Si borras por accidente la carpeta .git de tu proyecto, ¿qué se pierde exactamente? ¿Se pierde también el código fuente que está en el disco?**

   Perderás la staging área y el local repository. No se pierde el código del disco.

9. **Explica con tus propias palabras la diferencia entre Git y GitHub, sin usar la palabra "nube".**

   Git es una herramienta de versionado y github es una plataforma donde repositar el código, realizar ci/cd, etc.

10. **¿Por qué no se debe subir un archivo .env con contraseñas reales a un repositorio, aunque el repositorio sea privado?**

    Porque aunque sea privado o se borre después queda en el histórico de commits.

11. **Un compañero te dice: "hice push y ahora GitHub me rechaza el segundo push con 'non-fast-forward'". ¿Qué ha ocurrido probablemente y qué comando ejecutarías primero?**

    No tienes en local los mismos commits que en remoto. Deberías primero hacer un git pull para bajarte a local los commits de remoto y luego volver a hacer el git push.

12. **¿Qué tipo de Conventional Commit (feat, fix, docs, test…) usarías para: añadir un índice de rendimiento a una tabla, corregir una restricción mal definida, y actualizar el README?**

    Para mejorar el rendimiento de una tabla `perf`, corregir una restricción mal definida `fix` y actualizar README `docs`.
