# Analizador léxico y sintáctico LR(1) con árbol de análisis

Proyecto académico en Python para implementar las fases de análisis léxico y análisis sintáctico de un lenguaje de programación simplificado. El analizador léxico genera tokens a partir de un archivo fuente; el parser LR(1) utiliza la tabla entregada por el profesor para aceptar o rechazar la secuencia. Durante las reducciones, el parser construye un **árbol de análisis sintáctico**.

> Este proyecto cubre análisis léxico y sintáctico. No realiza análisis semántico, generación de código ni ejecución del programa.

## Archivos

| Archivo | Función |
|---|---|
| `SintactObjetos_con_arbol.py` | Programa principal: tokens, lexer, parser LR(1) y árbol. |
| `compilador.lr` | Tabla LR(1) y metadatos de las reglas; el programa lo lee. |
| `compilador.inf` | Referencia de categorías/números de tokens y producciones gramaticales. |
| `programa_valido.txt` | Programa sencillo aceptado. |
| `prueba_arbol_expresiones.txt` | Prueba con declaración, asignación y expresión aritmética. |
| `error_lexico.txt` | Prueba con un carácter no reconocido. |
| `error_sintactico.txt` | Prueba con una estructura sintáctica incorrecta. |
| `salida_arbol_valido.txt` | Salida de la prueba válida, incluyendo el árbol. |
| `salida_arbol_expresiones.txt` | Salida de una expresión aritmética, incluyendo el árbol. |
| `salida_error_lexico.txt` | Salida de la prueba de error léxico. |
| `salida_error_sintactico.txt` | Salida de la prueba de error sintáctico. |

## Requisitos

- Python 3.
- No se requieren paquetes externos.
- Mantén `compilador.lr` en la misma carpeta que el script o especifica su ubicación con `--lr`.

## Ejecución

Abre una terminal en la carpeta del proyecto y ejecuta uno de estos comandos:

```bash
python SintactObjetos_con_arbol.py programa_valido.txt
python SintactObjetos_con_arbol.py prueba_arbol_expresiones.txt
python SintactObjetos_con_arbol.py error_lexico.txt
python SintactObjetos_con_arbol.py error_sintactico.txt
```

Para utilizar otra ubicación para la tabla LR:

```bash
python SintactObjetos_con_arbol.py programa_valido.txt --lr ruta/a/compilador.lr
```

Para guardar la salida de una ejecución:

```bash
python SintactObjetos_con_arbol.py programa_valido.txt > salida_arbol_valido.txt
```

El primer argumento indica el archivo fuente que se analizará. No se elige automáticamente: se indica en cada ejecución.

## Flujo del programa

```text
Archivo fuente (.txt)
        |
        v
Analizador léxico
        |
        v
Lista de objetos Token
        |
        +---- Si hay error léxico: informar y detener
        |
        v
Parser LR(1) + compilador.lr
        |
        +---- Error sintáctico: informar rechazo
        |
        v
Reducciones de la gramática
        |
        v
Árbol de análisis sintáctico
```

## Tokens

Los números de token corresponden a las categorías de `compilador.inf`:

| Categoría | Número |
|---|---:|
| `identificador` | 0 |
| `entero` | 1 |
| `real` | 2 |
| `cadena` | 3 |
| `tipo` | 4 |
| `opSuma` | 5 |
| `opMul` | 6 |
| `opRelac` | 7 |
| `opOr` | 8 |
| `opAnd` | 9 |
| `opNot` | 10 |
| `opIgualdad` | 11 |
| `;` | 12 |
| `,` | 13 |
| `(` | 14 |
| `)` | 15 |
| `{` | 16 |
| `}` | 17 |
| `=` | 18 |
| `if` | 19 |
| `while` | 20 |
| `return` | 21 |
| `else` | 22 |
| `$` | 23 |

La clase `Token` guarda categoría numérica, lexema, línea, columna y nombre de categoría. El lexer agrega `$` al final para marcar el fin de entrada de la tabla LR.

