# 🏛️ Sistema de Gestión para Firma Jurídica — Modelo de Base de Datos

## ¿De qué trata esto?

Una firma jurídica necesitaba una forma de organizar toda su información: los clientes que tienen, los casos legales que manejan y los procuradores encargados de cada caso. El problema es que esa información está relacionada entre sí y no se puede guardar así no más en cualquier lado sin que quede un desorden.

Entonces lo que hice fue modelar toda esa información en una base de datos relacional, desde el modelo conceptual (el famoso DER) hasta convertirlo en tablas reales con sus claves y relaciones.

---

## 🧩 Entidades identificadas y sus atributos

### `CLIENTE`
Es la persona que contrata a la firma. Se identifica de forma única por su DNI, que es lo más lógico porque no puede haber dos personas con el mismo número de documento.

| Atributo | Tipo | Detalle |
|---|---|---|
| `dni` | VARCHAR(20) | PK — identificador único |
| `nombre` | VARCHAR(100) | Obligatorio |
| `direccion` | VARCHAR(200) | Opcional (nullable) |
| `fecha_nacimiento` | DATE | Opcional (nullable) |

---

### `ASUNTO`
Representa cada caso legal que maneja la firma. Se identifica por un número de expediente único (autoincremental). El estado del caso puede ser: *en proceso*, *cerrado* o *suspendido*.

| Atributo | Tipo | Detalle |
|---|---|---|
| `num_expediente` | INT | PK — autoincremental |
| `fecha_inicio` | DATE | Obligatorio |
| `fecha_fin` | DATE | Nullable — puede que el caso siga abierto |
| `estado` | ENUM | 'en proceso', 'cerrado', 'suspendido' |
| `dni_cliente` | VARCHAR(20) | FK → `cliente` |

---

### `PROCURADOR`
Son los abogados/procuradores de la firma que llevan los casos. También se identifican por DNI. El número de colegiado es único para cada uno.

| Atributo | Tipo | Detalle |
|---|---|---|
| `dni` | VARCHAR(20) | PK |
| `nombre` | VARCHAR(100) | Obligatorio |
| `apellidos` | VARCHAR(150) | Obligatorio |
| `num_colegiado` | VARCHAR(50) | UNIQUE |
| `casos_ganados` | INT | Default 0, nullable |

---

### `ASUNTO_PROCURADOR` *(tabla intermedia)*
Esta tabla existe para resolver la relación N:M entre asuntos y procuradores. Se explica más abajo.

| Atributo | Tipo | Detalle |
|---|---|---|
| `num_expediente` | INT | FK → `asunto` + parte de la PK compuesta |
| `dni_procurador` | VARCHAR(20) | FK → `procurador` + parte de la PK compuesta |

---

## 🔗 Relaciones y cardinalidades

### CLIENTE → ASUNTO (1:N)
Un cliente puede tener **varios asuntos** a lo largo del tiempo, pero cada asunto pertenece a **un único cliente**. Esto se resuelve poniendo la FK `dni_cliente` directamente en la tabla `asunto`. Sencillo.

### ASUNTO ↔ PROCURADOR (N:M)
Acá se complica un poco. Un asunto puede ser llevado por **varios procuradores** (dependiendo de la complejidad del caso), y un procurador puede encargarse de **varios asuntos** al mismo tiempo. Eso es una relación de muchos a muchos y no se puede representar directo en las tablas sin armar un lío.

---

## ⚙️ Del modelo conceptual al modelo relacional

El proceso fue este:

1. Se identificaron las tres entidades principales: `CLIENTE`, `ASUNTO` y `PROCURADOR`.
2. La relación 1:N entre cliente y asunto se resolvió con una FK en `asunto` que apunta a `cliente`.
3. La relación N:M entre asunto y procurador **no se puede representar directamente** en SQL sin romper la normalización, entonces se creó la tabla intermedia `ASUNTO_PROCURADOR`.

El modelo relacional final quedó así:

```
CLIENTE       (dni PK, nombre, direccion, fecha_nacimiento)
ASUNTO        (num_expediente PK, fecha_inicio, fecha_fin, estado, dni_cliente FK→CLIENTE)
PROCURADOR    (dni PK, nombre, apellidos, num_colegiado, casos_ganados)
ASUNTO_PROCURADOR (num_expediente FK→ASUNTO, dni_procurador FK→PROCURADOR)
                   ↑ PK compuesta por ambas FK
```

---

## 💡 Decisiones tomadas

**¿Por qué una tabla intermedia?**
Porque en SQL no existe forma de guardar "varios valores" en una sola celda sin violar las formas normales. Si se intentara meter los procuradores directo en `asunto` (o viceversa), tocaría repetir filas o hacer columnas del tipo `procurador_1`, `procurador_2`... lo cual es un desastre. La tabla `ASUNTO_PROCURADOR` resuelve eso de forma limpia: cada fila representa simplemente *"este procurador lleva este asunto"*.

**¿Por qué `fecha_fin` y `direccion` son nullable?**
Porque son datos que pueden no existir todavía. Un caso que está en proceso no tiene fecha de cierre, y un cliente puede no tener dirección registrada. Forzar esos campos como obligatorios generaría datos falsos solo para cumplir la restricción.

**¿Por qué ENUM para `estado`?**
Para limitar los valores posibles y evitar que alguien meta cualquier texto ahí. Solo puede ser `'en proceso'`, `'cerrado'` o `'suspendido'`, que son los tres estados que define el problema.

---

## 📁 Estructura del repositorio

```
/
├── diagramas/
│   └── DER_firma_juridica.png
├── modelo_logico/
│   └── tablas_modelo_relacional.sql
└── README.md
```

---

## 🛠️ Herramientas usadas

- **DrawSQL** — para construir el diagrama visual de la base de datos
- **MySQL** — motor de base de datos de referencia para los tipos de datos# Firma-Juridica

## 🛠️ Links por si acaso no sirven las imagenes

https://ibb.co/V0xLmY7D

https://ibb.co/v4GdMw35
