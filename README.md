# 🎓 SYSAC — Sistema de Administración Académica

**Del Caos al Insight: ¿qué decisión tomarías?**

Taller elaborado para **Talent Land 2026**, realizado por ***Luis Andrade*** y ***Techy Events***.

> Cada año, miles de estudiantes mexicanos abandonan sus estudios por razones que los datos ya conocían meses antes. SYSAC es el dataset sintético diseñado para que lo descubras tú mismo.

---

## 📋 Descripción del Proyecto

SYSAC es un **sistema de administración académica simulado** que modela una universidad privada en México con **10,369 estudiantes** de nivel licenciatura y posgrado. El dataset fue construido como caso de estudio para un taller práctico de ciencia de datos donde los participantes recorren el ciclo completo del dato: desde un CSV sucio y caótico hasta un modelo de clasificación que predice qué estudiantes están en riesgo de deserción y a cuáles se les puede apoyar con una beca.

La idea de fondo es simple pero poderosa: si una institución educativa ya tiene toda esta información dispersa en sus sistemas, ¿por qué sigue perdiendo estudiantes? La respuesta casi siempre está en que nadie conectó los puntos. Y es que conectar puntos entre bases de datos sucias, con formatos inconsistentes y campos vacíos, no es trivial — pero es exactamente lo que un analista de datos hace todos los días.

El sistema está compuesto por **dos fuentes de datos independientes** que reflejan cómo operan las instituciones educativas en la vida real. Una contiene lo que la escuela ya sabe del estudiante (su historial académico, perfil sociodemográfico, calificaciones, pagos). La otra contiene lo que el estudiante declara al momento de solicitar una beca a través de un cuestionario de 38 preguntas. Los participantes del taller reciben ambas fuentes como insumo y, a partir de ellas, **construyen la lógica de asignación de becas, calculan distancias de transporte y generan las evaluaciones** — exactamente como lo haría un equipo de analistas en una institución real.

---

## 🏫 Contexto Institucional

**Centro de Estudios Superiores y Posgrado de México (CESUPOM)**
Ubicación: Tecámac, Estado de México · CP 55740

Una universidad privada con **7 programas académicos** que operan en dos periodicidades distintas — cuatrimestral y semestral — tal como ocurre en muchas instituciones de educación superior en México. Los programas cubren tanto el área de humanidades como ciencias exactas, con dos maestrías que atienden perfiles de especialización profesional.

| Programa | Nivel | Periodicidad | Duración | % Matrícula |
|---|---|---|---|---|
| Pedagogía | Licenciatura | Cuatrimestral | 9 cuatrimestres | 16% |
| Psicología | Licenciatura | Cuatrimestral | 9 cuatrimestres | 27% |
| Ciencia de Datos | Licenciatura | Semestral | 7 semestres | 13% |
| Sistemas Computacionales | Licenciatura | Semestral | 8 semestres | 18% |
| Comunicaciones y Electrónica | Licenciatura | Semestral | 8 semestres | 12% |
| Psicopedagogía | Maestría | Cuatrimestral | 6 cuatrimestres | 6% |
| Ciencia de Datos e IA Aplicada | Maestría | Semestral | 4 semestres | 8% |

Las calificaciones se manejan en escala de **0 a 10**, con mínimas aprobatorias diferenciadas: **6.0 para licenciaturas** y **8.0 para posgrados**.

---

## 🏗️ Arquitectura del Dataset

El dataset se organiza en dos bloques: los **datos proporcionados** que los participantes reciben como insumo, y los **datos que construyen** durante el taller como resultado de su análisis.

### Datos Proporcionados — Base Institucional (8 tablas)

Representa la perspectiva de la institución. Son los datos que la universidad recopila como parte de su operación cotidiana: inscripciones, calificaciones, pagos, contacto. En un sistema real, estos datos vivirían en un ERP académico o en bases separadas que rara vez se hablan entre sí.

| Tabla | Descripción | Registros |
|---|---|---:|
| `HISTACA` | Historial Académico — estado, programa, periodo de cada estudiante | 10,568 |
| `PLANACA` | Plan Académico — catálogo de materias por programa | 260 |
| `INFCOM` | Información Complementaria — nombre, fecha de nacimiento, cargo | 10,509 |
| `DOMICOM` | Domicilio Complementario — calle, colonia, municipio, entidad, código postal | 10,369 |
| `CORCOM` | Correo Complementario — correos institucionales y personales | 18,626 |
| `TELCOM` | Teléfono Complementario — celulares y teléfonos fijos | 14,493 |
| `CALFIN` | Calificaciones Finales — calificación por materia, periodo y profesor | 205,442 |
| `PAGFIN` | Pagos Financieros — transacciones de inscripción, reinscripción y colegiatura | 53,263 |

