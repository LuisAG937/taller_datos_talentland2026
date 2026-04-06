# 🎓 SYSAC — Sistema de Administración Académica

**Del Caos al Insight: ¿qué decisión tomarías?**

> Cada año, miles de estudiantes mexicanos abandonan sus estudios por razones que los datos ya conocían meses antes. SYSAC es el dataset sintético diseñado para que lo descubras tú mismo.

---

## 📋 Descripción del Proyecto

SYSAC es un **sistema de administración académica simulado** que modela una universidad privada en México con **10,369 estudiantes** de nivel licenciatura. El dataset fue construido como caso de estudio para un taller práctico de ciencia de datos donde los participantes recorren el ciclo completo del dato: desde un CSV sucio y caótico hasta un modelo de clasificación que predice qué estudiantes están en riesgo de deserción y a cuáles se les puede apoyar con una beca.

La idea de fondo es simple pero poderosa: si una institución educativa ya tiene toda esta información dispersa en sus sistemas, ¿por qué sigue perdiendo estudiantes? La respuesta casi siempre está en que nadie conectó los puntos. Y es que conectar puntos entre bases de datos sucias, con formatos inconsistentes y campos vacíos, no es trivial — pero es exactamente lo que un analista de datos hace todos los días.

El sistema está compuesto por **dos bases de datos independientes** que reflejan cómo operan las instituciones educativas en la vida real. Una contiene lo que la escuela ya sabe del estudiante (su historial académico, perfil sociodemográfico, calificaciones, pagos). La otra contiene lo que el estudiante declara al momento de solicitar una beca (su situación económica, su motivación, su contexto familiar). Cuando cruzas ambas, empiezan a aparecer las historias que los números cuentan en silencio.

---

## 🏗️ Arquitectura del Dataset

### Base 1 — Información Institucional del Estudiante (8 tablas)

Esta base representa la perspectiva de la institución. Son los datos que la universidad recopila como parte de su operación cotidiana: inscripciones, calificaciones, pagos, contacto. En un sistema real, estos datos vivirían en un ERP académico o en bases de datos separadas que rara vez se hablan entre sí.

| Tabla | Descripción | Registros |
|---|---|---:|
| `HISTACA` | Historial Académico — estado, programa, periodo de cada estudiante | 10,568 |
| `PLANACA` | Plan Académico — catálogo de materias por programa con créditos y cuatrimestre | 260 |
| `INFCOM` | Información Complementaria — nombre, fecha de nacimiento, género, CURP | 10,509 |
| `DOMICOM` | Domicilio Complementario — calle, colonia, municipio, entidad, código postal | 10,369 |
| `CORCOM` | Correo Complementario — correos institucionales y personales | 18,626 |
| `TELCOM` | Teléfono Complementario — celulares y teléfonos fijos | 14,493 |
| `CALFIN` | Calificaciones Finales — calificación por materia, periodo y profesor | 205,442 |
| `PAGFIN` | Pagos Financieros — transacciones de inscripción, reinscripción y colegiatura | 53,263 |

### Base 2 — Sistema de Solicitudes de Beca (5 tablas)

Esta base captura la voz del estudiante. Lo que declara, lo que solicita, lo que necesita. Incluye también las reglas institucionales de asignación y los resultados de evaluación. Aquí es donde vive la lógica de negocio que determina quién recibe apoyo y quién no.

| Tabla | Descripción | Registros |
|---|---|---:|
| `CATBEC` | Catálogo de Tipos de Beca — 4 modalidades con sus reglas y restricciones | 4 |
| `REGBEC` | Reglas de Asignación — criterios escalonados por periodo y tipo de beca | 44 |
| `SOLBEC` | Solicitudes de Beca — la solicitud del estudiante con motivo, ingreso, dependientes | 7,777 |
| `EVABEC` | Evaluación de Solicitudes — resultado, monto otorgado, regla aplicada | 5,010 |
| `DISBEC` | Distancia para Beca de Transporte — distancia calculada por código postal | 10,369 |

### Cuestionario de Solicitud de Beca (2 archivos)

Además de las tablas anteriores, el dataset incluye un instrumento de captura que simula el formulario que los estudiantes llenan al solicitar beca. Contiene 38 preguntas organizadas en 7 bloques temáticos, con reglas de coherencia cruzada y campos condicionales.

