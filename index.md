---
layout: default
title: Página Principal
format:
  html:
    code-fold: true
jupyter: true
---

# Regresión Cuantíl

**Autores:** Alfredo Bistrain, Jesús García y Rodolfo Lucario  
**Institución:** CIMAT  
**Fecha:** 2025-05-15

---

# Motivación

---

# Orígenes del concepto

## Origen estadístico del cuantil

- Los cuantiles se remontan al siglo XIX (Francis Galton): introducción de los percentiles.
- La mediana (cuantil 0.5) se propuso como alternativa robusta a la media.
- El cuantil se consolidó como medida de posición resistente a outliers.

---

## Regresión cuantíl: nacimiento formal (1978)

- Propuesta formal por Roger Koenker y Gilbert Bassett:

  **Artículo fundacional:** *Regression Quantiles*, Econometrica, 1978.

- Generalización de la regresión de la mediana a cualquier cuantil  $\tau \in (0,1).$
- Utiliza una función de pérdida asimétrica y se resuelve con **programación lineal**.

---

# Difusión y aplicaciones

## Difusión en estadística y econometría

- Ampliamente adoptada en:
  - Economía laboral (retornos a la educación, brechas salariales).
  - Finanzas (riesgo en colas de distribución).
  - Medicina (efectos heterogéneos de tratamientos).
- Paquete `quantreg` desarrollado por Koenker en R.
- Libro de referencia:
  - *Quantile Regression*, Koenker (2005), Cambridge University Press.

## Limitación del modelo lineal tradicional

- La regresión lineal ordinaria estima:  
  $$  
    \mathbb{E}[Y \mid X = x]  
  $$
- Describe sólo el **promedio condicional**.
- Supone que el efecto de las covariables es constante a lo largo de la distribución.
- Puede ocultar:
  - Heterocedasticidad  
  - Asimetrías  
  - Efectos diferenciados en colas  

> **Nota:** Bajo el modelo clásico  
> $$  
>   Y_i = X_i^\top \beta + \varepsilon_i,  
>   \quad  
>   \mathbb{E}[\varepsilon_i \mid X_i] = 0  
> $$  
> se tiene  
> $$  
>   \mathbb{E}[Y_i \mid X_i] = X_i^\top \beta.  
> $$

## Ventaja de la regresión cuantíl

- La regresión cuantíl estima:  
  $$  
    Q_Y(\tau \mid X = x) = \inf\{y : \mathbb{P}(Y \le y \mid X = x) \ge \tau\}  
  $$
- Permite obtener la forma completa de la distribución condicional de $$Y$$.  
- Al variar $\tau \in (0,1)$ se construyen curvas cuantílicas. Por ejemplo:
  - Mediana condicional: $\tau = 0.5$
  - Colas:  
    $\tau = 0.1, \; 0.9$
- Modela:
  - Cambios en la variabilidad  
  - Comportamiento desigual en subgrupos  
  - Riesgo extremo y desigualdad  

## Comparación visual

![cuantile_vs_ols](cuantile_vs_ols.png)

---

# Fundamentos de la regresión cuantíl

## Definición de cuantiles

- Cuantil de orden \(\tau\) de la v.a. \(Y\):  
  $$  
    Q_Y(\tau) = \inf\{y : F_Y(y) \ge \tau\}  
  $$
- Para \(tau = 0.5\), se obtiene la mediana.  
- Se pueden estimar otros percentiles: 10%, 25%, 90%, etc.

## El cuantil como un problema de minimización

En este apartado veremos cómo los cuantiles pueden ser expresados como un problema de minimización. Es bien sabido que la media de \(Y\) es la solución al problema de minimización siguiente:

$$
\arg\min_c E[(Y - c)^2]
$$
Demostraremos más adelante que si reemplazamos la función de pérdida cuadrática por la función de pérdida absoluta, se tiene que la mediana, que denotaremos por $Q_{Y}\left(\frac{1}{2}\right)$ es solución de:

$$
\arg\min_c E[|Y - c|]
$$

Para generalizar el hecho anterior para cualquier cuantil, definimosla función de pérdida:
$$
\rho_{\theta}(y) = [\theta - I(y < 0)] \cdot |y| = y(1 - \theta) I(y \leq 0) + y \theta I(y > 0)
$$
Así, como demostraremos más adelnate, el cuantíl $Q_{Y}(\theta)$ es solución de la siguiente función de minimización 

$$
\arg\min_c E[\rho_{\theta}(Y - c)]
$$
Ahora demostraremos que la mediana minimiza la esperanza de la función de pérdida del valor absoluto, esto para el caso en el que $Y$ es una v.a. con densidad $f(y)$. Así, nótese que,



$$
\begin{align}
  \mathbb{E}[|Y-c|] &= \int_{y \in \mathbb{R}}|y-c|f(y)dy\\
  &= \int_{y<c}(c-y)f(y)dy + \int_{y>c}(y-c)f(y)dy
\end{align}
$$
Analizando la integral de la izquierda de la última expresión se tiene que:

$$
\int_{y<c}(c-y)f(y)dy = c \int_{-\infty}^{c} f(y)dy - \int_{-\infty}^{c} y f(y)dy
$$
Dado que el valor absoluto es una función convexa entonces derivando respecto a $c$ e igualando a 0, dbeemos ser capaces de encontrar el punto en el que se encuentra el mínimo. Así, 

$$
\begin{align*}
        \frac{\partial}{\partial c}\left[\int_{y<c}(c-y)f(y)dy\right] &= cf(c)+F(c)-cf(c)\\
        &= F(c)
    \end{align*}
$$

De forma similar:

$$
\begin{equation*}
        \frac{\partial}{\partial c}\left[\int_{y<c}(c-y)f(y)dy\right] = F(c) -1
    \end{equation*}
$$

Así, juntando los dos resultados anteriores, se tiene que 


$$
\frac{\partial}{\partial c}\mathbb{E}[|Y-c|] = 2F(c)-1
$$

por lo que 

$$
\frac{\partial }{\partial c}\mathbb{E}[|Y-c|] = 0 \Longleftrightarrow F(c) = \frac{1}{2} \Rightarrow c = Q_{Y}(0.5)
$$

Para el caso general, dónde la función de pérdida es $\rho_{\theta}$ se tiene que :

$$
\mathbb{E}[\rho_{\theta}(Y-c)] = (1-	\theta)\int_{y<c}(c-y)f(y)dy + 	\theta\int_{y>c}(y-c)f(y)dy
$$

y podemos ver que es prácticamente la misma expresión a la que llegamos en el caso de la mediana excepto que estas integrales están multiplicadas por unas constantes. Así, bajo el mismo procedimiento enterior, se tiene que 
$$
\frac{\partial }{\partial c}\mathbb{E}[\rho_{\theta}(Y-c)] = (1-	\theta)F(c) + \theta(F(c)-1) = F(c) - \theta
$$

por lo tanto, dado que $\rho_{\theta}$ es una función convexa entonces alcanza su mínimo en dódne la primera derivada se hace 0 y por lo tanto, 

$$
\frac{\partial }{\partial c}\mathbb{E}[\rho_{	\theta}(Y-c)] = 0 \Longleftrightarrow F(c) = \theta \Rightarrow Q_{Y}(\theta) = c
$$
Si ahora en lugar de usar la función de densidad de $Y$ usamos la función de distribución  empírica de $Y$ que esta dada por:

$$
F_n(x) = \frac{1}{n} \sum_{k=1}^{n} 1_{\{Y_k \leq x\}}
$$
dónde $\{Y_{i}\}_{i\geq 1}$ es una sucesión de v.a. tal que las v.a. $Y_{i}$ se distribuye igual que $Y$ para todo $i\geq 1$ entonces el problema de minimización se convierte en 

$$
\min_{c} \frac{1}{n} \sum_{k=1}^{n} \rho_{\theta}(y_k - c)
$$

aquí, $y_i$ son las observaciones de $Y_i$. El valor de $c$ que minimiza esta función es el cuantil $\theta$ muestral (ya que las observaciones de las v.a. $Y_{i}$ ya fueron observadas).

## Supuesto de linealidad 

Podemos extender la idea del cuantil como un problema de minimización a la regresión cuantílica. Dado que el cuantil $\tau$-ésimo ($ Q_{Y}(\tau) $ ) minimiza la esperanza de la función de pérdida antes mencionada, el cuantil $\tau$-ésimo condicional a $X$, que denotaremos por $Q_{Y}(\tau|X)$,  minimiza la esperanza condicional respecto a $X$ de la función de pérdida antes mencionada.\
Aún más, si asumimos 
$$Q_{Y}(\tau|x) = x^{T}\beta(\tau)$$
es posible estimar a $\beta(\tau )$ mediante el problema de minimización 
$$\min_{\beta}\sum_{k=1}^{n}\rho_{\theta}(y_{k}-x_{k}^{T}\beta(\tau))$$


##  El problema general de programación lineal