**Nota de especificación:** `compilador.inf` especifica las categorías y sus números, pero no enumera en ese archivo todos los lexemas posibles de `tipo` y de las categorías de operadores. El código usa provisionalmente los tipos `int`, `float`, `char`, `void`, operadores aritméticos (`+`, `-`, `*`, `/`), relacionales (`<`, `>`, `<=`, `>=`), igualdad (`==`, `!=`) y lógicos (`&&`, `||`, `!`). Hay que confirmar estas asociaciones con la especificación completa del curso.

## Analizador sintáctico LR(1)

La clase `ParserLR` lee `compilador.lr`, que contiene 52 reglas, una tabla de 95 estados y 46 columnas. Las columnas de terminales se consultan mediante el número del token; las columnas restantes se utilizan para transiciones `GOTO` a no terminales.

- **Shift (desplazamiento):** consume el token y almacena el terminal y el nuevo estado en la pila.
- **Reduce (reducción):** retira de la pila los símbolos del lado derecho de una producción y realiza la transición `GOTO` del no terminal resultante.
- **Aceptación:** indica que la secuencia pertenece al lenguaje descrito por la gramática.
- **Error:** informa el token inesperado y su posición.

## Construcción del árbol

La clase `Nodo` representa un nodo con:

- `nombre`: nombre del símbolo terminal o no terminal.
- `valor`: lexema para hojas terminales; `ε` para producciones vacías.
- `hijos`: lista de nodos hijos.

Cuando el parser hace `shift`, crea una hoja para el token. Cuando aplica una reducción, recupera los nodos de los símbolos reducidos, conserva su orden y crea un nodo padre con esos hijos. Al aceptar la gramática, el nodo raíz es `<programa>` y se imprime el árbol.

El árbol producido es un **árbol de análisis sintáctico concreto**: conserva los no terminales, terminales y producciones vacías de la gramática. No es todavía un AST simplificado para análisis semántico o generación de código.

## Estructuras y clases principales

| Clase | Responsabilidad |
|---|---|
| `Alumno`, `Bachillerato`, `Licenciatura` | Ejemplo de herencia y polimorfismo conservado del proyecto original. |
| `ElementoPila` | Clase base de los elementos de la pila LR. |
| `Terminal` | Símbolo terminal almacenado en la pila; conserva su nodo del árbol. |
| `NoTerminal` | Símbolo producido por una reducción; conserva el nodo padre creado. |
| `Estado` | Estado de la tabla LR. |
| `Pila` | Estructura LIFO para el algoritmo LR. |
| `Token` | Información de cada token léxico. |
| `Nodo` | Nodo del árbol sintáctico, con nombre, valor e hijos. |
| `Lexico` | Lee el archivo y genera la secuencia de tokens. |
| `ParserLR` | Carga la tabla LR y realiza shift/reduce. |

## Pruebas

### Entrada válida

`programa_valido.txt`:

```c
int main() {
    return 0;
}
```

Resultado esperado: aceptación y construcción de un árbol con raíz `programa`.

### Expresiones y árbol

`prueba_arbol_expresiones.txt`:

```c
int main() {
    int x;
    x = 2 + 3 * 4;
    return x;
}
```

Resultado esperado: aceptación y árbol donde se observan los nodos de suma y multiplicación según la estructura derivada por la tabla LR.

### Error léxico

`error_lexico.txt` contiene el carácter `@`. Resultado esperado: mensaje de error con línea y columna; el parser no se ejecuta.

### Error sintáctico

`error_sintactico.txt` contiene paréntesis y bloque mal formados. Resultado esperado: rechazo por el parser, indicando el token inesperado y su ubicación.

## Limitaciones y trabajo futuro

- Confirmar con el profesor los lexemas exactos asociados a `tipo` y a las categorías de operadores.
- El árbol actual es concreto; si se solicita un árbol de sintaxis abstracta (AST), se deberá simplificar la estructura y omitir nodos gramaticales auxiliares.
- No se realiza análisis semántico (declaración/uso de variables, compatibilidad de tipos o definición de funciones).
- No se genera código intermedio ni código ejecutable.

## Autor
Ian Alexandro de la Paz Mendoza
Proyecto académico desarrollado en Python para la práctica de análisis léxico, análisis sintáctico LR(1), estructuras de datos y programación orientada a objetos.