| Archivo | Descripción | Registros |
|---|---|---:|
| `CUESTIONARIO_BECAS` | Respuestas del instrumento de solicitud — 38 columnas por registro | 7,777 |
| `CATALOGO_RESPUESTAS` | Diccionario de datos del instrumento — tipo, opciones y dependencias | 38 |

**Total del sistema: 15 archivos CSV · ~354,500 registros**

---

## 🔑 Tipos de Beca

El sistema contempla cuatro modalidades de beca, cada una con criterios de elegibilidad distintos:

**Beca de Inscripción (BEC001)** — Apoyo promocional para estudiantes de nuevo ingreso. Se otorga una sola vez y cubre entre el 50% y el 100% del costo de inscripción. No requiere promedio mínimo. Es la beca de gancho, la que atrapa al estudiante en ese momento crítico donde decide si entra o no.

**Beca de Reinscripción (BEC002)** — Incentivo de permanencia para estudiantes en proceso de reinscripción. También se otorga una sola vez, con cobertura del 20% al 50%. Busca retener a quienes ya están dentro pero podrían no regresar el siguiente periodo.

**Beca de Colegiatura por Mérito (BEC003)** — Apoyo parcial sobre la colegiatura mensual para estudiantes con promedio destacable y cero materias reprobadas. Es la beca meritocrática: se escalonan porcentajes del 10% al 50% según el promedio general, y se puede renovar cada periodo mientras se cumplan los criterios.

**Beca de Transporte (BEC004)** — Apoyo para gastos de traslado, calculado con base en la distancia entre el código postal del estudiante y el de la institución. Utiliza datos de referencia del INEGI para la georreferenciación. Los porcentajes van del 25% al 100% según la distancia.

---

## 🔗 Relaciones Clave entre Tablas

El verdadero poder del dataset emerge cuando se cruzan las bases. Estas son las conexiones principales:

- **`SID`** — Identificador universal del estudiante. Conecta `SOLBEC` con `HISTACA`, `INFCOM`, `DOMICOM`, `CALFIN`, `PAGFIN` y `DISBEC`. Es la llave maestra para cruzar lo que el estudiante solicita con lo que la escuela ya sabe de él.
- **`MATRICULA`** — Validación cruzada entre `SOLBEC` y `HISTACA`.
- **`CVE_BECA`** — Vincula `SOLBEC` con `CATBEC` y `REGBEC` para conectar cada solicitud con sus reglas de evaluación.
- **`CVE_SOLICITUD`** — Traza el camino completo de `SOLBEC` a `EVABEC`, desde la solicitud hasta su resolución.
- **`CP`** — El código postal en `DOMICOM` alimenta `DISBEC` para el cálculo de distancia de la beca de transporte.
- **`PERIODO`** — Presente en `REGBEC`, `SOLBEC`, `CALFIN` y `PAGFIN`, permite análisis temporal y verificación de vigencia de reglas.

---

## 🧹 Ruido Intencional en los Datos

Porque la realidad no viene limpia, el dataset incluye **ruido controlado** que simula los problemas típicos de datos institucionales. Esto es fundamental para el taller: los participantes deben identificar, documentar y resolver estas inconsistencias antes de poder construir cualquier modelo.

- **Fechas con formatos inconsistentes** — Mezcla de `YYYY-MM-DD`, `DD/MM/YYYY` y `MM/DD/YYYY` en la misma columna
- **Valores de texto con variaciones** — Géneros como `"M"`, `"Masculino"`, `"H"` o vacíos; tipos de correo como `"Personal"`, `"personal"`, `"PERSONAL"`
- **Registros duplicados** — Aproximadamente 2% de duplicados en `HISTACA`
- **CURPs incompletos o malformados** — Algunos truncados, en minúsculas o con valor `"PENDIENTE"`
- **Calificaciones con valores no numéricos** — Campos con `"N/A"`, `"NULL"` o vacíos en `CALFIN`
- **Teléfonos con formato irregular** — Incompletos, con paréntesis, dígitos extra o vacíos
- **Códigos postales truncados o con sustitución de caracteres** — `"O"` en lugar de `"0"`, longitud incorrecta
- **Registros fantasma en `INFCOM`** — Algunos con SID vacío o nulo (~1.5%)
- **Espacios en blanco al inicio o final** de campos de texto

> **Nota:** El ruido NO está presente en las tablas de catálogo (`CATBEC`, `REGBEC`, `PLANACA`) ni en el cuestionario de becas (`CUESTIONARIO_BECAS`), ya que estos simulan interfaces con listas desplegables donde el error humano es mínimo.

