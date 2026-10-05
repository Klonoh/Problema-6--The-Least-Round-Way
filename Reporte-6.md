# Reporte: The Least Round Way

**Problema:** Codeforces 2B — *The Least Round Way*  
**Técnica:** Programación Dinámica — Top-Down con memoización  
**Dificultad:** 2000  
**Tags:** `dp`, `math`

---

## 1. Descripción del problema

Se entrega una matriz cuadrada:

\[
A[0..n-1][0..n-1]
\]

formada por números enteros no negativos. Se debe encontrar un camino que comience en la esquina superior izquierda:

\[
(0,0)
\]

y termine en la esquina inferior derecha:

\[
(n-1,n-1)
\]

En cada paso solo están permitidos dos movimientos:

- **D:** bajar desde \((i,j)\) hacia \((i+1,j)\).
- **R:** avanzar a la derecha desde \((i,j)\) hacia \((i,j+1)\).

Por lo tanto, todo camino factible contiene exactamente \(n-1\) movimientos hacia abajo y \(n-1\) movimientos hacia la derecha.

Si \(P\) es un camino, se define su producto como:

\[
Producto(P)=\prod_{(i,j)\in P}A[i][j]
\]

El objetivo es encontrar un camino que **minimice la cantidad de ceros finales de este producto**.

Las restricciones originales son:

\[
2\le n\le1000
\]

\[
0\le A[i][j]\le10^9
\]

Además del mínimo número de ceros finales, se debe entregar un camino que alcance dicho mínimo.

### 1.1. ¿Qué determina la cantidad de ceros finales?

Un cero final se produce por cada factor \(10\) presente en un número, y:

\[
10=2\cdot5
\]

Por ejemplo:

\[
200=2^3\cdot5^2
\]

Como existen tres factores 2 y dos factores 5, pueden formarse solamente dos parejas \(2\cdot5\). Por lo tanto:

\[
200
\]

tiene dos ceros finales.

Si \(v_2(x)\) representa la cantidad de factores 2 de \(x\), y \(v_5(x)\) la cantidad de factores 5, entonces para un producto positivo:

\[
Z(x)=\min(v_2(x),v_5(x))
\]

Por ello no es necesario calcular directamente los productos de los caminos. Basta con conocer cuántos factores 2 y 5 acumula cada uno. Esta es también la observación utilizada por el tutorial oficial del problema.

Para cada celda definimos:

\[
c_2(i,j)=v_2(A[i][j])
\]

\[
c_5(i,j)=v_5(A[i][j])
\]

De esta manera el problema original se transforma en dos problemas de camino mínimo:

1. encontrar el camino que acumula la menor cantidad de factores 2;
2. encontrar el camino que acumula la menor cantidad de factores 5.

Finalmente se escoge el mejor de ambos.

### 1.2. Ejemplo: solución óptima y no óptima

Considérese:

```text
1    10    10
1     1    10
10    1     1
```

Un camino posible es:

```text
1 → 10 → 10
          ↓
          10
          ↓
           1
```

Corresponde a `RRDD`. Su producto es:

\[
1\cdot10\cdot10\cdot10\cdot1=1000
\]

por lo tanto tiene:

\[
3
\]

ceros finales.

Sin embargo, existe:

```text
1
↓
1 → 1
    ↓
    1 → 1
```

correspondiente a `DRDR`.

Su producto es:

\[
1\cdot1\cdot1\cdot1\cdot1=1
\]

por lo tanto tiene:

\[
0
\]

ceros finales.

Así, `RRDD` es una solución factible pero no óptima, mientras que `DRDR` es óptima.

---

## 2. Subestructura óptima

### 2.1. Decisión

Supongamos que actualmente nos encontramos en la celda:

\[
(i,j)
\]

La decisión que reduce el problema es:

> **¿Cuál será el siguiente movimiento del camino?**

Existen como máximo dos alternativas:

```text
                    (i,j)
                   /     \
                  /       \
              ABAJO      DERECHA
                ↓            →
            (i+1,j)       (i,j+1)
```

Es decir:

### Alternativa 1: bajar

Se elige:

\[
(i,j)\rightarrow(i+1,j)
\]