### Datos Proporcionados — Solicitud de Beca (4 archivos)

Captura la voz del estudiante a través de un cuestionario estructurado, más las reglas institucionales que definen los tipos de beca y sus criterios de asignación. Estos archivos son el insumo con el que los participantes construyen la lógica de evaluación.

| Archivo | Descripción | Registros |
|---|---|---:|
| `CATBEC` | Catálogo de Tipos de Beca — 4 modalidades con sus reglas y restricciones | 4 |
| `REGBEC` | Reglas de Asignación — criterios escalonados por periodo y tipo de beca | 44 |
| `CUESTIONARIO_BECAS` | Instrumento de captura — respuestas a 38 preguntas en 7 bloques temáticos | 7,777 |
| `CATALOGO_RESPUESTAS` | Diccionario del cuestionario — tipo, opciones y dependencias de cada pregunta | 38 |

### Datos Construidos por los Participantes

Estos artefactos **no vienen incluidos en el dataset**. Son el resultado que los asistentes producen durante el taller al cruzar, limpiar y analizar las fuentes proporcionadas:

| Artefacto | Descripción | Cómo se construye |
|---|---|---|
| **Tabla de distancias (DISBEC)** | Distancia entre el CP del estudiante y la escuela (CP 55740) | Cruzando `DOMICOM` con datos de SEPOMEX/INEGI para georreferenciar códigos postales y calcular distancia geodésica |
| **Solicitudes estructuradas (SOLBEC)** | Solicitudes de beca normalizadas con datos declarados y calculados | Transformando `CUESTIONARIO_BECAS` en una tabla operativa, cruzada con la base institucional para validar información |
| **Evaluaciones de beca (EVABEC)** | Resultado de aplicar las reglas de `REGBEC` a cada solicitud | Aplicando la lógica de negocio de `CATBEC` y `REGBEC` sobre las solicitudes, promedios, materias reprobadas y distancias |
| **Modelo de clasificación** | Predicción de riesgo de deserción y elegibilidad de apoyo | Entrenando un clasificador con features derivadas de ambas bases para predecir y priorizar asignación de becas |

Esta separación es deliberada: el taller no es un ejercicio de lectura de datos — es un ejercicio de **construcción de decisiones** a partir de datos crudos.

---

## 🔑 Tipos de Beca

El sistema contempla cuatro modalidades de beca, cada una con criterios de elegibilidad distintos:

**Beca de Inscripción (BEC001)** — Apoyo promocional para estudiantes de nuevo ingreso. Se otorga una sola vez y cubre entre el 50% y el 100% del costo de inscripción. No requiere promedio mínimo. Es la beca de gancho, la que atrapa al estudiante en ese momento crítico donde decide si entra o no.

**Beca de Reinscripción (BEC002)** — Incentivo de permanencia para estudiantes en proceso de reinscripción. También se otorga una sola vez, con cobertura del 20% al 50%. Busca retener a quienes ya están dentro pero podrían no regresar el siguiente periodo.

**Beca de Colegiatura por Mérito (BEC003)** — Apoyo parcial sobre la colegiatura mensual para estudiantes con promedio destacable y cero materias reprobadas. Es la beca meritocrática: se escalonan porcentajes del 10% al 50% según el promedio general, y se puede renovar cada periodo mientras se cumplan los criterios.

**Beca de Transporte (BEC004)** — Apoyo para gastos de traslado, calculado con base en la distancia entre el código postal del estudiante y el de la institución (CP 55740, Tecámac). Los participantes del taller calculan esta distancia cruzando `DOMICOM` con datos de SEPOMEX e INEGI. Los porcentajes van del 25% al 100% según la distancia.

---

## 🔗 Relaciones Clave entre Tablas

El verdadero poder del dataset emerge cuando se cruzan las fuentes. Estas son las conexiones principales:

- **`SID`** — Identificador universal del estudiante. Presente en `HISTACA`, `INFCOM`, `DOMICOM`, `CORCOM`, `TELCOM`, `CALFIN` y `PAGFIN`. Es la llave maestra para construir el perfil completo de cada estudiante.
- **`MATRICULA`** — Validación cruzada entre tablas institucionales y el campo `P10101` del cuestionario de becas, que permite vincular lo que el estudiante declara con lo que la escuela ya registra.
- **`CVE_PROGRAMA`** — Vincula `HISTACA`, `PLANACA` y `CALFIN` para reconstruir la trayectoria académica completa.
- **`CVE_MATERIA`** — Conecta `PLANACA` con `CALFIN` para validar calificaciones contra el plan de estudios.
- **`CVE_BECA`** — Vincula `CATBEC` y `REGBEC` para conectar cada tipo de beca con sus reglas de evaluación. Los participantes usan esta conexión al construir la lógica de asignación.
- **`CP`** — El código postal en `DOMICOM` se cruza con fuentes externas (SEPOMEX, INEGI) para calcular distancias y con CONEVAL para contextualizar vulnerabilidad social.
- **`PERIODO`** — Presente en `HISTACA`, `CALFIN`, `PAGFIN` y `REGBEC`, permite análisis temporal y verificación de vigencia de reglas.

---

## 🌐 Fuentes Externas Integradas

El taller incorpora dos fuentes de datos abiertos del gobierno mexicano que los participantes utilizan para enriquecer su análisis:

**SEPOMEX (Servicio Postal Mexicano)** — Catálogo Nacional de Códigos Postales. Proporciona la georreferenciación necesaria para calcular la distancia entre el domicilio de cada estudiante y la institución, habilitando la lógica de la beca de transporte. También permite mapear código postal → municipio → entidad federativa.

**CONEVAL (Consejo Nacional de Evaluación de la Política de Desarrollo Social)** — Índice de Rezago Social a nivel municipal. Clasifica cada municipio del país en cinco grados (muy bajo, bajo, medio, alto, muy alto), lo que permite contextualizar la vulnerabilidad socioeconómica del estudiante según su zona de origen. Esto habilita una validación ética: ¿estamos asignando recursos a quien realmente los necesita, o el sistema tiene puntos ciegos?

---

## 📐 Criterios de Homogeneización

A lo largo de las tablas se aplicaron criterios de estandarización que los participantes encontrarán en algunas tablas pero no en otras — parte del reto consiste en detectar dónde la disciplina se rompió:

- **`FECHA_ACT`** — Presente en todas las tablas, siempre como último campo, formato DATE `YYYY-MM-DD` (cuando no tiene ruido)
- **`SID`** — Mismo nombre y tipo en todas las tablas donde aparece
- **`MATRICULA`** — Formato `CHAR(10)` consistente: año ingreso (4) + partición (1) + SID (5)
- **`CVE_PROGRAMA`** — Unificado en `HISTACA`, `PLANACA` y `CALFIN`
- **`CVE_MATERIA`** — Unificado en `PLANACA` y `CALFIN`
- **`PERIODO`** — Mismo nombre y formato en `HISTACA`, `CALFIN`, `PAGFIN` y `REGBEC`
- **`ENTIDAD`** — Renombrado desde `ESTADO` en `DOMICOM` para evitar ambigüedad semántica con el campo `ESTADO` de `HISTACA`

---

## 🧹 Ruido Intencional en los Datos

Porque la realidad no viene limpia, el dataset incluye **ruido controlado** que simula los problemas típicos de datos institucionales. Los participantes deben identificar, documentar y resolver estas inconsistencias antes de poder construir cualquier modelo.

- **Fechas con formatos inconsistentes** — Mezcla de `YYYY-MM-DD`, `DD/MM/YYYY` y `MM/DD/YYYY` en la misma columna
- **Valores de texto con variaciones** — Tipos de correo como `"Personal"`, `"personal"`, `"PERSONAL"`; cargos con y sin acentos
- **Registros duplicados** — Aproximadamente 2% de duplicados en `HISTACA`
- **Calificaciones con valores no numéricos** — Campos con `"N/A"`, `"NULL"` o vacíos en `CALFIN`
- **Teléfonos con formato irregular** — Incompletos, con paréntesis, dígitos extra o vacíos
- **Códigos postales truncados o con sustitución** — `"O"` en lugar de `"0"`, longitud incorrecta
- **Registros fantasma en `INFCOM`** — Algunos con SID vacío o nulo (~1.5%)
- **Espacios en blanco** al inicio o final de campos de texto
- **Acentos y caracteres especiales inconsistentes** — `"Pedagogia"` vs `"Pedagogía"`, `"Comunicaciones y Electronica"`

> **Nota:** El ruido NO está presente en las tablas de catálogo (`CATBEC`, `REGBEC`, `PLANACA`) ni en el cuestionario de becas (`CUESTIONARIO_BECAS`), ya que estos simulan interfaces con listas desplegables donde el error humano es mínimo.

