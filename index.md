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

- Generalización de la regresión de la mediana a cualquier cuantil  
  $$  
    \tau \in (0,1)  
  $$
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
- Permite obtener la forma completa de la distribución condicional de \(Y\).  
- Al variar  
  $$  
    \tau \in (0,1)  
  $$  
  se construyen curvas cuantílicas, por ejemplo:
  - Mediana condicional:  
    $$  
      \tau = 0.5  
    $$
  - Colas:  
    $$  
      \tau = 0.1, \; 0.9  
    $$
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
- Para \(\tau = 0.5\), se obtiene la mediana.  
- Se pueden estimar otros percentiles: 10%, 25%, 90%, etc.

## El cuantil como un problema de minimización 1

Sea \(Y\) una v.a., entonces:
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

## El cuantil como un problema de minimización 2

**Demostración:**

$$
  \mathbb{E}|Y - c|
  = \int_{y<c}(c - y)\,f(y)\,dy
  + \int_{y>c}(y - c)\,f(y)\,dy.
$$

## El cuantil como un problema de minimización 3

Al derivar el primer término:

$$
  \frac{d}{dc}\int_{y<c}(c - y)\,f(y)\,dy
  = F(c).
$$
Análogamente, para el segundo término:
$$
  \frac{d}{dc}\int_{y>c}(y - c)\,f(y)\,dy
  = F(c) - 1.
$$
Por tanto,
$$
  \frac{d}{dc}\mathbb{E}|Y - c|
  = 2F(c) - 1.
$$

## El cuantil como un problema de minimización 4

La condición de óptimo:
$$
  2F(c) - 1 = 0
  \;\Longleftrightarrow\;
  F(c) = \tfrac12
  \;\Longrightarrow\;
  c = q_{0.5}.
$$
Y de manera análoga se obtiene \(q_{\theta}\) para cualquier \(\theta\).

---

# Regresión Cuantílica

## Parte 1

Con la distribución empírica \(F_n\):
$$
  F_n(x) = \tfrac1n \sum_{i=1}^n \mathbf{1}_{\{Y_i \le x\}},
$$
el problema muestral es:
$$
  \min_c \tfrac1n \sum_{i=1}^n \rho_{\theta}(y_i - c).
$$
La solución es el cuantil muestral de orden \(\theta\).

## Parte 2

Asumiendo un modelo lineal condicional:
$$
  Q_Y(\tau \mid x) = x^\top \beta(\tau),
$$
los coeficientes se estiman por:
$$
  \min_{\beta}
    \sum_{i=1}^n \rho_{\theta}(y_i - x_i^\top \beta(\tau)).
$$

---

# Formulación de la RQ como un problema de Programación Lineal

## Problema general de programación lineal

- Variables de decisión \(x_i \ge 0\).  
- Objetivo:  
  $$
    z = \sum_{i=1}^n c_i x_i.
  $$
- Restricciones:
  $$
    A x \le b,
    \quad
    x \ge 0.
  $$

## Regresión Cuantíl y Programación Lineal

Minimizar:
$$
  \sum_{i=1}^n \rho_{\tau}(y_i - x_i^\top \beta),
$$
se reformula introduciendo variables de holgura y separación de partes positivas/negativas.

## Dualidad primal–dual

Para cada PL existe un dual, intercambiando costo y restricciones. En regresión cuantílica, el dual es:
$$
  \max_{z}
    \{ y^\top z \mid X^\top z = (1-\tau)\,X^\top\mathbf{1}, \; z \in [0,1]^n \}.
$$

---

# Equivarianza

## Motivación

Para la transformación lineal de la respuesta,
$$
  y^* = \tfrac59(y - 32),
$$
se derivan las relaciones entre \(\beta\) y \(\beta^*\).

## Propiedades básicas

1. \(\hat\beta(\tau; a y, X) = a\,\hat\beta(\tau; y,X)\).  
2. \(\hat\beta(\tau; -a y, X) = -a\,\hat\beta(1-\tau; y,X)\).  
3. \(\hat\beta(\tau; y + X\gamma, X) = \hat\beta(\tau; y,X) + \gamma\).  
4. \(\hat\beta(\tau; y, X A) = A^{-1}\,\hat\beta(\tau; y,X)\).

## Transformaciones monótonas

Para \(h\) monotónica no decreciente:
$$
  Q_{h(Y)}(\tau) = h\bigl(Q_Y(\tau)\bigr).
$$

## Datos censurados