La programación lineal es una herramienta sumamente flexible, ampliamente aplicada en distintos campos y con diversos fines, tales como asignar recursos, programar trabajadores y planes, planificar inversiones en carteras, formular estrategias militares y definir estrategias de mercadeo. Un tratamiento exhaustivo del tema está fuera de este texto. Sin embargo, se presentan las ideas básicas útiles para comprender el procedimiento de resolución de la regresión cuantílica mediante programación lineal.

La programación lineal es una rama de la matemática que se ocupa de la asignación eficiente de recursos limitados a actividades conocidas con el objetivo de alcanzar una meta deseada, como minimizar costos o maximizar beneficios.

Las variables:
$$
x_i \geq 0 \quad i=1, \ldots, n,
$$
cuyos valores deben decidirse de manera óptima, se denominan variables de decisión.

En un programa lineal general, el objetivo es encontrar un vector $\mathbf{x}^* \in \mathbb{R}_{+}^n$ que minimice o maximice, el valor de una función lineal llamada función objetivo, dada entre todos los vectores $\mathbf{x} \in \mathbb{R}_{+}^n$ que satisfacen un sistema dado de ecuaciones e inecuaciones lineales llamado sistema de restricciones. El papel de la linealidad es, por tanto, doble:
\begin{itemize}
  \item la función objetivo, es decir, la calidad del plan, se mide mediante una función lineal de las cantidades consideradas;
  \item los planes factibles están restringidos por restricciones lineales (inequaciones).
\end{itemize}
La linealidad de algunos modelos puede justificarse sobre la base de las propiedades típicas del problema. Sin embargo, algunos problemas no lineales pueden ser linealizados mediante un uso adecuado de transformaciones matemáticas. Este es el caso del problema de regresión cuantílica (QR, por sus siglas en inglés). La representación (o a veces la aproximación) de un problema mediante una formulación de programación lineal garantiza que existan procedimientos eficientes para calcular soluciones.

Para recapitular, un problema típico de programación lineal satisface las siguientes condiciones:

- Las $n$ variables de decisión son no negativas:
$$
x_i \geq 0 \quad i=1, \ldots, n .
$$

Geométricamente, esto restringe las soluciones a $\mathbb{R}_{+}^n$.
En caso de que el problema esté caracterizado por variables sin restricción de signo, es decir, variables que pueden ser positivas, negativas o cero, se puede utilizar un truco simple para restringirlas a variables no negativas, introduciendo dos variables no negativas $[x]^{+}$ y $[-x]^{+}$ y convirtiendo cualquier variable de decisión no restringida como la diferencia entre dos variables no negativas sin cambiar el problema de optimización:
$$
\begin{aligned}
& x=[x]^{+}-[-x]^{+} \\
& {[x]^{+} \geq 0} \\
& {[-x]^{+} \geq 0 .}
\end{aligned}
$$

Para $x>0$, se fija $[x]^{+}=x$ y $[-x]^{+}=0$, mientras que para $x<0$ se fija $[-x]^{+}=-x$ y $[x]^{+}=0$.
Si $x=0$, entonces $[x]^{+}=[-x]^{+}=0$.

-  La función objetivo, es una función lineal de las mismas variables:
$$
z=\sum_{i=1}^n c_i x_i=\mathbf{c x} .
$$

La conversión de un problema de minimización a uno de maximización es trivial: maximizar $z$ es equivalente a minimizar $-z$.

- Las $m$ restricciones que regulan el proceso pueden expresarse como ecuaciones lineales o desigualdades lineales escritas en términos de las variables de decisión. Una restricción genérica consiste en una igualdad o desigualdad asociada a combinaciones lineales de las variables de decisión:
$$
a_1 x_1+\cdots+a_i x_i+\cdots+a_n x_n\left\{\begin{array}{l}
\leq \\
= \\
\geq
\end{array}\right\} b .
$$

Se pueden pasar las restricciones de una forma a otra. Por ejemplo, una restricción de desigualdad:
$$
a_1 x_1+\cdots+a_i x_i+\cdots+a_n x_n \leq b
$$
puede convertirse en una restricción de mayor o igual simplemente multiplicándola por $-1$. En cambio, puede convertirse en una restricción de igualdad añadiendo una variable no negativa (variable de holgura),
$$
a_1 x_1+\cdots+a_i x_i+\cdots+a_n x_n+w=b, \quad w \geq 0 .
$$
Por otro lado, una restricción de igualdad,
$$
a_1 x_1+\cdots+a_i x_i+\cdots+a_n x_n=b
$$
puede convertirse en una forma de desigualdad mediante la introducción de dos restricciones de desigualdad:
$$
\begin{aligned}
& a_1 x_1+\cdots+a_i x_i+\cdots+a_n x_n \leq b \\
& a_1 x_1+\cdots+a_i x_i+\cdots+a_n x_n \geq b .
\end{aligned}
$$

Esto significa que no hay diferencias en la forma en que se plantean las restricciones.
Finalmente, vale la pena recordar que, desde un punto de vista geométrico, una ecuación lineal corresponde a un hiperplano, mientras que una desigualdad divide el espacio $n$-dimensional en dos, una parte en la que se satisface la desigualdad y otra en la que no.

Al combinar todas las desigualdades, el conjunto de vectores que satisfacen las restricciones, conjunto factible, es entonces la intersección de todos los semiespacios involucrados, los $n$ semiespacios impuestos por la no negatividad de las variables de decisión y los $m$ semiespacios impuestos por las $m$ restricciones que regulan el proceso.

Consideremos un problema de programación lineal con $n$ incógnitas (variables de decisión) y $m$ restricciones. La formulación matricial del problema es la siguiente (forma estándar),
```{=tex}
\begin{equation}\label{eq21}
\begin{array}{ll}
\operatorname{minimize} & \mathbf{c x} \\
\text { subject to } & \mathbf{A x} \leq \mathbf{b} \\
& \mathbf{x} \geq \mathbf{0} .
\end{array}
\end{equation}
```
La forma estándar plantea las desigualdades como de tipo menor o igual y requiere la no negatividad de las variables de decisión.  
    - El vector $\mathbf{c}_{[n]}$ contiene los costos asociados a las variables de decisión.  
    - La matriz $\mathbf{A}_{[m \times n]}$ y el vector $\mathbf{b}_{[m]}$ permiten considerar las $m$ restricciones.  
    -  Una solución $\mathbf{x}_{[n]}$ se denomina factible si satisface todas las restricciones.  
    - Se denomina óptima, y se denota por $\mathbf{x}^*$, si alcanza el mínimo y es factible.  


Ya se ha mencionado cómo la condición $\mathbf{x} \geq \mathbf{0}$ restringe a $\mathbf{x}$ a $\mathbb{R}_{+}^n$, es decir, al cuadrante positivo en el espacio $n$-dimensional. En $\mathbb{R}^2$ es un cuarto del plano, en $\mathbb{R}^3$ es un octavo del espacio, y así sucesivamente. Las restricciones $\mathbf{A x} \leq \mathbf{b}$ generan $m$ semiespacios adicionales. El conjunto factible consiste en la intersección de los $m+n$ semiespacios mencionados. Dicho conjunto puede ser acotado, no acotado o vacío. La función de costo $\mathbf{c x}$ genera una familia de hiperplanos paralelos, es decir, el hiperplano $\mathbf{c x} = \text{constante}$ corresponde al conjunto de puntos cuyo costo es igual a dicha constante; cuando la constante varía, el hiperplano recorre todo el espacio $n$-dimensional. El óptimo $\mathbf{x}^*$ es el vector $n$-dimensional, que garantiza el menor costo dentro del conjunto factible.
Un problema que no tiene solución factible se llama no factible. Un problema con valores arbitrariamente grandes de la función objetivo se llama no acotado.

 Asociado con cada programación lineal está su programa dual.
Partiendo del problema lineal introducido anteriormente, también llamado el problema primal, su formulación dual comienza con la misma $\mathbf{A}$ y $\mathbf{b}$ pero invierte todo lo demás. El vector de costo primal $\mathbf{c}$ y el vector de restricciones $\mathbf{b}$ se intercambian con el vector de ganancia dual $\mathbf{b}$ y $\mathbf{c}$ se usa como el vector de restricciones. El desconocido dual (variable de decisión) $\mathbf{y}$ es ahora un vector con $m$ componentes, siendo las $n$ restricciones representadas por $\mathbf{A^{\top}y} \geq \mathbf{c}$. Dado el problema de programación lineal primal, el programa lineal dual asociado es,
$$
\begin{array}{ll}
\text{maximizar} & \mathbf{b^{\top} y} \\
\text{sujeto a} & \mathbf{A^{\top}y} \geq \mathbf{c} \\
& \mathbf{y} \geq \mathbf{0} .
\end{array}
$$

Los programas lineales vienen en pares primal/dual. Resulta que cada solución factible para uno de estos dos programas lineales da un límite sobre el valor objetivo óptimo para el otro.   
- El problema dual proporciona límites superiores para el problema primal (teorema de dualidad débil):
$$
\mathbf{c x} \leq \mathbf{b y} ;
$$
- Si el problema primal tiene una solución óptima, entonces el dual también tiene una solución óptima (teorema de dualidad fuerte),
$$
c^* = \mathbf{b y}^* .
$$
Cabe mencionar que a veces es más fácil resolver el programa lineal desde la formulación dual, en lugar de atacar la formulación primal.