---

## 📊 Cifras del Universo

| Métrica | Valor |
|---|---:|
| Total de estudiantes | 10,369 |
| Solicitudes de beca (cuestionario) | 7,777 |
| Programas académicos | 7 |
| Materias en catálogo | ~260 |
| Tipos de beca | 4 |
| Reglas de asignación | 44 |
| Profesores | 101 |
| Administrativos | 33 |

---

## 🎯 Caso de Uso: Taller "Del Caos al Insight"

Este dataset fue diseñado específicamente para un taller práctico donde los participantes construyen un pipeline completo de datos usando Python y Jupyter Notebook. El objetivo es doble: **predecir el riesgo de deserción** de cada estudiante y, con base en esa predicción, **diseñar y ejecutar la asignación de becas como herramienta de retención**. No se trata solo de diagnosticar quién se va a ir, sino de decidir a quién apoyar para que se quede.

### Temario del Taller

1. **Contexto, reto y presentación del caso** — Entender el problema de deserción estudiantil en México y por qué importa. Conocer la estructura del dataset y las preguntas que vamos a responder.

2. **Carga y exploración del dato crudo** — Leer los CSV, dimensionar el universo, identificar problemas de calidad, entender las relaciones entre tablas.

3. **Limpieza, ingeniería de features y pipeline** — Resolver inconsistencias, normalizar campos, crear variables derivadas (promedio acumulado, porcentaje de avance, saldo pendiente, distancia a la escuela, índice de rezago social), construir el dataset analítico final. Aquí es donde los participantes construyen DISBEC y SOLBEC.

4. **Exportar, entrenar y evaluar el modelo** — Clasificador que predice riesgo de deserción y elegibilidad de beca. Métricas de evaluación, validación cruzada, selección de features.

5. **Interpretación, ética y siguiente paso** — Qué significan las predicciones, qué sesgos pueden existir, qué decisiones se toman con los resultados. Los participantes ejecutan la evaluación de becas (EVABEC) aplicando las reglas de CATBEC y REGBEC.

6. **Recursos, reto y comunidad** — Herramientas para continuar aprendiendo, reto de cierre para aplicar lo aprendido, y conexión con la comunidad de datos.

---

## 📁 Estructura de Archivos

```
sysac/
├── 📂 base_institucional/          # Lo que la escuela ya sabe
│   ├── HISTACA.csv                 # Historial académico
│   ├── PLANACA.csv                 # Planes de estudio y materias
│   ├── INFCOM.csv                  # Información personal
│   ├── DOMICOM.csv                 # Domicilios
│   ├── CORCOM.csv                  # Correos electrónicos
│   ├── TELCOM.csv                  # Teléfonos
│   ├── CALFIN.csv                  # Calificaciones finales
│   └── PAGFIN.csv                  # Pagos financieros
│
├── 📂 base_becas/                  # Reglas de beca + lo que el estudiante declara
│   ├── CATBEC.csv                  # Catálogo de tipos de beca
│   ├── REGBEC.csv                  # Reglas de asignación por periodo
│   └── 📂 2026A_solicitudes/
│       ├── CUESTIONARIO_BECAS.csv  # Instrumento de captura (38 preguntas × 7,777 registros)
│       └── CATALOGO_RESPUESTAS.csv # Diccionario del cuestionario
│
├── README.md                       # Este archivo
└── README_SOLICITUDES.md           # Documentación detallada del módulo de becas y cuestionario
```

> **Nota:** Las tablas `SOLBEC` (solicitudes estructuradas), `DISBEC` (distancias) y `EVABEC` (evaluaciones) **no están incluidas** en el repositorio. Son artefactos que los participantes construyen durante el taller como parte del pipeline de datos.

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

Este dataset es **100% sintético**. Ningún registro corresponde a una persona real. Los nombres, matrículas, correos, teléfonos y demás datos fueron generados algorítmicamente con distribuciones que reflejan patrones demográficos mexicanos, pero no contienen información personal identificable.

Puedes usar este dataset libremente para fines educativos, académicos y de investigación. Si lo compartes o lo adaptas, una mención al proyecto original se agradece mucho.

---

## 🤝 Créditos

Diseñado como material de apoyo para el taller **"Del Caos al Insight: ¿qué decisión tomarías?"**, enfocado en estudiantes y profesionales que quieren aprender ciencia de datos con un caso real, relevante y con impacto social.

---

> *"Al final, no solo habrás corrido código. Habrás tomado una decisión basada en datos, y eso cambia todo."*