Si observamos \(y_i^* = \max\{0, y_i\}\), entonces
$$
  Q_{y_i^*}(\tau \mid x)
    = \max\{0,\,x_i^\top \beta(\tau)\}.
$$

---

# Robustez

## Función de Influencia de Hampel

Definición:
$$
  IF_{\hat\theta}(y,F)
  = \lim_{\varepsilon\to0}
    \frac{\hat\theta(F_\varepsilon) - \hat\theta(F)}{\varepsilon}.
$$

- Para la media: \(IF(y,F)=y-\mu\).  
- Para la mediana: acotada por \(1/f(F^{-1}(1/2))\).  

Para regresión cuantílica:
$$
  IF_{\beta_F(\tau)}((y,x),F)
    = Q^{-1}\,x\,\mathrm{sgn}(y - x^\top \hat\beta_F(\tau)).
$$

---

# Distribución teórica para muestra finita

## Hipótesis

Si \(Y_i\) iid con F y densidad \(f\) continua en \(\xi_\tau=F^{-1}(\tau)\), el cuantil muestral:
$$
  \hat\xi_\tau
  = \inf\Bigl\{\xi : \sum \rho_\tau(Y_i - \xi)\text{ mínimo}\Bigr\}.
$$

## Función objetivo

$$
  \rho_\tau(y) = [\tau - I(y<0)]\,|y|.
$$

## Función de supervivencia

$$
  g_n(\xi) = \sum_{i=1}^n (I(Y_i < \xi) - \tau).
$$
$$
  P(\hat\xi_\tau > \xi)
  = P\{g_n(\xi) < 0\}
  = P\{B(n, F(\xi)) < n\tau\}.
$$

## Función de distribución

Si \(m=\lceil n\tau\rceil\):
$$
  G(\xi)
  = \sum_{k=m}^n \binom nk F(\xi)^k (1-F(\xi))^{n-k}.
$$

## Densidad

$$
  g(\xi)
  = n \binom{n-1}{m-1}
    F(\xi)^{m-1}(1-F(\xi))^{n-m} f(\xi).
$$

---

# Asintótica del cuantil

$$
  \sqrt{n}(\hat\xi_\tau - \xi_\tau)
  \xrightarrow{d}
  \mathcal{N}(0, \omega^2),
  \quad
  \omega^2 = \frac{\tau(1-\tau)}{f(\xi_\tau)^2}.
$$

Para el vector de coeficientes:
$$
  \sqrt{n}(\hat\zeta_n - \zeta)
  \xrightarrow{d}
  \mathcal{N}\bigl(0,\,\Omega \otimes Q_0^{-1}\bigr).
$$

---

# Ejemplos de Aplicación

## Interpretación de pendientes

- OLS: pendiente promedio.  
- Cuantílica: pendiente por \(\tau\).

## Visualización progresiva

Curvas cuantílicas para \(\tau = 0.1, 0.25, 0.5, 0.75, 0.9\).

## Distribución de ingresos

Estudio ENIGH 2022 (CDMX): ingreso vs. educación.

---

# Resultados Empíricos

## Coeficientes por cuantíl

| Cuantil \(\tau\) | \(\hat\beta_\tau\) (MXN/año) |
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

## Residuos

$$
  e_i^{(\tau)} = y_i - x_i^\top \hat\beta_\tau.
$$
Aproximadamente \(\tau\cdot100\%\) residuos negativos.

## Pseudo \(R^1(\tau)\)

$$
  R^1(\tau)
  = 1 - 
    \frac{\sum_i \rho_\tau(y_i - x_i^\top \hat\beta_\tau)}
         {\sum_i \rho_\tau(y_i - \tilde y)},
$$
donde \(\tilde y\) es el cuantil empírico de \(y\) sin covariables.

## Diagnóstico gráfico

- Residuos vs. predichos  
- Q–Q empírico  
- Fan chart de cuantílicas

---

# Conclusiones y referencias

## Conclusiones

- Amplía más allá de la media condicional.  
- Captura heterogeneidad y es robusta.  
- Sólidas propiedades estadísticas y aplicaciones.

## Preguntas

**¿Preguntas o comentarios?**

## Referencias

- Koenker, R. (2005). *Quantile Regression*. Cambridge UP.  
- Hao, L. & Naiman, D. Q. (2007). *Quantile Regression*. Sage.  
- Koenker & Hallock (2001). “Quantile Regression”, J. Econ. Perspectives.  
- Documentación de `statsmodels` en Python.  