### La formulación de programación lineal para el problema QR
Para poder abordar el problema de la regresión cuantilica como un problema de programación lineal, se necesita llevar la función objetivo y el conjunto de restricciónes a una forma lineal. Para esto observe que el problema general se puede plantear como,
$$
\begin{aligned}
\min _{\boldsymbol{\beta}(\theta)}& \sum_{i=1}^n \rho_\theta\left(y_i-\mathbf{x}_i^{\top} \boldsymbol{\beta}(\theta)\right) \\
s.a&\quad \boldsymbol{y}=X\boldsymbol{\beta}(\theta)+\boldsymbol{\varepsilon}.
\end{aligned}
$$
Para llevar este problema a uno que se pueda resolver en programación lineal, observe que la función objetivo está dada por, 
$$\rho_\theta\left(y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)\right)= \begin{cases}\theta\left|y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)\right| & \text { si } y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)>0 \\ (1-\theta)\left|y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)\right| & \text { si } y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta) \leq 0.\end{cases}$$
Considere las siguientes funciones funciones, 
$$[y]_{+}=\left\{\begin{array}{ll}y & \text { si } y>0 \\ 0 & \text { o.c. }\end{array}, \quad[y]_{-}\left\{\begin{array}{lll}-y & \text { si } & y<0 \\ 0 & o . c\end{array}\right.\right.$$
Note que si se utilizan estás funciones, la función objetivo se puede reescribir de la siguiente manera, 
$$\rho_\theta\left(y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)\right)=\theta\left[y_i-\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)\right]_{+}+(1-\theta)\left[\boldsymbol{x}_i^{\top} \boldsymbol{\beta}(\theta)-y_i\right]_{+}.$$
Por otro lado, defina al vector de errores positivos $\mathbf{s}_1$ y al vector de errores negativos $\mathbf{s}_2,$
$$
\mathbf{s}_1=[\mathbf{y}-\mathbf{X} \boldsymbol{\beta}(\theta)]_{+}, \quad \mathbf{s}_2=[\mathbf{X} \boldsymbol{\beta}(\theta)-\mathbf{y}]_{+}.
$$
Note que $\mathbf{s}_1$ y $\mathbf{s}_2$ son dos vectores de dimensión n, donde $\mathbf{s}_1$ en sus entradas solo tiene los errores positivos y donde los errores son negativos tiene cero, mientras que  $\mathbf{s}_2$ en sus entradas tiene el valor absoluto de los errores negativos y donde son negativos tienen cero. Observe que el problema original se puede reescribir de la siguiente manera, 
$$\begin{aligned} & \min _{\beta(\theta)} \sum_{i=1}^n \theta s_{1 i}+(1-\theta) s_{2 i} \\  \text { s.a } \boldsymbol{y} &=X \boldsymbol{\beta}(\theta)+\boldsymbol{s}_1-\boldsymbol{s}_2 \\ &=X [\boldsymbol{\beta}(\theta)]_{+}-X [\boldsymbol{\beta}(\theta)]_{-}+I \boldsymbol{s}_1-I \boldsymbol{s}_2.\end{aligned}$$
Si se define a,
$$
\boldsymbol{\psi} = \begin{pmatrix}
[\boldsymbol{\beta}(\theta)]_{+} \\ 
[\boldsymbol{\beta}(\theta)]_{-} \\ 
\mathbf{s}_1 \\ 
\mathbf{s}_2 
\end{pmatrix}, \quad 
\mathbf{d} = \begin{pmatrix}
\mathbf{0}_p \\ 
\mathbf{0}_p \\ 
\theta \mathbf{1}_n \\ 
(1-\theta) \mathbf{1}_n
\end{pmatrix}, \quad 
\mathbf{B} = \bigl[\,\mathbf{X},  -\mathbf{X},  \mathbf{I} , -\mathbf{I}\,\bigr],
$$
 el problema primal se puede reescribir como, 
$$
\begin{aligned}
\min & _\psi\quad \mathbf{d}^{\top} \psi \\  \text { sujeto a }&  \quad\mathbf{B} \psi=\mathbf{y}.
\end{aligned}
$$
De la expresión primal anterior se tiene que el problema dual está dado por, 
$$\begin{array}{ll}\max_{\boldsymbol{z}} & \boldsymbol{y}^{\top} \boldsymbol{z} \\ \text { s.a } & B^{\top} \boldsymbol{z} \leq 0 .\end{array}$$
  Observe que si se analiza el producto $B^{\top} \boldsymbol{z},$
$$B^{\top} \boldsymbol{z}=\left[\begin{array}{c}X^{\top} \boldsymbol{z} \\ -X^{\top} \boldsymbol{z} \\ I \boldsymbol{z} \\ -I \boldsymbol{z}\end{array}\right] \leq\left[\begin{array}{c}\boldsymbol{0}_p \\ \boldsymbol{0}_p \\ \theta \boldsymbol{1}_n \\ (1-\theta) \boldsymbol{1}_n\end{array}\right] \Rightarrow \begin{aligned} & X^{\top} \boldsymbol{z}=0 \\ & \boldsymbol{z} \in[\theta-1, \theta]^n.\end{aligned}$$
Esto debido a que en las dos primeras restricciones se tiene,
$$
\begin{aligned}
X^{\top} \boldsymbol{z} &\leq \boldsymbol{0}_p \\
X^{\top} \boldsymbol{z} &\geq \boldsymbol{0}_p 
\end{aligned}
$$
lo que equivale a la restricción de igualdad. Mientras que las dos ultimas restricciones,

$$
\begin{aligned}
I \boldsymbol{z} &\leq \theta \boldsymbol{1}_n  \\
I \boldsymbol{z} &\geq (\theta-1) \boldsymbol{1}_n, 
\end{aligned}
$$
lo que nos indica que cada entrada del vector $\boldsymbol{z}$ debe ser menor o igual a $\theta$ y que cada entrada del vector $\boldsymbol{z}$ debe ser mayor o igual a 
$(\theta-1).$  Sea $\boldsymbol{z}_0 \in [0,1]^n,$ observe que, 
$$\boldsymbol{z}_0 \in[0,1]^n \Rightarrow \boldsymbol{z}=\boldsymbol{z}_0+(\theta-1) \boldsymbol{1}_n.$$
    La función objetivo toma la forma,
    $$\max _{\boldsymbol{z}}\quad \boldsymbol{y}^{\top} \boldsymbol{z}=\max _{\boldsymbol{z}_0} y^{\top}\left(\boldsymbol{z}_0+(\theta-1) \boldsymbol{1}_{n}\right)=\max_{\boldsymbol{z}_0} \boldsymbol{y}^{\top} \boldsymbol{z}_0.$$
Las restricciones se reescriben como, 
$$0=X^{\top} \boldsymbol{z}=X^{\top}\left(\boldsymbol{z}_0+(\theta-1) \boldsymbol{1}_n\right) \Leftrightarrow X^{\top} \boldsymbol{z}_0=(1-\theta) X^{\top} \boldsymbol{1}_n.$$
Por lo tanto se tiene la siguiente formulación dual para el problema de regresión cuantílica,
$$
\max _{\mathbf{z}}\left\{\mathbf{y}^{\top} \mathbf{z} \mid \mathbf{X}^{\top} \mathbf{z}=(1-\theta) \mathbf{X}^{\top} \mathbf{1}, \mathbf{z} \in[0,1]^n\right\}.
$$



## La condición de subgradiente
A continuación se introducirá la condición básica de optimalidad que caracteriza los cuantiles de regresión. Se ha visto que podemos restringir nuestra atención a soluciones candidatas de la forma básica,

$$
b(h)=X(h)^{-1} y(h)
$$

Para algunos $h$, $X(h)$ puede ser singular. Esto no debe preocuparnos, podemos restringirnos a $b(h)$ con $h \in \mathcal{H}^{*}=\{h \in \mathcal{H}:|X(h)| \neq 0\}$. También hemos visto que nuestra condición de optimalidad implica verificar que las derivadas direccionales sean no negativas en todas direcciones. Para verificar esto en $b(h)$, debemos considerar,

$$
\nabla R(b(h), w)=-\sum_{i=1}^{n} \psi_{\tau}^{*}\left(y_{i}-x_{i}^{\prime} b(h),-x_{i}^{\prime} w\right) x_{i}^{\prime} w,
$$
con $$
\psi^{*}(u, v)= \begin{cases}\tau-I(u<0) & \text{ si } u \neq 0 \\ \tau-I(v<0) & \text{ si } u=0\end{cases}.
$$
Reparametrizando las direcciones para que $v=X(h) w$, tenemos optimalidad si y solo si,

$$
0 \leq-\sum_{i=1}^{n} \psi_{\tau}^{*}\left(y_{i}-x_{i}^{\prime} b(h),-x_{i} X(h)^{-1} v\right) x_{i}^{\prime} X(h)^{-1} v
$$

para todo $v \in \mathbb{R}^{p}$. Notemos que para $i \in h$, tenemos $e_{i}^{\prime}=x_{i}^{\prime} X(h)^{-1}$, el $i$-ésimo vector unitario base de $\mathbb{R}^{p}$, por lo que podemos reescribir esto como:

$$
0 \leq-\sum_{i \in h} \psi_{\tau}^{*}\left(0, v_{i}\right) v_{i}-\xi^{\prime} v=-\sum_{i \in h}\left(\tau-I\left(v_{i}<0\right)\right) v_{i}-\xi^{\prime} v
$$

donde,

$$
\xi\left(v_{i}\right)=\sum_{i \in \bar{h}} \psi_{\tau}^{*}\left(y_{i}-x_{i} b(h),-x_{i} X(h)^{-1} v_{i}\right) x_{i}^{\prime} X(h)^{-1}
$$

Finalmente, notemos que el espacio de "direcciones" $v \in \mathbb{R}^{p}$ está generado por $v= \pm e_{k}, k=1, \ldots, p$. Es decir, la condición de derivada direccional se cumple para todo $v \in \mathbb{R}^{p}$ si y solo si se cumple para las $2p$ direcciones canónicas $\{ \pm e_{i}: i=1, \ldots, p\}$. Así, para $v=e_{i}$ tenemos las $p$-desigualdades:

$$
0<-(\tau-1)+\xi_{i}\left(e_{i}\right) \quad i=1, \ldots, p
$$

mientras que para $v=-e_{i}$ tenemos:

$$
0<\tau-\xi_{i}\left(-e_{i}\right) \quad i=1, \ldots, p
$$

Combinando estas desigualdades, tenemos nuestra condición de optimalidad en toda su generalidad. Si ninguno de los residuos de las observaciones no básicas ($i \in \bar{h}$) es cero (como sería el caso con probabilidad uno si las $y$ tienen una densidad respecto a la medida de Lebesgue), entonces la dependencia de $\xi$ en $v$ desaparece y podemos combinar los dos conjuntos de desigualdades para obtener:

$$
(\tau-1) 1_{p} \leq \xi_{h} \leq \tau 1_{p}
$$
Resumiendo la discusión anterior, podemos reformular el Teorema de Koenker y Bassett (1978) con ayuda de la siguiente definición introducida por Rousseeuw y Leroy (1987).

**DEFINICIÓN:** Diremos que las observaciones de regresión $(y, X)$ están en posición general si cualquier subconjunto de $p$ de ellas produce un ajuste exacto único, es decir, para todo $h \in \mathcal{H}^{*}$,

$$
y_{i}-x_{i} b(h) \neq 0 \quad \text{para todo } i \notin h.
$$

Nótese que si las $y_{i}$ tienen densidad respecto a la medida de Lebesgue, entonces las observaciones $(y, X)$ estarán en posición general con probabilidad uno.

**TEOREMA.** Si $(y, X)$ están en posición general, entonces existe una solución al problema de regresión cuantílica de la forma $b(h)=X(h)^{-1} y(h)$ si y solo si para algún $h \in \mathcal{H}^{*}$ se cumple

$$
(\tau-1) 1_{p} \leq \xi_{h} \leq \tau 1_{p} \tag{2.3.1}
$$

donde $\xi_{h}=\sum_{i \in \bar{h}} \psi_{\tau}\left(y_{i}-x_{i}^{\prime} b(h)\right) x_{i}^{\prime} X(h)^{-1}$ y $\psi_{\tau}=\tau-I(u<0)$. Además, $b(h)$ es la solución única si y solo si las desigualdades son estrictas; de lo contrario, el conjunto solución es la envolvente convexa de varias soluciones de la forma $b(h)$.




## Intervalos de predicción en QR

Las estimaciones de QR pueden utilizarse para obtener intervalos de predicción de la distribución condicional de la variable respuesta. Este enfoque aprovecha la naturaleza no paramétrica de QR y ofrece una herramienta flexible para estimar intervalos de predicción en cualquier valor específico de las variables explicativas. A diferencia de los intervalos de predicción ordinarios utilizados en regresión por mínimos cuadrados, estos intervalos no son sensibles a desviaciones de los supuestos distribucionales del modelo lineal normal clásico.

Para un modelo dado, el intervalo proporcionado por dos estimaciones cuantílicas distintas, $\hat{q}_Y(\theta_1, X=x)$ y $\hat{q}_Y(\theta_2, X=x)$, en cualquier valor específico del regresor $X$, constituye un intervalo de predicción del $(\theta_2-\theta_1)\%$ para una única observación futura. Si consideramos, por ejemplo, $\theta_1=0.1$ y $\theta_2=0.9$, entonces el intervalo $[\hat{q}_Y(\theta_1=0.1, X=x); \hat{q}_Y(\theta_2=0.9, X=x)]$ puede interpretarse como un intervalo de predicción del 80\% correspondiente al valor dado $X=x$. 

El uso de este tipo de intervalo de predicción garantiza un nivel de cobertura consistente con el nivel nominal independientemente de la naturaleza del término de error, ya que QR no se ve afectado por desviaciones de los supuestos clásicos del modelo lineal normal. Además, este nivel de cobertura se mantiene estable para cualquier valor específico de $x$. Esta es una gran ventaja respecto a la regresión clásica, donde la correspondencia entre los niveles de cobertura empíricos y nominales solo se garantiza en el caso de los supuestos distribucionales usuales (modelo $_1$), mientras que se deteriora cuando aumenta el grado de violación de estos supuestos.

## Una distribución teorica para una muestra finita
Supongamos que $Y_{1}, \ldots, Y_{n}$ son variables aleatorias independientes e idénticamente distribuidas (iid) con función de distribución común $F$ y asumamos que $F$ tiene una densidad continua $f$ en una vecindad de $\xi_{\tau}=F^{-1}(\tau)$ con $f\left(\xi_{\tau}\right)>0$. La función objetivo del $\tau$-ésimo cuantil muestral

$$
\hat{\xi}_{\tau} \equiv \inf _{\xi}\left\{\xi \in \mathbb{R} \mid \sum \rho_{\tau}\left(Y_{i}-\xi\right)=\min \right\}
$$

Observe que,
    $$\rho_{\tau}(y) = [\tau-I(y<0)]|y| = |y|(1-\theta)I(y\leq 0) + |y|\theta I(y>0),$$

luego la función objetivo se puede ver como  la suma de funciones convexas y por lo tanto es convexa. 
Por otro lado, recuerde que, 
$$\begin{aligned} & \rho_\tau\left(Y_i-\xi\right)= \begin{cases}\tau\left(Y_i-\xi\right), & \text { si }\left(Y_i-\xi\right)>0 \\ (1-\tau)(-1)\left(Y_i-\xi\right) & \text { si }\left(Y_{i}- \xi\right)<0\end{cases} \\ & \rho_\tau^{\prime}\left(Y_i-\xi\right)= \begin{cases}-\tau & \text { si }\left(Y_i-\xi\right)>0 \\ (1-\tau) & \text { si }\left(Y_i-\xi\right)<0.\end{cases}, \end{aligned}$$
de este modo, la derivada respecto a $\xi$ se puede escribir como,
$$\frac{\partial\rho_\tau^\prime(Y_i-\xi)}{\partial \xi}=-\tau I_{Y_i-\xi>0}+(1-\tau)I_{Y_i-\xi<0}=I_{Y_i<\xi}-\tau  $$
Se sigue, que el gradiente tiene la forma,

$$
g_{n}(\xi)=\sum_{i=1}^{n}\left(I\left(Y_{i}<\xi\right)-\tau\right)
$$
Luego, por la monotonía del gradiente en funciones convexas se tiene,

$$
\begin{aligned}
P\left\{\hat{\xi}_{\tau}>\xi\right\} & =P\left\{g_{n}(\xi)<0\right\} \\
& =P\left\{\sum I\left(Y_{i}<\xi\right)<n \tau\right\} \\
& =P\{B(n, F(\xi))<n \tau\},
\end{aligned}
$$

donde $B(n, p)$ denota una variable aleatoria binomial con parámetros $(n, p)$. Así, dejando que $m=\lceil n \tau\rceil$ denote el entero más pequeño $\geq n \tau$, podemos expresar la función de distribución $G(\xi) \equiv P\left\{\hat{\xi}_{\tau} \leq \xi\right\}$ de $\hat{\xi}(\tau)$ usando la función beta incompleta como

$$
\begin{aligned}
G(\xi) & =\sum_{k=m}^{n}\binom{n}{k} F(\xi)^{k}(1-F(\xi))^{n-k} \\
& =n\binom{n-1}{m-1} \int_{0}^{F(\xi)} t^{m-1}(1-t)^{n-m} d t
\end{aligned}
$$

Diferenciando, obtenemos la función de densidad para $\hat{\xi}(\tau)$

$$
g(\xi)=n\binom{n-1}{m-1} F(\xi)^{m-1}(1-F(\xi))^{n-m} f(\xi)
$$

Esta forma de la densidad puede deducirse directamente notando que el evento $\{x<$ $\left.Y_{(m)}<x+\delta\right\}$ requiere que $m-1$ observaciones estén por debajo de $x$, $n-m$ estén por encima de $x+\delta$ y una esté en el intervalo $(x, x+\delta)$. El número de formas en que puede ocurrir esta disposición es

