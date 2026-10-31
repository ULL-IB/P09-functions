# Práctica 09. Funciones. Expresiones constantes.

# Factor de ponderación: 8

### Objetivos
Los objetivos de esta práctica son que el alumnado:
- Sea capaz de resolver problemas sencillos en C++ usando todos los conocimientos adquiridos hasta ahora, y en particular utilizando funciones
- Diseñe, desarrolle y utilice funciones en sus programas haciendo que sus programas sean modulares

### Rúbrica de evaluacion de esta práctica
Se señalan a continuación los aspectos más relevantes (la lista no es exhaustiva) que se tendrán en cuenta a la hora de evaluar esta práctica. Se comprobará que el alumnado:
- Conoce los conceptos expuestos en el material de referencia de esta práctica
- Ha realizado todos los ejercicios propuestos en este enunciado
- Es capaz de escribir programas simples en C++ que resuelvan problemas de complejidad similar a los que se proponen en este documento
- Hace que sus programas se estructuren en torno a diferentes funciones (sean modulares)
- Todos sus programas se estructuran en directorios diferentes, cada uno con un fichero `Makefile` que automatice su compilación
- Utiliza en todos sus programas comentarios adecuados en el formato requerido por [Doxygen](https://www.doxygen.nl/index.html)
- Acredita que es capaz de subir programas a la plataforma [Jutge](https://jutge.org/) para su evaluación
- Ha incluido un comentario de prólogo en todos los ficheros (`*.cc`, `*.h`) de sus ejercicios
- Que todos los programas que desarrolla imprimen en pantalla un mensaje indicando la finalidad del programa, así como la información que precisará del usuario para su correcta ejecución
- Hace que todos los programas que se presentan para su evaluación cumplan con los estándares definidos en la [Guía de estilo de Google para C++](https://google.github.io/styleguide/cppguide.html) 
- Utiliza siempre identificadores significativos en su programa (para constantes, variables, etc.) y no utiliza nunca identificadores de una única letra, excepto tal vez para las variables que utilice para iterar en un bucle
- Acredita que es capaz de editar ficheros remotos en su VM usando Vi
- Ha realizado todos sus ejercicios en la máquina virtual Ubuntu de la asignatura
- Demuestra que es capaz de ejecutar comandos Linux en su VM

### Expresiones constantes
Como ya se estudió en clase de teoría, cuando se declara una constante usando la etiqueta `const` el compilador comprobará implícitamente si se trata de una constante *en tiempo de compilación* o *en tiempo de ejecución*. En la mayoría de los casos esto sólo es relevante para la optimización del código, pero hay casos en los que C++ requiere una expresión constante que sólo use valores definidos en tiempo de compilación. Como este tipo de constantes permiten una mejor optimización, en general siempre intentaremos que las constantes sean evaluables en tiempo de compilación.

Recuerde que, de acuerdo a la [Guía de Estilo](https://google.github.io/styleguide/cppguide.html#Constant_Names), los identificadores de constantes debieran comenzar por una *k* inicial, seguida del nombre usando *CamelCase*.

Que una constante sea evaluable en tiempo de compilación o en tiempo de ejecución depende de si su inicialización se puede realizar durante la compilación. En algunos casos, esto puede ser difícil de determinar. Por ejemplo:

``` .cpp
int variable{5};                // no es constante, no se declara como tal
const int kMyConst1{x};         // constante en ejecución, x no es constante
const int kMyConst2{5};         // constante en compilación, 5 es constante
const int kResult{getValue()};  // depende de cómo se haya definido getValue
```

Afortunadamente, se puede indicar al compilador cuándo se necesita que una constante sea evaluable en tiempo de compilación. Para ello, se usa la etiqueta `constexpr` en lugar de `const`. Una `constexpr` (*expresión constante*) sólo puede ser evaluada en tiempo de compilación. Si su inicializador no es una expresión constante, el compilador generará un error.

``` .cpp
#include <iostream>

int Five() {
  return 5;
}

int main() {
  constexpr double kGravity{9.8};  // correcto: 9.8 es una expresión constante
  constexpr int kSum{4 + 5};       // correcto: 4 + 5 es una expresión constante
  constexpr int kSomething{kSum};  // correcto: kSum es una expresión constante

  int age;
  std::cin >> age;
  constexpr int kMyAge{age};      // error: age no es una expresión constante
  constexpr int kYoung{Five()};   // error: Five() no es una expresión constante
  return 0;
}
```
Como buena práctica, todo valor que no deba ser modificado tras su inicialización debería declararse:

- como `constexpr` si su valor se conoce en tiempo de compilación
- como `const` si su valor no se conoce en tiempo de compilación

Tenga en cuenta que algunos tipos de datos que reservan memoria dinámicamente, tales como `std::string` y `std::vector`, no son compatibles con `constexpr`. Para constantes de estos tipos, se debe usar `const`. Asimismo, los parámetros de las funciones siempre se evalúan en tiempo de ejecución y, por tanto, no pueden ser declaradas como `constexpr`.

En algunos contextos, el compilador puede decidir que no es necesario evaluar una expresión en tiempo de compilación.

``` .cpp
constexpr int kIntensidad{3 + 4};  // 3 + 4 siempre se evaluará en tiempo de compilación
const int my_var{3 + 4};           // 3 + 4 siempre se evaluará en tiempo de compilación
int result{3 + 4};                 // 3 + 4 puede que se evalúe en tiempo de compilación o en tiempo de ejecución
```

Para explicar por qué esto es así, considere el siguiente ejemplo:

``` .cpp
#include <iostream>

int main() {
  const int kValor{3 + 4};    // 3 + 4 es una expresión constante
  std::cout << kValor;        // Se evalúa en tiempo de ejecución
  return 0;
}
```

`3 + 4` es una expresión constante, así que el compilador la evaluará en tiempo de compilación y la sustituirá por el valor `7`. Pero como `kValor` es también una constante en tiempo de compilación, el compilador probablemente optimizará el programa sustituyendo  `std::cout << kValor` por `std::cout << 7`, que se ejecutará en tiempo de ejecución. Sin embargo, como `kValor` sólo se usa una vez, es más probable que el compilador optimice el programa eliminando `kValor` antes de evaluar `3 + 4`:

``` .cpp
#include <iostream>

int main() {
  std::cout << 3 + 4;  // Se evalúa en tiempo de ejecución
  return 0;
}
```

La expresión `std::cout << 3 + 4` ya no es constante, así que surge la duda de si la subexpresión `3 + 4` se evaluaría en tiempo de compilación o no. En general sí se optimizará, ya que los compiladores son capaces de detectar subexpresiones constantes incluso si son parte de expresiones en tiempo de ejecución. A este proceso de optimización se le conoce como *"constant folding"* (*plegado de constantes*). Etiquetar las expresiones constantes como `constexpr` asegura que dichos valores sean reconocidos por el plegado de constantes siempre que éstas se usen en una expresión en tiempo de ejecución.

### Material de estudio complementario
Estudie todo lo que se indica en el epígrafe [Functions](https://google.github.io/styleguide/cppguide.html#Functions) de la Guía de Estilo de Google y ponga en práctica todo lo que en ella se propone. Es normal si encuentra en ese epígrafe algún contenido que aún no ha estudiado: no se preocupe. Centre su atención en aquellos aspectos que ya conozca, para consolidarlos.

Estudie del [tutorial de referencia](https://www.learncpp.com/) en la asignatura los siguientes apartados:
- [2.7 — Forward declarations and definitions](https://www.learncpp.com/cpp-tutorial/forward-declarations/)
- [A multi-file example](https://www.learncpp.com/cpp-tutorial/programs-with-multiple-code-files/) de la sección 2.8 del tutorial (omita la parte introductoria relativa a los IDEs).
- [2.11 — Header files](https://www.learncpp.com/cpp-tutorial/header-files/)
- [Local variables](https://www.learncpp.com/cpp-tutorial/local-variables/)
- [Introduction to global variables](https://www.learncpp.com/cpp-tutorial/introduction-to-global-variables/)
- [Variable shadowing (name hiding)](https://www.learncpp.com/cpp-tutorial/variable-shadowing-name-hiding/)
- [Scope, duration, and linkage summary](https://www.learncpp.com/cpp-tutorial/scope-duration-and-linkage-summary/)
- [Command line arguments](https://www.learncpp.com/cpp-tutorial/command-line-arguments/)
- [Constant expressions](https://www.learncpp.com/cpp-tutorial/constexpr-variables/)

### Ejercicios
- Al realizar los ejercicios, cree dentro de su repositorio de esta práctica un directorio diferente para cada uno de los ejercicios. Asigne a cada uno de esos directorios nombres significativos
- Automatice la compilación del programa correspondiente a cada ejercicio utilizando ficheros `Makefile`
- Haga que todos los programas tomen su entrada por la línea de comandos y, en caso de que se ejecuten sin pasarles el número adecuado de parámetros, impriman en pantalla un mensaje indicando el modo correcto de ejecutar el programa
- El código de cada uno de los programas deberá organizarse de forma modular, haciendo uso de funciones
- Cada función deberá realizar una única tarea y hacerla correctamente
- El identificador de una función debe reflejar claramente lo que la función hace

1. Escriba un programa `scalar_product.cc` que defina y utilice una función que tome como parámetros dos vectores de números reales y devuelva su producto escalar, calculado como el sumatorio del producto de sus elementos en igual posición
   
   $$\left(a_1, a_2, a_3\right) \cdot \left(b_1, b_2, b_3\right) = a_1 \times b_1 + a_2 \times b_2 + a_3 \times b_3$$
   
   Si la función detecta que los vectores no tienen la misma longitud, debe mostrar un mensaje de error y devolver -1. Para obtener los dos vectores desde la línea de comandos, recuerde que `argc` indica cuántos argumentos se han recibido, incluyendo el nombre del programa, y que ambos vectores deben tener la misma longitud ($\geq1$). Por ejemplo,
   
   ```
   ./scalar_product 1.3 -5 -7.9 0 2.4 -6.8
   ```
   
   debería producir los vectores $\langle1.3,-5,-7.9\rangle$ y $\langle0,2.4,-6.8\rangle$.

2. Escriba un programa `capitalize_vowels.cc`que incluya una función que tome como parámetro una cadena de caracteres sin espacios y la modifique de forma que queden las consonantes en minúscula y las vocales en mayúscula. Los caracteres que no sean letras deberán permanecer inalterados.
   
   ```
   Public test cases
   Input        Output
   Abecedario   AbEcEdArIO
   PyThon       pythOn
   AlFa2022     AlfA2022
   ```

3. Escriba un programa `polynomial.cc` que incluya una función C++ que reciba como parámetros un vector de coeficientes reales `coefficients` y un valor real `value` y calcule el valor del polinomio correspondiente en el valor dado, usando el siguiente esquema:
   
   $$\left(\cdots\left(\left(\left(\left(c_0 \times v + c_1\right) \times v + c_2\right) \times v + c_3\right) \times v + c_4\right) \times v + ...\right) \times v + c_{N-1}$$
   
   donde $c_0, c_1, c_2, c_3, c_4, \cdots, c_{N-1}$ son los coeficientes, $v$ es el valor y $N$ es el tamaño del vector.
   
   ```
   Public test cases
   Coefficients   Value   Output
   2 4 6 8        0       8
   1 3 5          2       15
   1 1 1 1 1      4       341
   ```
   Se sugiere utilizar los argumentos `argv[1]` a `argv[argc-2]` como coeficientes y el argumento `argv[argc-1]` como valor de evaluación.

4. Desarrolle un programa `approximate_root.cc` que incluya una función que tome como parámetro un número real mayor que cero $n$ y devuelva su raíz cuadrada con un error de $\pm\varepsilon,$ donde $\varepsilon$ (*epsilon*) es una constante real suficientemente pequeña, por ejemplo `1e-4`. La función debe usar internamente dos valores reales: la raíz a calcular $r$ y un valor de incremento $\delta,$ con cualesquiera valores iniciales (se recomienda $1.0$ para ambos). Para realizar el cálculo, siga el siguiente proceso:
   
    - Mientras que el *valor absoluto* de $(r^2-n)$ sea mayor que $\varepsilon:$
        - Mientras que $\delta$ y $(r^2-n)$ tengan *igual signo:*
            - Sume $\delta$ a $r$
        - Multiplique $\delta$ por $-0.5$
    - Al llegar a este punto, $r$ será la raíz cuadrada de $n$ con una precisión de $\pm\varepsilon$
   
   Para comprobar si dos valores reales tienen *igual signo*, cree una función booleana `IgualSigno` que devuelva *verdadero* si ambos son positivos o ambos son negativos (y, claro está, *falso* si uno de ellos es positivo y el otro es negativo). Recuerde que los identificadores de todas las variables y parámetros deben ser significativos, aunque en este enunciado se haya usado $n,r,\delta$ y $\varepsilon$ para simplificar.

5. Desarrolle un programa `hypotenuse.cc` que incluya una función que tome como parámetros reales los catetos de un triángulo rectángulo y devuelva su hipotenusa, haciendo uso de la función implementada en el ejercicio anterior para calcular la raíz cuadrada:
   
   $$hipotenusa = \sqrt{{cateto_1}^2 + {cateto_2}^2}$$
   
   ```
   Public test cases
   Input   Output
   3 4     5
   7 9     10.4018
   10 12   15.6205
   ```