Después de esta decisión queda el subproblema:

> Encontrar el camino óptimo desde \((i+1,j)\) hasta \((n-1,n-1)\).

### Alternativa 2: ir a la derecha

Se elige:

\[
(i,j)\rightarrow(i,j+1)
\]

Después queda:

> Encontrar el camino óptimo desde \((i,j+1)\) hasta \((n-1,n-1)\).

En ambos casos queda **el mismo tipo de problema que el original**, pero más pequeño, ya que la distancia restante hasta el destino disminuye en una unidad.

Podemos medir este tamaño mediante:

\[
d(i,j)=(n-1-i)+(n-1-j)
\]

Después de cualquiera de las dos decisiones:

\[
d'=d-1
\]

### 2.2. Cómo se combinan los subproblemas

Fijemos por ahora un factor \(p\), donde:

\[
p\in\{2,5\}
\]

La celda actual aporta:

\[
c_p(i,j)
\]

factores \(p\).

Si se baja, el costo completo es:

\[
c_p(i,j)+D_p(i+1,j)
\]

Si se avanza a la derecha:

\[
c_p(i,j)+D_p(i,j+1)
\]

Como el objetivo es minimizar, se debe escoger:

\[
\min(D_p(i+1,j),D_p(i,j+1))
\]

Por lo tanto, la solución se construye como:

```text
Costo de la celda actual
            +
     mejor continuación
        /        \
     abajo      derecha
```

### 2.3. ¿Por qué la subestructura es realmente óptima?

No basta con decir que el problema puede dividirse. Debemos demostrar que los subproblemas utilizados también deben resolverse óptimamente.

Supongamos que una solución óptima desde \((i,j)\) comienza bajando hacia:

\[
(i+1,j)
\]

Entonces el resto de ese camino conecta \((i+1,j)\) con el destino.

Supongamos que ese camino restante **no fuese óptimo** para el subproblema \((i+1,j)\). Entonces existiría otro camino desde \((i+1,j)\) hasta el destino que acumularía menos factores.

Podríamos reemplazar únicamente esa parte de la solución:

```text
(i,j)
  |
  ↓
(i+1,j) ---- camino no óptimo
```

por:

```text
(i,j)
  |
  ↓
(i+1,j) ---- camino óptimo
```

manteniendo válida la primera decisión y obteniendo un costo total menor.

Esto contradice que el camino original fuese óptimo.

El mismo argumento se aplica si la primera decisión fue ir hacia la derecha.

Por lo tanto:

> **Una vez fijado el primer movimiento de una solución óptima, el camino restante debe ser una solución óptima del subproblema generado por esa decisión.**

Esta propiedad permite aplicar programación dinámica.

---

## 3. Relación de recurrencia

### 3.1. Estado

Para un factor fijo:

\[
p\in\{2,5\}
\]

definimos:

\[
D_p(i,j)
\]

como:

> **La mínima cantidad de factores \(p\) que puede acumular un camino desde la celda \((i,j)\) hasta \((n-1,n-1)\), incluyendo la celda actual.**

El estado queda definido únicamente por:

\[
(i,j)
\]

una vez fijado \(p\).

No necesitamos recordar el camino anterior ni la cantidad acumulada anteriormente, porque las decisiones futuras solo dependen de la posición actual.

### 3.2. Caso general

Desde una celda interior existen las dos alternativas identificadas anteriormente:

\[
\boxed{
D_p(i,j)
=
c_p(i,j)+
\min
\left(
D_p(i+1,j),
D_p(i,j+1)
\right)
}
\]

para:

\[
p\in\{2,5\}
\]

Cada término corresponde directamente a una alternativa:

\[
D_p(i+1,j)
\]

representa bajar, mientras que:

\[
D_p(i,j+1)
\]

representa avanzar a la derecha.

### 3.3. Casos base

Cuando se alcanza la esquina inferior derecha ya no existe ninguna decisión pendiente. Solo debe contabilizarse la celda actual:

\[
\boxed{
D_p(n-1,n-1)=c_p(n-1,n-1)
}
\]

También se define:

\[
D_p(i,j)=+\infty
\]

si:

\[
i\ge n
\]

o:

\[
j\ge n
\]