$$
\frac{n!}{(m-1)!1!(n-m)!}=n\binom{n-1}{m-1}
$$

y cada disposición tiene la probabilidad $\left.F(\xi)^{m-1}(1-F(\xi))^{n-m}[F(\xi+\delta)-F(\xi))\right]$. Así,

$$
P\left\{x<Y_{(m)}<x+\delta\right\}=n\binom{n-1}{m-1} F(\xi)^{m-1}(1-F(\xi))^{n-m} f(\xi) \delta+o\left(\delta^{2}\right),
$$

y obtenemos la función de densidad dividiendo ambos lados por $\delta$ y dejando que tienda a cero.\\
Este enfoque también puede usarse para construir intervalos de confianza para $\xi_{\tau}$ de la forma

$$
P\left\{\hat{\xi}_{\tau_{1}}<\xi_{\tau}<\hat{\xi}_{\tau_{2}}\right\}=1-\alpha
$$

donde $\tau_{1}$ y $\tau_{2}$ se eligen para satisfacer

$$
P\left\{n \tau_{1}<B(n, \tau)<n \tau_{2}\right\}=1-\alpha
$$

En el caso de $F$ continua, estos intervalos tienen la notable propiedad de ser libres de distribución, es decir, se mantienen independientemente de $F$. En el caso de $F$ discreta, se pueden construir límites libres de distribución estrechamente relacionados para $P\left\{\hat{\xi}_{\tau_{1}}<\xi_{\tau}<\hat{\xi}_{\tau_{2}}\right\}$ y $P\left\{\hat{\xi}_{\tau_{1}} \leq \xi_{\tau} \leq \hat{\xi}_{\tau_{2}}\right\}$.

La densidad de muestra finita del estimador de regresión cuantílica $\hat{\beta}(\tau)$ bajo errores iid puede derivarse de la condición de subgradiente  introducida anteriormente de una manera muy similar a la derivación de la densidad en el caso de una muestra dado anteriormente.

**Teorema de (Bassett y Koenker (1978)).**   Considere el modelo lineal

$$
Y_{i}=x_{i}^{\prime} \beta+u_{i} \quad i=1, \ldots, n,
$$

con errores iid $\left\{u_{i}\right\}$ que tienen función de distribución común $F$ y densidad estrictamente positiva $f$ en $F^{-1}(\tau)$. Entonces la densidad de $\hat{\beta}(\tau)$ toma la forma,

$$
g(b)=\sum_{h \in \mathcal{H}} P\left\{\xi_{h}(b) \in \mathcal{C}\right\}|X(h)| \prod_{i \in h} f\left(x_{i}^{\prime}(b-\beta(\tau))+F^{-1}(\tau)\right) \tag{3.1.2}
$$

donde $\xi_{h}(b)=\sum_{i \in \bar{h}} \psi_{\tau}\left(y_{i}-x_{i} b\right) x_{i}^{\prime} X(h)^{-1}$ y $\mathcal{C}$ denota el cubo $[\tau-1, \tau]^{p}$.\\
A continuación se da una demostración de este resultado. Bajo las condiciones de soluciones básicas se sabe que, $\hat{\beta}(\tau)=b(h) \equiv(X(h))^{-1} Y(h)$ si y solo si $\xi_{h}(b(h)) \in$ $\mathcal{C}$. Para cualquier $b \in \mathbb{R}^{p}$, sea $B(b, \delta)=b+[-\delta / 2, \delta / 2]^{p}$ el cubo centrado en $b$ con aristas de longitud $\delta$ y escribimos