---

## 📊 Cifras del Universo

| Métrica | Valor |
|---|---:|
| Total de estudiantes | 10,369 |
| Solicitudes de beca | 7,777 |
| Evaluaciones procesadas | 5,010 |
| Programas académicos | 12 |
| Materias en catálogo | 260 |
| Tipos de beca | 4 |
| Reglas de asignación | 44 |
| Registros totales en el sistema | ~354,500 |

---

## 📁 Estructura de Archivos

```
sysac/
├── 📂 data/
│   ├── 📂 base_institucional/
│   │   ├── HISTACA.csv
│   │   ├── PLANACA.csv
│   │   ├── INFCOM.csv
│   │   ├── DOMICOM.csv
│   │   ├── CORCOM.csv
│   │   ├── TELCOM.csv
│   │   ├── CALFIN.csv
│   │   └── PAGFIN.csv
│   │
│   ├── 📂 base_becas/
│   │   ├── CATBEC.csv
│   │   ├── REGBEC.csv
│   │   ├── SOLBEC.csv
│   │   ├── EVABEC.csv
│   │   └── DISBEC.csv
│   │
│   └── 📂 cuestionario/
│       ├── CUESTIONARIO_BECAS.csv
│       └── CATALOGO_RESPUESTAS.csv
│
├── 📂 notebooks/
│   └── (pipeline del taller)
│
├── README.md
└── README_SOLICITUDES.md
```

---

## 🛠️ Criterios de Homogeneización

A lo largo de las tablas institucionales se aplicaron criterios de estandarización que los participantes encontrarán en algunas tablas pero no en otras — parte del reto consiste en detectar dónde la disciplina se rompió:

- **`FECHA_ACT`** — Presente en las 13 tablas, siempre como último campo, formato DATE `YYYY-MM-DD` (cuando no tiene ruido)
- **`SID`** — Mismo nombre y tipo en todas las tablas donde aparece
- **`MATRICULA`** — Formato `CHAR(10)` consistente
- **`CVE_PROGRAMA`** — Unificado en `HISTACA`, `PLANACA` y `CALFIN`
- **`CVE_MATERIA`** — Unificado en `PLANACA` y `CALFIN`
- **`PERIODO`** — Mismo nombre y formato en `HISTACA`, `CALFIN`, `PAGFIN`, `REGBEC` y `SOLBEC`
- **`ENTIDAD`** — Renombrado desde `ESTADO` en `DOMICOM` para evitar ambigüedad semántica con el campo `ESTADO` de `HISTACA`

---

## 🎯 Caso de Uso: Taller "Del Caos al Insight"

Este dataset fue diseñado específicamente para un taller práctico donde los participantes construyen un pipeline completo de datos usando Python y Jupyter Notebook. El recorrido cubre:

1. **Contexto y reto** — Entender el problema de deserción estudiantil en México y por qué importa
2. **Carga y exploración del dato crudo** — Leer los CSV, dimensionar el universo, identificar problemas
3. **Limpieza e ingeniería de features** — Resolver inconsistencias, crear variables derivadas, construir el dataset analítico
4. **Entrenamiento y evaluación del modelo** — Clasificador que predice riesgo de deserción y elegibilidad de beca
5. **Interpretación y ética** — Qué significan las predicciones, qué sesgos pueden existir, qué decisiones se toman
6. **Comunidad y siguiente paso** — Recursos para continuar aprendiendo

---

## 🧰 Requisitos Técnicos

```
Python >= 3.9
pandas
numpy
scikit-learn
matplotlib / seaborn
jupyter notebook
```

---

## 📜 Licencia y Uso

Este dataset es **100% sintético**. Ningún registro corresponde a una persona real. Los nombres, matrículas, CURPs, correos, teléfonos y demás datos fueron generados algorítmicamente con distribuciones que reflejan patrones demográficos mexicanos, pero no contienen información personal identificable.

Puedes usar este dataset libremente para fines educativos, académicos y de investigación. Si lo compartes o lo adaptas, una mención al proyecto original se agradece mucho.

---

## 🤝 Créditos

Diseñado como material de apoyo para el taller **"Del Caos al Insight: ¿qué decisión tomarías?"**, enfocado en estudiantes y profesionales que quieren aprender ciencia de datos con un caso real, relevante y con impacto social.

---

> *"Al final, no solo habrás corrido código. Habrás tomado una decisión basada en datos, y eso cambia todo."*