Esto evita que un movimiento que salga de la matriz pueda ser escogido por el mínimo.

### 3.4. Obtención de la respuesta

Sean:

\[
E_2(P)=\sum_{(i,j)\in P}c_2(i,j)
\]

y:

\[
E_5(P)=\sum_{(i,j)\in P}c_5(i,j)
\]

Para todo camino positivo:

\[
Z(P)=\min(E_2(P),E_5(P))
\]

Entonces:

\[
\min_P Z(P)
=
\min_P\min(E_2(P),E_5(P))
\]

y esto equivale a:

\[
\boxed{
\min
\left(
\min_P E_2(P),
\min_P E_5(P)
\right)
}
\]

Por lo tanto:

\[
\boxed{
Respuesta=
\min(D_2(0,0),D_5(0,0))
}
\]

para matrices sin considerar todavía el caso especial de los ceros.

### 3.5. Ejemplo de aplicación de la recurrencia

Considérese:

```text
2    10
5     4
```

Las cantidades de factores 2 son:

```text
1    1
0    2
```

Por lo tanto:

\[
D_2(1,1)=2
\]

\[
D_2(1,0)=0+2=2
\]

\[
D_2(0,1)=1+2=3
\]

y:

\[
D_2(0,0)=1+\min(2,3)=3
\]

Para los factores 5:

```text
0    1
1    0
```

Entonces:

\[
D_5(1,1)=0
\]

\[
D_5(1,0)=1+0=1
\]

\[
D_5(0,1)=1+0=1
\]

\[
D_5(0,0)=0+\min(1,1)=1
\]

Finalmente:

\[
\min(3,1)=1
\]

Por ejemplo, el camino `DR` produce:

\[
2\cdot5\cdot4=40
\]

que tiene exactamente un cero final.

La recurrencia llega correctamente a su caso base y reproduce el óptimo.

### 3.6. Caso especial: celdas con valor 0

No puede calcularse \(v_2(0)\) ni \(v_5(0)\) de la forma habitual. Además, un camino que pasa por una celda cero produce un producto igual a cero.

El tratamiento utilizado para este problema consiste en calcular primero el mejor camino que **no pasa por ceros**, asignando a las celdas cero un costo suficientemente grande.

También se guarda la posición de algún cero.

Si existe un cero y el mejor camino sin ceros tiene más de un cero final, se construye un camino que pase por esa celda y se obtiene una respuesta de valor `1`, según el tratamiento requerido por el problema. Si el mejor camino sin cero tiene costo `0` o `1`, ese camino ya es igual o mejor.

Este caso especial también aparece explícitamente en el tutorial oficial de Codeforces.

---

## 4. Algoritmo Top-Down con memoización

Para implementar directamente la recurrencia se utilizan dos tablas:

```text
memo[2][n][n]
memo[5][n][n]
```

conceptualmente, o equivalentemente dos matrices \(n\times n\).

Cada posición se inicializa con el valor centinela:

```text
-1
```

para representar un subproblema todavía no calculado.

Además se utiliza una tabla `dir` para recordar qué alternativa produjo el mínimo y posteriormente reconstruir el camino.

### Pseudocódigo

```text
memo[p][i][j] = -1 para todo i,j
dir[p][i][j]  = NONE

function D(p, i, j):

    if i >= n or j >= n:
        return INFINITO

    if memo[p][i][j] != -1:
        return memo[p][i][j]

    if i == n-1 and j == n-1:
        memo[p][i][j] = c_p(i,j)
        return memo[p][i][j]

    abajo   = D(p, i+1, j)
    derecha = D(p, i, j+1)

    if abajo <= derecha:
        dir[p][i][j] = 'D'
    else:
        dir[p][i][j] = 'R'

    memo[p][i][j] =
        c_p(i,j) + min(abajo, derecha)

    return memo[p][i][j]
```

El problema completo se resuelve mediante:

```text
respuesta2 = D(2, 0, 0)
respuesta5 = D(5, 0, 0)

mejor = min(respuesta2, respuesta5)

if existe_cero AND mejor > 1:
    respuesta = 1
    camino = construirCaminoPorCero()
else:
    if respuesta2 <= respuesta5:
        respuesta = respuesta2
        camino = reconstruir(dir[2])
    else:
        respuesta = respuesta5
        camino = reconstruir(dir[5])
```