$$
\begin{align*}
P\{\beta(\tau) \in B(b, \delta) & =\sum_{h \in \mathcal{H}} P\left\{b(h) \in B(b, \delta), \xi_{h}(b(h)) \in \mathcal{C}\right\}  \tag{3.1.3}\\
& =\sum_{h \in \mathcal{H}} E I(b(h) \in B(b, \delta)) P\left\{\xi_{h}(b(h)) \in \mathcal{C} \mid Y(h)\right\}.
\end{align*}$$


 La probabilidad condicional anterior se define a partir de la distribución de $\xi_{h}(b(h))$ que es una variable aleatoria discreta (tomando $2^{n-p}$ valores para cada $h \in \mathcal{H}$ ). Cuando $\delta \rightarrow 0$, esta probabilidad condicional tiende a $P\left\{\xi_{h}(b) \in \mathcal{C}\right\}$; esta probabilidad es independiente de $Y(h)$.

Ahora, dividimos ambos lados por Volumen $(B(b, \delta))=\delta^{p}$ y dejamos $\delta \rightarrow 0$ para expresarlo en forma de densidad. La probabilidad condicional tiende a $P\left\{\xi_{h}(b) \in \mathcal{C}\right\}$ que ya no depende de $Y(h)$. El otro factor tiende a la densidad conjunta del vector $X(h)^{-1} Y(h)$ que como

$$
f_{Y(h)}(y)=\prod_{i \in h} f\left(y_{i}-x_{i}^{\prime} \beta\right)
$$

puede escribirse como

$$
\begin{align*}
f_{(X(h))^{-1} Y(h)}(b) & =|X(h)| \prod_{i \in h} f\left((X(h) b)_{i}-x_{i}^{\prime} \beta(\tau)+F^{-1}(\tau)\right)  \tag{3.1.4}\\
& =|X(h)| \prod_{i \in h} f\left(x_{i}^{\prime}(b-\beta(\tau))+F^{-1}(\tau)\right)
\end{align*}$$


Nótese que los $h$ para los cuales $X(h)$ es singular no contribuyen a la densidad. El resultado ahora sigue al reensamblar las piezas.

Desafortunadamente, desde un punto de vista práctico los $\binom{n}{p}$ sumandos  no son muy manejables en la mayoría de las aplicaciones y, como en la teoría de mínimos cuadrados, debemos recurrir a aproximaciones asintóticas para una teoría de distribución adaptable a la inferencia estadística práctica. En la siguiente sección tratamos de proporcionar una introducción heurística a la teoría asintótica de la regresión cuantílica. 

## Algunas Heurísticas Asintóticas

La visión de optimización de los cuantiles muestrales también permite un enfoque elemental de su teoría asintótica. Supongamos que $Y_{1}, \ldots, Y_{n}$ son independientes e idénticamente distribuidos (iid) de la distribución $F$ y para algún cuantil $\xi_{\tau}=F^{-1}(\tau)$ asumamos que $F$ tiene una densidad continua $f$ en $\xi_{\tau}$ con $f\left(\xi_{\tau}\right)>0$. La función objetivo del $\tau$-ésimo cuantil muestral

$$
\hat{\xi}_{\tau} \equiv \inf _{\xi}\left\{\xi \in \mathbb{R} \mid \sum \rho_{\tau}\left(Y_{i}-\xi\right)=\min !\right\}
$$

como ya hemos notado, esta función objetivo es la suma de funciones convexas, por lo tanto es ella misma convexa. Consecuentemente su gradiente

$$
g_{n}(\xi)=n^{-1} \sum_{i=1}^{n}\left(I\left(Y_{i}<\xi\right)-\tau\right)
$$

es monótono en $\xi$. Por supuesto, cuando $\xi$ iguala a uno de los $Y_{i}$ entonces este "gradiente" necesita la interpretación de subgradiente discutida anteriormente, pero esto no es crucial para el argumento que sigue. Por monotonía, $\hat{\xi}_{\tau}$ es mayor que $\xi$ si y solo si $g_{n}(\xi)<0$. Así que

$$
\begin{aligned}
P\left\{\sqrt{n}\left(\hat{\xi}_{\tau}-\xi_{\tau}\right)>\delta\right\} & =P\left\{\left(\hat{\xi}_{\tau}\right)>\frac{\delta}{\sqrt{n}}+\xi_\tau\right\}
\\&=P\left\{g_{n}\left(\xi_{\tau}+\delta / \sqrt{n}\right)<0\right\} \\
& =P\left\{n^{-1} \sum\left(I\left(Y_{i}<\xi_{\tau}+\delta / \sqrt{n}\right)-\tau\right)<0\right\}
\end{aligned}
$$

Así, hemos reducido el comportamiento de $\hat{\xi}_{\tau}$ a un problema de teorema del límite central DeMoivre-Laplace en el que tenemos un arreglo de variables aleatorias Bernoulli. Los sumandos toman los valores $(1-\tau)$ y $-\tau$ con probabilidades $F\left(\xi_{\tau}+\delta / \sqrt{n}\right)$ y $1-F\left(\xi_{\tau}+\delta / \sqrt{n}\right)$. Dado que

$$
E [g_{n}\left(\xi_{\tau}+\delta / \sqrt{n}\right)]=\left(F\left(\xi_{\tau}+\delta / \sqrt{n}\right)-\tau\right) \rightarrow f\left(\xi_{\tau}\right) \delta / \sqrt{n}
$$

y

$$
\begin{aligned}
&V\left(g_{n}\left(\xi_{\tau}+\delta / \sqrt{n}\right)\right)=\\ &F\left(\xi_{\tau}+\delta / \sqrt{n}\right)\left(1-F\left(\xi_{\tau}+\delta / \sqrt{n}\right)\right) / n \rightarrow \tau(1-\tau) / n .
\end{aligned}$$

podemos establecer $\omega^{2}=\tau(1-\tau) / f^{2}\left(\xi_{\tau}\right)$ y escribir

$$
\begin{aligned}
P\left\{\sqrt{n}\left(\hat{\xi}_{\tau}-\xi_{\tau}\right)>\delta\right\} & =P\left\{\frac{g_{n}\left(\xi_{\tau}+\delta / \sqrt{n}\right)-f\left(\xi_{\tau}\right) \delta / \sqrt{n}}{\sqrt{\tau(1-\tau) / n}}<-\omega^{-1} \delta\right\} \\
& \rightarrow 1-\Phi\left(\omega^{-1} \delta\right)
\end{aligned}
$$

y por lo tanto


$$
\sqrt{n}\left(\hat{\xi}_{\tau}-\xi_{\tau}\right) \leadsto \mathcal{N}\left(0, \omega^{2}\right) \tag{3.2.1}
$$


El efecto $\tau(1-\tau)$ tiende a hacer $\hat{\xi}_{\tau}$ más preciso en las colas, pero esto típicamente sería dominado por el efecto del término de densidad que tiende a hacer $\hat{\xi}_{\tau}$ menos preciso en regiones de baja densidad.

Extendiendo el argumento anterior para considerar la forma límite de la distribución conjunta de varios cuantiles, sea $\hat{\zeta}_{n}=\left(\hat{\xi}_{\tau_{1}}, \ldots, \hat{\xi}_{\tau_{m}}\right)$ con $\zeta_{n}=\left(\xi_{\tau_{1}}, \ldots, \xi_{\tau_{m}}\right)$ y obtenemos


$$
\sqrt{n}\left(\hat{\zeta}_{n}-\zeta\right) \leadsto \mathcal{N}(0, \Omega) \tag{3.2.2}
$$


donde $\Omega=\left(\omega_{i j}\right)=\left(\tau_{i} \wedge \tau_{j}-\tau_{i} \tau_{j}\right) /\left(f\left(F^{-1}\left(\tau_{i}\right) f\left(F^{-1}\left(\tau_{j}\right)\right)\right.\right.$. 
Estos resultados para los cuantiles muestrales ordinarios en el modelo de una muestra se generalizan elegantemente al modelo de regresión lineal clásico

$$
y_{i}=x_{i}^{\prime} \beta+u_{i}
$$

con errores iid $\left\{u_{i}\right\}$. Supongamos que los $\left\{u_{i}\right\}$ tienen función de distribución común $F$ con densidad asociada $f$, con $f\left(F^{-1}\left(\tau_{i}\right)\right)>0$ para $i=1, \ldots, m$, y $n^{-1} \sum x_{i} x_{i}^{\prime} \equiv Q_{n}$ converge a una matriz definida positiva $Q_{0}$. Entonces la distribución asintótica conjunta de los estimadores de regresión cuantílica $m p$-variantes $\hat{\zeta}_{n}=\left(\hat{\beta}_{n}\left(\tau_{1}\right)^{\prime}, \ldots, \hat{\beta}_{n}\left(\tau_{m}\right)^{\prime}\right)^{\prime}$ toma la forma


$$
\sqrt{n}\left(\hat{\zeta}_{n}-\zeta\right)=\left(\sqrt{n}\left(\hat{\beta}_{n}\left(\tau_{j}\right)-\beta\left(\tau_{j}\right)\right)\right)_{j=1}^{m}=\mathcal{N}\left(0, \Omega \otimes Q_{0}^{-1}\right) . \tag{3.2.3}
$$


En el escenario de regresión con errores iid, la forma de los $\beta\left(\tau_{j}\right)$ es particularmente simple como hemos visto. Los planos cuantiles condicionales de $y \mid x$ son paralelos, así que suponiendo que la primera coordenada de $\beta$ corresponde al parámetro "intercepto", tenemos $\beta(\tau)=\beta+\xi_{\tau} e_{1}$ donde $\xi_{\tau}=F^{-1}(\tau)$ y $e_{1}$ es el primer vector unitario base de $\mathbb{R}^{p}$. Dado que $\Omega$ toma la misma forma que en el escenario de una muestra, muchos de los resultados clásicos sobre L-estadísticas pueden llevarse directamente a la regresión con errores iid usando este resultado.


# Principio de Equivarianza

## Principios de Equivarianza Básicos
Existen algunas características que nos gustaría que preservara nuestro modelo, por ejemplo, si cambiáramos de escala el vector de respuesta, esperaríamos que las betas estimadas del la nueva regresión cuantílica se escalaran de la misma forma. De hecho esta propiedad se cumple y es una de las propiedades que enunciamos en el siguiente teorema. Antes que nada, denotamos a la estimación de $\beta$ para el $\tau$-ésimo cuantil basado en las observaciones $(y,X)$ como $\hat{\beta}(\tau; y,X)$. 

**Teorema:** Sea $A$ una matriz de $p\times p$ no singular, $\gamma\in \mathbb{R}^{n}$ y $a>0$. Entonces, para cualquier $\tau\in [0,1]$,

$i)\quad  \hat{\beta}(\tau;ay,X) = a\hat{\beta}(\tau;y, X)$\
$ii) \quad \hat{\beta}(\tau, -ay,X) = -a\hat{\beta}(1-\tau; y,X)$\
$iii) \quad\hat{\beta}(\tau;y+X\gamma,X) = \hat{\beta}(\tau;y, X)+\gamma$\
$iv)\quad \hat{\beta}(\tau; y, XA) = A^{-1}\hat{\beta}(\tau:y, X)$\


Por poner un ejemplo, lo que estaría diciendo la primera probpiedad es: de hacer un escalamiento de la variable de respuesta, la estimación de la beta para el cuantíl $\tau$ usando las covariables originales, sería lo mismo que multiplicar el vector estimado de las betas usando como variable de respuesta el vector original y las mismas covariables.

## Equivarianza ante Transformaciones Monótonas

Sabemos que uno de los métodos más populares que se suelen ocupar para normalizar los datos de respuesta es aplicar la transformación de BoxCox , donde en muchos de los casos se toma la transformación $Y_{i}^{\lambda} = log(Y_{i})$, uno de los casos más típicos es cuándo nuestros datos están sesgados hacia la izquierda y todos son positivos.Así, hacemos regresión sobre modelo

$$Y_{i}^{(\lambda)} = \beta_{0} + \beta_{1}x_{i,1}+\dots + \beta_{n}x_{i,n} + \varepsilon_{i} 
$$

así, asumimos que $\mathbb{E}[Y^{(\lambda)}] = X^{T}\beta$ pero ocurre que no es posible recuperar el modelo original, en el sentido de que

$$ \exp\left[\mathbb{E}[Y_{i}^{(\lambda)}] \right] < \mathbb{E}[\exp\{\log(Y_{i})\}] = \mathbb{E}[Y_{i}]$$
esto por la desigualdad de Jensen. Sin embargo, hay una propiedad de los cuantiles que sí nos permiten recuperar el modelo original para este caso y se enuncia a continuación.

**Teorema:** Sea $h:\mathbb{R}\rightarrow \mathbb{R}$ una función monótona no decreciente en $\mathbb{R}$, entonces, para cualquier variable aleatoria $Y$ se tiene que 
$$Q_{h(Y)}(\tau) = h(Q_{Y}(\tau)).$$
**Dem.**
Este resultado se sigue facilmente de notar que 

$$\tau = \mathbb{P}[Y\leq y_{\tau}] = \mathbb{P}[h(Y))\leq h(y_{\tau})]$$
la última igualdad se da dado que $h$ es una función monótona no decreciente. Así, por la últimaexpresión, se tiene que 
$$Q_{h(Y)}(\tau) = h(y_{\tau}) = h(Q_{Y}(\tau)).$$

## Datos Censurados y Regresión Cuantílica

Una aplicación directa del teorema anterior y que además es una fortaleza de la regresión cuantílica es la aplicación a datos censurados. Considerese que se quiere hacer inferencia sobre el siguiente modelo 
    $$y_{i} = x_{i}^{T}\beta + \varepsilon_{i}$$
    y supongamos que $y_{i}$ no es directamente observable, que en su lugar observamos $y^{*}_{i} = \max\{0,y_{i}\}$ (Censura por la izquierda). Entonces, del resultado anterior se sigue que 
    $$Q_{y_{i}^{*}}(\tau|x) = \max\{0,x_{i}^{T}\beta(\tau)\}$$
    por lo que podemos encontrar a $\beta_{0}$ y $\beta_{1}$ mediante la solución de 
    $$\min_{\beta}\sum_{k=1}^{n}\rho(y_{i}-\max \{0, x_{i}^{T}\beta\})$$
También es posible hacer este análisis usando la función de máxima verosimilitud que nos llevaría a obtener los mismos estimadores que en la minimización del error cuadrático medio si la distirbución de los errores es Normal. En caso de que los erroes no sean normales, no se obtienen los mismos estimadores que minimizan el error cuadrático medio. Así, la virtud de la regresión cuantílica es que esta no asume una función de distirbución para los errores.

