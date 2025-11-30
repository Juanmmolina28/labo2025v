# Entrega_Final_Molina_Chiavassa: Pipeline de Análisis de Churn (LightGBM)

Este repositorio contiene el flujo de trabajo (workflow/pipeline) implementado para la competencia de la Maestría, 
cuyo objetivo es predecir la salida de clientes (*churn*) en una entidad financiera y optimizar la ganancia. 
El modelo central se basa en LightGBM, utilizando Feature Engineering histórico (medias móviles exponenciales - EMA) y 
optimización bayesiana de hiperparámetros.

## 🛠️ Entorno de Ejecución

El script está diseñado para correr en una máquina virtual con **R runtime**.

* **Máquina Recomendada:** `desktop-jr`
    * **Características:** 64 GB de RAM, 8 vCPU.
    * **Ubicación del Datacenter:** São Paulo, Brasil.
* **Lenguaje:** **R** (Asegúrese de cambiar el tipo de entorno de ejecución/runtime a R si usa Google Colab o Jupyter con kernel R).

---

## ⚙️ Parámetros Clave (Ajustables)

Los siguientes parámetros están definidos en el código y son críticos para la reproducibilidad del experimento:

| Parámetro | Valor por defecto | Descripción | Bloque de Código |
| :--- | :--- | :--- | :--- |
| **`PARAM$semilla_primigenia`** | `292267` | Semilla de aleatoriedad principal para la reproducibilidad. | *Inicialización* |
| **`PARAM$CA$metodo`** | `"MachineLearning"` | Método para el Catastrophe Analysis (asignar NA a variables con valores cero). | *Corregir_Rotas* |
| **`PARAM$DR$metodo`** | `"deflacion"` | Método para corregir el Data Drifting (ajuste de variables monetarias). | *Drift* |
| **`PARAM$FE_rf$arbolitos`** | `20` | Número de árboles para el Feature Engineering de Random Forest. | *FE_rf* |
| **`PARAM$trainingstrategy$training_pct`**| `0.4` | Porcentaje de *undersampling* aplicado a la clase **CONTINUA** en el training set (para velocidad). | *Training Strategy* |
| **`PARAM$hipeparametertuning$num_interations`**| `50` | Cantidad de iteraciones de la **Optimización Bayesiana**. | *Hyperparameter Tuning* |
| **`PARAM$kaggle$cortes`** | `seq(1800, 2400, by = 100)` | Rangos de cortes de probabilidad (envíos) para generar los archivos de Kaggle. | *Kaggle Submit* |

---

## 🚀 Instrucciones de Corrida

Para ejecutar y reproducir este experimento, siga los siguientes pasos:

### 1. Entorno de R y Librerías

1.  Asegúrese de que el *runtime* de su entorno (Colab, Jupyter, etc.) esté configurado en **R**.
2.  Ejecute las celdas de inicialización para limpiar la memoria y cargar las librerías necesarias (`data.table`, `R.utils`, `mice`, `lightgbm`, `DiceKriging`, `mlrMBO`, `yaml`). El script instalará automáticamente las que falten.

### 2. Preprocesamiento (Celdas 61 a 95)

1.  **Carga y Catastrophe Analysis (CA):**
    * Ejecute las celdas que cargan el *dataset* y aplican el `Catastrophe Analysis` para manejar los datos rotos (ceros generalizados).

2.  **Data Drifting (DR):**
    * Ejecute la celda que crea la tabla de índices (`tb_indices`) con `IPC`, `dolar_blue`, `dolar_oficial` y `UVA`.
    * El método actual es **`deflacion`** (ajuste por IPC), aplicado a todos los campos monetarios (`m*` variables).

3.  **Feature Engineering (FEhist):**
    * Esta es la etapa más rica, donde se crean las **medias móviles exponenciales (EMA)** con ventanas de 3, 6 y 12 meses, y las **razones (ratios)** de estas.
    * Ejecute las celdas del bloque `FEhist` para generar las nuevas variables históricas.

### 3. Modelado y Optimización (Celdas 99 a 107)

1.  **Training Setup:**
    * Ejecute las celdas que definen los conjuntos de *training* (`foto_mes` hasta `202105` con *undersampling* de 40% de la clase `CONTINUA`) y *validation* (`202107`).

2.  **Optimización Bayesiana (Hyperparameter Tuning):**
    * Ejecute el bloque que inicia la `mbo()`. Este proceso puede tardar, ya que realiza `50` iteraciones (o más, según se modifique el parámetro `PARAM$hipeparametertuning$num_interations`) para encontrar los mejores hiperparámetros de LightGBM (optimizando el **AUC** en el set de *validation*).

### 4. Producción y Entrega (Celdas posteriores)

1.  **Final Training:**
    * Una vez terminada la optimización bayesiana, el script extrae los mejores hiperparámetros (`PARAM$out$lgbm$mejores_hiperparametros`) y los utiliza para entrenar el **`final_model`** sobre todo el *final\_train* set (hasta `202107`) sin *undersampling*.

2.  **Scoring y Kaggle Submit:**
    * El modelo final se aplica al set de **futuro** (`202109`).
    * Se generan los archivos de envío (`KAxxxxxx_yyyy.csv`) para Kaggle en este caso se deja solo con 2100 envíos ya que se trata de la mejor alternativa evaluada.