La implementación es una traducción directa de:

\[
D_p(i,j)=c_p(i,j)+
\min(D_p(i+1,j),D_p(i,j+1))
\]

La memoización evita recalcular un mismo estado. Por ejemplo, la celda \((1,1)\) puede alcanzarse mediante `RD` o mediante `DR`, pero su solución se calcula una única vez y posteriormente se reutiliza.

---

## 5. Análisis del algoritmo

### 5.1. Número de subproblemas

El estado está determinado por:

\[
(i,j)
\]

Existen:

\[
n
\]

posibles valores de \(i\) y:

\[
n
\]

posibles valores de \(j\).

Por lo tanto:

\[
n\cdot n=n^2
\]

estados distintos.

Como se resuelve el problema para \(p=2\) y para \(p=5\):

\[
2n^2
\]

estados en total.

Asintóticamente:

\[
\boxed{O(n^2)}
\]

### 5.2. Tiempo por subproblema

En cada estado se evalúan como máximo dos alternativas:

\[
D_p(i+1,j)
\]

y:

\[
D_p(i,j+1)
\]

Fuera del costo de las llamadas recursivas, que se contabilizan en sus propios estados, se realizan solamente operaciones constantes:

- dos consultas;
- una comparación;
- un `min`;
- una suma;
- una asignación en la tabla.

Por lo tanto:

\[
\boxed{O(1)}
\]

por subproblema.

### 5.3. Complejidad total

Utilizando:

\[
T=
O(\#subproblemas\times trabajo\ por\ subproblema)
\]

se obtiene:

\[
T=O(n^2)\cdot O(1)
\]

por lo tanto:

\[
\boxed{T(n)=O(n^2)}
\]

El conteo inicial de factores 2 y 5 también requiere recorrer las \(n^2\) celdas. Como cada valor es como máximo \(10^9\), la cantidad de divisiones sucesivas por 2 o 5 está acotada por una constante respecto de \(n\), por lo que este preprocesamiento también es:

\[
O(n^2)
\]

La reconstrucción del camino utiliza exactamente:

\[
2n-2
\]

movimientos, es decir:

\[
O(n)
\]

y no modifica la cota total.

El consumo de memoria está dominado por las tablas:

\[
\boxed{O(n^2)}
\]

### 5.4. Comparación con recursión sin memoización

Sin memoización, un mismo estado puede calcularse muchas veces.

Por ejemplo:

```text
(0,0)
 /   \
D     R
|     |
(1,0) (0,1)
  \    /
   (1,1)
```

El estado \((1,1)\) aparece en ambas ramas.

A medida que crece la matriz, la cantidad de caminos posibles desde la esquina superior izquierda hasta la inferior derecha es:

\[
\binom{2n-2}{n-1}
\]

cantidad que crece exponencialmente.

Una implementación recursiva ingenua puede, por lo tanto, realizar un número exponencial de llamadas, aproximadamente acotado por:

\[
O(4^n)
\]

mientras que la memoización reduce el trabajo a:

\[
O(n^2)
\]

al calcular cada estado una sola vez.

---

## 6. Correctitud

Demostraremos primero que \(D_p(i,j)\) calcula correctamente el mínimo número de factores \(p\) para \(p\in\{2,5\}\).

### 6.1. Teorema

Para todo estado válido \((i,j)\):

\[
D_p(i,j)
\]

es el mínimo costo en factores \(p\) de cualquier camino válido desde \((i,j)\) hasta \((n-1,n-1)\).

### 6.2. Demostración por inducción

Utilizamos inducción sobre:

\[
d(i,j)=(n-1-i)+(n-1-j)
\]

que corresponde a la cantidad de movimientos restantes hasta el destino.

#### Caso base

Si:

\[
d(i,j)=0
\]

entonces:

\[
(i,j)=(n-1,n-1)
\]

No existe ninguna decisión pendiente y el único camino posible contiene únicamente esta celda.

Por definición:

\[
D_p(n-1,n-1)=c_p(n-1,n-1)
\]

que es exactamente su costo óptimo.

Por lo tanto, el caso base es correcto.

#### Hipótesis inductiva

Supongamos que la función calcula correctamente el costo óptimo para todos los estados cuya distancia al destino sea menor que \(d\).

#### Paso inductivo

Consideremos un estado \((i,j)\) con distancia \(d>0\).

Todo camino factible desde esta celda debe comenzar necesariamente con una de las siguientes alternativas:

1. bajar hacia \((i+1,j)\), si la posición existe;
2. avanzar hacia \((i,j+1)\), si la posición existe.

No existe una tercera posibilidad.

Si se baja, por hipótesis inductiva:

\[
D_p(i+1,j)
\]

es el costo mínimo posible para todo el camino restante.

Por ello el mejor camino que comienza bajando cuesta:

\[
c_p(i,j)+D_p(i+1,j)
\]

Análogamente, el mejor camino que comienza a la derecha cuesta:

\[
c_p(i,j)+D_p(i,j+1)
\]

Ambas alternativas generan soluciones factibles y, además, cubren todos los caminos posibles.

Por lo tanto, el mejor camino es exactamente:

\[
c_p(i,j)+
\min(D_p(i+1,j),D_p(i,j+1))
\]

que corresponde a la recurrencia utilizada.

Así, \(D_p(i,j)\) es óptimo.

Por inducción, el resultado es correcto para todos los estados, incluyendo:

\[
D_p(0,0)
\]

### 6.3. Correctitud de la combinación de factores 2 y 5

Para cualquier camino positivo \(P\):

\[
Z(P)=\min(E_2(P),E_5(P))
\]

Sean:

\[
m_2=\min_P E_2(P)
\]

y:

\[
m_5=\min_P E_5(P)
\]

Para todo camino \(P\):

\[
E_2(P)\ge m_2
\]

y:

\[
E_5(P)\ge m_5
\]

por lo que:

\[
Z(P)\ge\min(m_2,m_5)
\]

Además, existe un camino que alcanza \(m_2\) y otro que alcanza \(m_5\). Por lo tanto existe al menos uno cuyo número de ceros finales es a lo más:

\[
\min(m_2,m_5)
\]

De ambas desigualdades:

\[
\boxed{
\min_P Z(P)=\min(m_2,m_5)
}
\]

Entonces basta calcular:

\[
D_2(0,0)
\]

y:

\[
D_5(0,0)
\]

y escoger el menor.

Esta propiedad también es demostrada en el tutorial oficial de Codeforces.

---

## 7. Experimentos

Los experimentos se realizaron con una implementación en Python del algoritmo Top-Down descrito anteriormente.

Para las mediciones de rendimiento se generaron matrices de números positivos aleatorios entre \(1\) y \(10^9\), utilizando una semilla fija para hacer el experimento reproducible.

Cada tamaño se ejecutó cinco veces y se utilizó la **mediana** de los tiempos obtenidos para disminuir el efecto de variaciones ocasionales del sistema.

### 7.1. Pruebas de correctitud

#### Caso 1 — Ejemplo oficial

**Entrada:**

```text
3
1 2 3
4 5 6
7 8 9
```

**Salida obtenida:**

```text
0
DDRR
```

El camino recorre:

\[
1\rightarrow4\rightarrow7\rightarrow8\rightarrow9
\]

Su producto es:

\[
1\cdot4\cdot7\cdot8\cdot9=2016
\]

Como 2016 no termina en cero, el resultado es:

\[
0
\]

No puede existir una solución mejor, ya que el número de ceros finales nunca puede ser negativo.

Por lo tanto la salida es óptima.

---

#### Caso 2 — Todos los elementos son múltiplos de 10

**Entrada:**

```text
2
10 10
10 10
```

**Salida obtenida:**

```text
3
DR
```

Cualquier camino de una matriz \(2\times2\) visita tres celdas.

Por lo tanto:

\[
10\cdot10\cdot10=1000
\]

que termina en tres ceros.

Ambos caminos posibles, `DR` y `RD`, tienen el mismo costo.

Por ello:

\[
3
\]

es la respuesta correcta.

---

#### Caso 3 — Presencia de un cero útil

**Entrada:**

```text
3
1 10 1
1  0 1
10 10 10
```

**Salida obtenida:**

```text
1
DRDR
```

El camino `DRDR` pasa por la posición central, cuyo valor es `0`.

Los caminos que evitan esa celda y atraviesan la parte inferior acumulan múltiples factores 2 y 5. En este caso el tratamiento especial del cero produce una solución de valor `1`, que es mejor que cualquier camino positivo con más de un cero final.

El algoritmo detecta correctamente este caso y construye un camino que atraviesa la celda cero.

---

#### Caso 4 — Matriz sin factores 2 ni 5

**Entrada:**

```text
2
1 1
1 1
```

**Salida obtenida:**

```text
0
DR
```

Todo camino tiene producto:

\[
1
\]

y:

\[
v_2(1)=v_5(1)=0
\]

Por lo tanto el producto no contiene ningún cero final.

La respuesta óptima es:

\[
0
\]

---

### 7.2. Experimento de tiempo de ejecución

Los resultados obtenidos fueron:

| \(n\) | Tiempo mediano |
|---:|---:|
| 50 | 0.002954 s |
| 100 | 0.010544 s |
| 150 | 0.026927 s |
| 200 | 0.046397 s |
| 300 | 0.107860 s |
| 400 | 0.196303 s |
| 500 | 0.325953 s |

![Tiempo de ejecución de The Least Round Way](least_round_way_timing.png)

### 7.3. Análisis del gráfico

El tiempo aumenta de manera no lineal al crecer \(n\), pero el comportamiento es consistente con una función cuadrática.

Por ejemplo:

- al pasar aproximadamente de \(n=100\) a \(n=200\), el número de estados se multiplica por cuatro;
- al pasar de \(n=200\) a \(n=400\), ocurre nuevamente lo mismo.

Los tiempos experimentales presentan la misma tendencia general.

También puede analizarse:

\[
\frac{T(n)}{n^2}
\]

Para los tamaños experimentados, este cociente permanece aproximadamente constante, alrededor de \(1.1\)–\(1.3\) microsegundos por celda cuadrática en el entorno utilizado.

Esto concuerda con el análisis teórico:

\[
T(n)=O(n^2)
\]

Las pequeñas diferencias respecto de una curva cuadrática perfecta se deben a factores propios de una medición real, como administración de memoria, caché, recolector de basura e interpretación de Python.

Por lo tanto, los experimentos entregan evidencia empírica consistente con la cota teórica obtenida.

### 7.4. Código y reproducibilidad

Se adjunta un notebook con:

- implementación completa del algoritmo;
- casos de prueba;
- medición de tiempos;
- generación del gráfico experimental.

**Link a Colab/GitHub:**  
`[REEMPLAZAR POR EL ENLACE PÚBLICO AL SUBIR EL NOTEBOOK ADJUNTO A GOOGLE COLAB O GITHUB]`

---

## 8. Conclusión

*The Least Round Way* puede resolverse mediante programación dinámica gracias a que cada solución se construye tomando una decisión local entre bajar o avanzar a la derecha, dejando en ambos casos un subproblema del mismo tipo y estrictamente más pequeño.

La cantidad de ceros finales de un producto puede expresarse mediante sus factores 2 y 5, permitiendo transformar el problema en dos problemas independientes de camino mínimo.

Para un factor \(p\in\{2,5\}\), la recurrencia es:

\[
\boxed{
D_p(i,j)=
c_p(i,j)+
\min(D_p(i+1,j),D_p(i,j+1))
}
\]

La memoización garantiza que cada uno de los \(O(n^2)\) estados sea resuelto una sola vez y, dado que cada estado evalúa únicamente dos alternativas, se obtiene una complejidad temporal:

\[
\boxed{O(n^2)}
\]

y espacial:

\[
\boxed{O(n^2)}
\]

La demostración por inducción prueba que la recurrencia considera todas las soluciones factibles y selecciona la mejor, mientras que los experimentos realizados muestran un crecimiento temporal coherente con la complejidad cuadrática esperada.

---

## Referencias

- Codeforces, **Problem 2B — The Least Round Way**.
- Codeforces Beta Round #2, **Another Tutorial**, sección correspondiente al problema B.