## Función de Influencia de Hampel
Ahora analizaremos el por qué bajo el criterio de la función de influencia de Hampel, la mediana es más robusta que la media. La función de influencia, introducida por Hampel, ofrece una descripción concisa de cómo un estimador $\hat{\theta}$ evaluado en una distribución $F$ se ve afectado al ``contaminar'' $F$. Formalmente, podemos ver a  $\hat{\theta}$ como una función de $F$ y escribirlo como $\hat{\theta}(F)$, y podemos considerar la contaminación de $F$ reemplazando una pequeña cantidad de masa $\varepsilon$ de $F$ por una equivalente masa concentrada en $$y$$, permitiéndonos escribir la función de distribución contaminada como

$$F_{\varepsilon}(x)=\varepsilon\delta_{y}(x)+(1-\varepsilon)F(x),$$

donde $$\delta_{y}$$ denota la función de distribución que asigna masa 1 al punto $$y$$. Expresamos la función de influencia de $$\hat{\theta}$$ en $$F$$ como

$$IF_{\hat{\theta}}(y,F)=\lim_{\varepsilon\to 0}\frac{\hat{\theta}(F_{ \varepsilon})-\hat{\theta}(F)}{\varepsilon}.$$
Calculando la función de influencia de Hampel para la mediana se tiene que

$$\hat{\theta}(F_{\varepsilon})=\int x dF_{\varepsilon}(x)=\varepsilon y+(1- \varepsilon)\hat{\theta}(F)$$

y por lo tanto

$$IF_{\hat{\theta}}(y,F)=y-\hat{\theta}(F),$$

mientras que para la mediana,

$$\hat{\theta}(F_{\varepsilon})=F_{\varepsilon}^{-1}(1/2)$$

y es posible demostrar que 

$$IF_{\hat{\theta}}(y,F)=\text{sgn}(y-\bar{\theta}(F))/f(F^{-1}(1/2)),$$
De lo anterior sacamos las siguiente conclusiones 

- En el caso de la media, la influencia de contaminar $$F$$ en $$y$$ es  proporcional a $$y$$, lo que implica que una pequeña contaminación, \textit{por mínima que sea}, en un punto $$y$$ suficientemente lejano a $$\theta(F)$$ puede alejar arbitrariamente la media de su valor inicial en $$F$$. Esto lo podemos ver dado que para una $\varepsilon$ pequeña se tiene la aproximación 
$$\hat{\theta}(F_{\varepsilon}) - \hat{\theta}(F) \approx \epsilon (y-\hat{\theta}(F))$$

- En contraste, la influencia de la contaminación en $$y$$ sobre la mediana está \textit{acotada} por la constante $$1/f(F^{-1}(1/2))$$, esto último dado que,  
$$-\frac{\varepsilon}{f(F^{-1}(\frac{1}{2}))}\leq \hat{\theta}(F_{\varepsilon}) - \hat{\theta}(F) \leq \frac{\varepsilon}{f(F^{-1}(\frac{1}{2}))}$$
aproximadamente.

Es posible extende esta misma idea a la regresión cuantílica, sin embargo, es posible demostrar que la función de información para la beta estimada en la regresión cuantílica no es sencible a lo extremos dela variable de respuesta, sin embargo, sí lo es para las covariables. 


Sea $$Y$$ una v.a., entonces:
$$
  \mu = \arg\min_c \mathbb{E}(Y - c)^2,
  \quad
  q_{0.5} = \arg\min_c \mathbb{E}|Y - c|.
$$
En general,
$$
  q_{\theta} = \arg\min_c \mathbb{E}[\rho_{\theta}(Y - c)],
  \quad
  \rho_{\theta}(y) = [\theta - I(y<0)]\,|y|.
$$


---

# Ejemplos de Aplicación

## Interpretación de pendientes

- OLS: pendiente promedio.  
- Cuantílica: pendiente por $$\tau$$.

## Visualización progresiva

Curvas cuantílicas para $$\tau = 0.1, 0.25, 0.5, 0.75, 0.9$$.

```{code-cell} ipython3
# Simular datos
np.random.seed(42)
n = 500
education = np.random.randint(5, 21, size=n)
income = 5 + 2.5 * education + (education - 12) * np.random.normal(scale=3, size=n)

df = pd.DataFrame({'education': education, 'income': income})
X = sm.add_constant(df['education'])

# Función de visualización
def plot_quantile_regression(tau=0.5):
    model = sm.QuantReg(df['income'], X).fit(q=tau)
    y_pred = model.predict(X)
    
    plt.figure(figsize=(8, 5))
    plt.scatter(df['education'], df['income'], alpha=0.3, label="Datos")
    plt.plot(df['education'], y_pred, color='red', label=f"Regresión Cuantíl (τ={tau:.2f})")
    plt.xlabel("Años de Educación")
    plt.ylabel("Ingreso estimado")
    plt.title(f"Regresión Cuantíl para τ = {tau:.2f}")
    plt.legend()
    plt.grid(True)
    plt.tight_layout()
    plt.show()

# Interactivo
interact(plot_quantile_regression, tau=FloatSlider(value=0.5, min=0.05, max=0.95, step=0.05))

```

## Distribución de ingresos

## Estudio ENIGH 2022 (CDMX): ingreso vs. educación.

En el análisis realizado con los datos de la ENIGH 2022 para la Ciudad de México, se aplicó un modelo de regresión cuantíl para estimar el efecto de la educación del jefe del hogar sobre el ingreso trimestral. Los resultados muestran una fuerte heterogeneidad en los retornos educativos a lo largo de la distribución del ingreso. Específicamente, mientras que en el cuantil 5% el coeficiente asociado a un año adicional de educación es cercano a \$2,000 MXN, en el cuantil 95% asciende a más de \$19,000 MXN. Este patrón evidencia que el efecto de la educación no es constante, sino que se amplifica significativamente en los hogares de mayores ingresos.

Estos hallazgos coinciden con lo reportado en la literatura económica. Por ejemplo, Koenker y Hallock (2001) documentan que los retornos a la educación tienden a ser mayores en los cuantiles superiores del ingreso, lo que refleja la interacción entre capital humano y segmentación del mercado laboral. Asimismo, estudios aplicados al caso mexicano, como los de López-Acevedo (2001) y Campos-Vázquez (2013), confirman que la educación produce mayores rendimientos en ocupaciones mejor remuneradas, mientras que en los estratos bajos su efecto es limitado por restricciones estructurales como la informalidad, la baja productividad o el acceso desigual a oportunidades laborales.

En conjunto, estos resultados refuerzan la necesidad de adoptar políticas complementarias a la expansión educativa, tales como mejoras en la calidad del empleo, programas de capacitación focalizados y estrategias para reducir la segmentación del mercado laboral. La regresión cuantíl permite visualizar estas desigualdades de forma más precisa que los métodos tradicionales, al capturar no solo el efecto promedio, sino la forma en que dicho efecto varía en distintos contextos sociales y económicos.


---

# Resultados Empíricos

```{code-cell} ipython3
# Formato del gráficoimport pandas as pd
import statsmodels.api as sm
import plotly.graph_objects as go
from ipywidgets import interact, FloatSlider

# --- Carga de datos (reemplaza por tus archivos) ---
df_ingresos = pd.read_csv('ingresos.csv')
df_hogar = pd.read_csv('concentradohogar.csv')

# --- Limpieza y unión ---
df_hogar = df_hogar[['folioviv', 'educa_jefe']]
df_hogar['folioviv'] = df_hogar['folioviv'].astype(str).str.zfill(10)
df_ingresos['folioviv'] = df_ingresos['folioviv'].astype(str).str.zfill(10)
df_ingresos = df_ingresos[['folioviv', 'entidad', 'ing_tri']]

df_ingresos = (
    df_ingresos[df_ingresos['entidad'] == 9]
    .groupby(['folioviv'], as_index=False)['ing_tri'].sum()
)

df = df_ingresos.merge(df_hogar, on='folioviv', how='left')
df = df.dropna()
df = df[(df['ing_tri'] > 0) & (df['educa_jefe'] >= 0)]
df = df.rename(columns={'ing_tri': 'ingreso_trimestral', 'educa_jefe': 'educacion_jefe'})

# --- Visualización interactiva con Plotly ---
def plot_quantile_plotly(tau=0.5):
    X = sm.add_constant(df['educacion_jefe'])
    y = df['ingreso_trimestral']
    model = sm.QuantReg(y, X).fit(q=tau)
    beta = model.params['educacion_jefe']
    pred = model.predict(X)

    fig = go.Figure()

    # Puntos
    fig.add_trace(go.Scatter(
        x=df['educacion_jefe'], y=y,
        mode='markers',
        name='Datos',
        marker=dict(color='rgba(0,0,0,0.3)', size=5)
    ))

    # Línea de regresión cuantíl
    fig.add_trace(go.Scatter(
        x=df['educacion_jefe'], y=pred,
        mode='lines',
        name=f'Regresión Cuantíl (τ = {tau:.2f})',
        line=dict(color='red', width=3)
    ))

    fig.update_layout(
        title=f"CDMX - ENIGH 2022 - Regresión Cuantíl (τ = {tau:.2f})",
        xaxis_title="Educación del jefe del hogar",
        yaxis_title="Ingreso trimestral (MXN)",
        template='plotly_white'
    )


    fig.add_annotation(
        text=f"β = {beta:.2f}",
        xref="paper", yref="paper",
        x=0.05, y=0.95,
        showarrow=False,
        font=dict(size=14, color="red"),
        bgcolor="rgba(255,255,255,0.7)",
        bordercolor="red",
        borderwidth=1
    )

    fig.show()

# --- Control deslizante interactivo ---
interact(plot_quantile_plotly, tau=FloatSlider(value=0.5, min=0.05, max=0.95, step=0.05))
```

## Coeficientes por cuantíl

| Cuantil $$\tau$$ | $$\hat\beta_\tau$$ (MXN/año) |
|:---------------:|:----------------------------:|
| 0.05            | 2 000                        |
| 0.50            | 5 300                        |
| 0.95            | 19 000                       |

## Interpretación

- Bajas rentas: +\$2 000  
- Media: +\$5 300  
- Altas: +\$19 000  

## Política pública

- Cobertura y calidad educativa.  
- Empleo formal y capacitación.  
- Riesgo de amplificar desigualdad.

---

# Verificación de modelos

### Métodos de validación en regresión cuantíl

A diferencia de la regresión lineal ordinaria, donde se evalúa el ajuste del modelo mediante la suma de cuadrados de los residuos y el coeficiente de determinación $$ R^2 $$, la regresión cuantíl utiliza criterios diferentes, ya que no modela la media condicional sino los cuantiles condicionales de la variable respuesta. Por ello, los métodos de validación deben adaptarse a la función de pérdida que optimiza este tipo de modelos: la pérdida de cuantíl o función de “check”.

Una primera forma de validación consiste en verificar que los residuos estén distribuidos conforme al cuantil modelado. Por ejemplo, si se estima el cuantil $$ \tau = 0.25 $$, se espera que aproximadamente el 25% de los residuos $$ e_i = y_i - \hat{Q}_\tau(x_i) $$ sean negativos. Esta proporción sirve como un chequeo rápido de la coherencia del modelo, y puede interpretarse como una forma empírica de validar si la regresión cuantíl está ubicando correctamente la frontera de probabilidad deseada.

Un segundo criterio más formal es el pseudo $$ R^1(\tau) $$, propuesto por Koenker y Machado (1999). Este coeficiente se define como una analogía al $$ R^2 $$ clásico, pero adaptado a la pérdida asimétrica de la regresión cuantíl:

$$
R^1(\tau) = 1 - \frac{\sum_{i=1}^n \rho_\tau(y_i - x_i^\top \hat{\beta}_\tau)}{\sum_{i=1}^n \rho_\tau(y_i - \tilde{y})}
$$

donde $$ \rho_\tau(u) = u(\tau - \mathbf{1}_{\{u < 0\}}) $$ es la función de pérdida de cuantíl, y $$ \tilde{y} $$ es el cuantil empírico $$ \tau $$ de la muestra, es decir, el valor que se obtendría al predecir siempre el mismo cuantíl sin usar ninguna covariable.

Este índice compara el desempeño del modelo completo con covariables contra un modelo base sin explicativas. Si el valor es cercano a 1, el modelo está explicando bien el cuantil condicional; si es cercano a 0, las covariables apenas aportan; y si es negativo, el modelo con variables explicativas resulta peor que simplemente usar el cuantil empírico de la muestra.

En conjunto, estas herramientas permiten validar modelos de regresión cuantíl respetando su estructura no paramétrica y robusta, y ofrecen criterios objetivos para comparar diferentes especificaciones de modelo o para identificar efectos heterogéneos mal capturados.


## Residuos

$$
  e_i^{(\tau)} = y_i - x_i^\top \hat\beta_\tau.
$$
Aproximadamente $$\tau\cdot100\%$$ residuos negativos.

## Pseudo $$R^1(\tau)$$



$$
  R^1(\tau)
  = 1 - 
    \frac{\sum_i \rho_\tau(y_i - x_i^\top \hat\beta_\tau)}
         {\sum_i \rho_\tau(y_i - \tilde y)},
$$
donde $$\tilde y$$ es el cuantil empírico de $$y$$ sin covariables.

## Diagnóstico gráfico

### Gráfico Q-Q empírico para validación de residuos

Otra herramienta útil para diagnosticar la calidad del ajuste en regresión cuantíl es el gráfico Q-Q (cuantil-cuantil) empírico de los residuos. A diferencia del gráfico Q-Q tradicional en regresión lineal, que compara los residuos contra una distribución normal teórica, en regresión cuantíl se utiliza un enfoque **no paramétrico y empírico**: se comparan los residuos ordenados contra los cuantiles teóricos esperados de una distribución simétrica para el nivel de $$ \tau $$ considerado.

Recordemos que si el modelo está bien especificado, los residuos $$ e_i = y_i - x_i^\top \hat{\beta}_\tau $$ deberían estar distribuidos de forma tal que aproximadamente un $$ \tau \cdot 100\% $$ de ellos sean negativos, y el resto positivos. Esta distribución esperada no es necesariamente normal, pero sí debe mantener una simetría alrededor del cero para ciertos cuantiles centrales (por ejemplo, la mediana). Por eso, en lugar de usar una referencia gaussiana, se puede comparar la distribución de los residuos observados con la de una distribución simétrica simulada o simplemente contrastar su comportamiento con lo esperado bajo el supuesto de independencia e identidad del modelo.

En la práctica, el gráfico Q-Q empírico se construye de la siguiente manera:
1. Se ordenan los residuos estimados de menor a mayor.
2. Se calculan los cuantiles teóricos esperados para $$ n $$ observaciones suponiendo una distribución simétrica alrededor de cero.
3. Se grafican los cuantiles teóricos en el eje $$ x $$ y los residuos observados en el eje $$ y $$.

Un buen ajuste se refleja en una nube de puntos alineada con la diagonal $$ y = x $$. Desviaciones sistemáticas indican asimetrías, colas más pesadas o estructuras no capturadas por el modelo. Este tipo de diagnóstico visual es especialmente útil cuando se combinan regresiones cuantílicas para distintos valores de $$ \tau $$, ya que permite comparar cómo cambia la forma de los residuos a lo largo de la distribución condicional.


### ¿Cómo se construye un Q-Q plot en regresión cuantíl?

En un Q-Q plot tradicional (como en regresión lineal), transformamos los datos usando la inversa de una distribución teórica —por ejemplo, la normal— y los comparamos con los cuantiles empíricos ordenados. Sin embargo, en regresión cuantíl este enfoque no aplica directamente porque:

- No se asume una distribución específica para los errores.
- El modelo estima cuantiles condicionales, no una función de distribución completa.
- Los residuos no son necesariamente simétricos ni normalizables.

---

### Enfoque 1: Q-Q contra cuantiles simulados bajo el modelo

1. Se calculan los residuos:  
   $$
   e_i = y_i - x_i^\top \hat{\beta}_\tau
   $$

2. Se ordenan de menor a mayor:  
   $$
   e_{(1)} < e_{(2)} < \cdots < e_{(n)}
   $$

3. Se calculan cuantiles teóricos esperados bajo la hipótesis de un modelo bien ajustado:
   - No hay una distribución explícita, pero se puede construir una referencia simétrica por remuestreo (bootstrap) o suponer una estructura empírica.

4. Se grafican los cuantiles teóricos vs. los residuos ordenados, como en un Q-Q clásico.

Este enfoque busca detectar si los residuos muestran asimetrías o estructuras no capturadas por el modelo, sin requerir una distribución paramétrica.

---

### Enfoque 2: Q-Q de residuos contra distribución uniforme centrada

Otra estrategia consiste en analizar los **residuos transformados** como si siguieran una distribución uniforme si el modelo está bien especificado:

1. Calcular los residuos $$ e_i $$.
2. Asignarles sus posiciones empíricas (ranks normalizados):  
   $$
   u_i = \frac{\text{rango}(e_i)}{n+1}
   $$

3. Compararlos con los cuantiles de una distribución uniforme $$ U(0,1) $$.

Este enfoque es más informal pero útil para validar si la dispersión y el ordenamiento de los residuos parecen razonables.

---

### Idea central

En regresión cuantíl, no comparamos contra una distribución normal teórica, sino que evaluamos si los residuos tienen la forma esperada **dados los cuantiles estimados**. Esto permite detectar asimetrías, colas pesadas o falta de ajuste sin imponer una estructura paramétrica estricta.


---

## Referencias

- Koenker, R. (2005). *Quantile Regression*. Cambridge UP.  
- Hao, L. & Naiman, D. Q. (2007). *Quantile Regression*. Sage.  
- Koenker & Hallock (2001). “Quantile Regression”, J. Econ. Perspectives.  
- Documentación de `statsmodels` en Python.  
