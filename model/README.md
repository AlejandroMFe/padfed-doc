# PADFED - Blockchain - Modelo de Datos

Especificación del modelo de datos de la implementación basada en blockchain del Padrón Federal.

El Padrón Federal mantiene registros de contribuyentes (personas físicas o jurídica) y de personas físicas que sin ser contribuyentes están relacionadas con un contribuyente.

Los registros se identifican por una key y se persisten en formato json.

## Convenciones generales

- **`key`**: clave que identifica a un registro. La estructura de las keys respetan patrones establecidos. En la especificación de estos patrones sus componentes variables están encerrados entre llaves `{}`.
- **`min`** y **`max`**: Para los strings son longitudes y para los integers son valores.
- **`ds`**: Fecha de la más reciente modificación del registro en la base de datos de AFIP.

### Formatos

- **`#cuit`**: Número de 11 dígitos que debe cumplir con la validación de dígito verificador. Puede ser una CUIT, CUIL, CDI o CIE (Clave de Inversor del Exterior).
- **`#organización`**: Código de organización explicado en [Atributo organización](#atributo-organización)
- **`#fecha`**: Es la representación textual de una fecha con formato `YYYY-MM-DD` y los valores de `DD`, `MM` y `YYYY` deben cumplir las reglas de fechas de calendario estándar.
- Períodos:
  - **`#periodomensual`**: Formato `YYYYMM`, donde `MM` debe estar en rango [`00`, `12`] e `YYYY` debe estar en el rango [`1000`,`9999`].
  - **`#periododiario`**: Formato `YYYYMMDD`, donde `MM` debe estar en rango [`00`, `12`] y `DD` puede debe ser `00` si el `MM` es `00` o bien estar en el rango [`01`,`NN`] donde `NN` es la cantidad de días correspondiente al mes `MM` e `YYYY` debe estar en el rango [`1000`,`9999`].

## Registros de una Persona

Los datos correspondiente a una persona se persisten en un conjunto de registros de distintos tipos.

Las keys de los registros de una persona cumplen con el siguiente patrón:

`per:{id}#{tag}[:{item-id}]`

donde

- `{id}` es la clave que identifica a la persona, formato #cuit.
- `{tag}` identificador del tipo de registro, formato string(3)
- `{item-id}` identifica al ítem dentro del tipo de registro, compuesto por valores de las propiedades que conforman la clave primaria del ítem, separados por punto.

| \#  | nombre                                   | desc                                                  | tipo      | key tag | key ejemplo                           |  req  |
| --- | ---------------------------------------- | ----------------------------------------------------- | --------- | ------- | ------------------------------------- | :---: |
|     | id                                       | id de la persona                                      | #cuit     |         |                                       |   x   |
| 1   | [persona](#personapersona)               | datos identificatorios de la persona                  | objeto    | `per`   | `per:20123412340#per`                 |   x   |
| 2   | [impuestos](#personaimpuestos)           | inscripciones en impuestos                            | colección | `imp`   | `per:20123412340#imp:20`              |       |
| 3   | [domicilios](#personadomicilios)         | domicilios                                            | colección | `dom`   | `per:20123412340#dom:900.3.4`         |       |
| 4   | [domisroles](#personadomisroles)         | roles de los domicilios                               | colección | `dor`   | `per:20123412340#dor:900.3.4.3`       |       |
| 5   | [categorias](#personacategorias)         | categorias de monotributo y autonomos                 | colección | `cat`   | `per:20123412340#cat:20.1`            |       |
| 6   | [contribmunis](#personacontribmunis)     | contribuciones municipales                            | colección | `con`   | `per:20123412340#con:5244.98`         |       |
| 7   | [actividades](#personaactividades)       | actividades económicas                                | colección | `act`   | `per:20123412340#act:1.883-123456`    |       |
| 8   | [etiquetas](#personaetiquetas)           | caracterizaciones                                     | colección | `eti`   | `per:20123412340#eti:160`             |       |
| 9   | [telefonos](#personatelefonos)           | telefonos                                             | colección | `tel`   | `per:20123412340#tel:5`               |       |
| 10  | [emails](#personaemails)                 | emails                                                | colección | `ema`   | `per:20123412340#ema:2`               |       |
| 11  | [relaciones](#personarelaciones)         | relaciones con otras personas                         | colección | `rel`   | `per:20123412340#rel:20012531001.3.4` |       |
| 12  | [jurisdicciones](#personajurisdicciones) | jurisdicciones                                        | colección | `jur`   | `per:20123412340#jur:900.0`           |       |
| 13  | [cmsedes](#personacmsedes)               | provincias sedes para el convenio multilateral        | colección | `cms`   | `per:20123412340#cms:900.3`               |       |
| 14  | [archivos](#personaarchivos)             | archivos digitalizados                                | colección |  `arc`    |  `per:20123412340#arc:1` |
| 18  | [testigo](#personatestigo)               | registro unico por persona. No puede ser actualizado. | 1(uno)    | `wit`   | `per:20123412340#wit`                 |   x   |

---

### persona.persona

#### Cantidad de registros

```json
Total 16.200.000

- Contribuyentes: 15.4 millones
- No contribuyentes: 221.000
- Personas físicas: 14 millones
- Personas jurídicas: 1.5 millones
```

#### Estructura de la key

`per:{id}#per`

#### Estructura del objeto

##### Datos comunes

| nombre   | tipo     | enum       | min | max | req |
| -------- | -------- | ---------- | --- | --- | --- |
| id       | #cuit    |            |     |     | x   |
| tipoid   | string   | C, E, I, L |     |     | x   |
| tipo     | string   | F, J       |     |     | x   |
| estado   | string   | A, I       |     |     | x   |
| pais     | integer  |            | 100 | 999 |     |
| activoid | #cuit    |            |     |     |     |
| ch       | []string |            |     |     |     |
| ds       | #fecha   |            |     |     |     |

Aclaraciones:

- **`tipoid`**: C: CUIT, E: CIE, I: CDI, L: CUIT
- **`activoid`**: nueva cuit que se le asignó a la persona
- **`ch`**: array de nombres de campos cuyos valores fueron modificados en la mas reciente tx

##### Datos de personas físicas

| nombre           | tipo    | enum | min | max | req |
| ---------------- | ------- | ---- | --- | --- | --- |
| apellido         | string  |      | 1   | 200 | x   |
| nombre           | string  |      | 1   | 200 |     |
| materno          | string  |      | 1   | 200 |     |
| sexo             | string  | M, F |     |     |     |
| documento        | object  |      |     |     |     |
| documento.tipo   | integer |      | 1   | 99  | x   |
| documento.numero | string  |      |     |     | x   |
| nacimiento       | #fecha  |      |     |     |     |
| fallecimiento    | #fecha  |      |     |     |     |

Aclaraciones:

- **`materno`**: apellido materno

##### Datos de personas jurídicas

| nombre               | tipo    | enum | min | max          | req |
| -------------------- | ------- | ---- | --- | ------------ | --- |
| razonsocial          | string  |      | 1   | 200          | x   |
| formajuridica        | integer |      | 1   | 999          |     |
| mescierre            | integer |      | 1   | 12           |     |
| contratosocial       | #fecha  |      |     |              |     |
| duracion             | integer |      | 1   | 999          |     |
| inscripcion          | object  |      |     |              |     |
| inscripcion.registro | integer |      | 1   | 99           |     |
| inscripcion.numero   | integer |      | 1   | 999999999999 | x   |

Aclaraciones:

- **`inscripcion`** puede ser en IGJ (`registro: 1`) o en otro registro público de sociedades

#### Ejemplo: Persona física

`per:20000000168#per`

```json
{
    "id": 20000000168,
    "tipoid": "C",
    "tipo": "F",
    "estado": "A",
    "nombre": "XXXXX",
    "apellido": "XXXXXX",
    "materno": "XXXXX",
    "sexo": "M",
    "nacimiento": "1963-01-01",
    "fallecimiento": "2009-08-02",
    "documento": {
        "tipo": 96,
        "numero": "XX"
    },
    "ds": "2010-02-14"
}
```

#### Ejemplo: Persona jurídica

`per:30120013439#per`

```json
{
    "id": 30120013439,
    "tipoid": "C",
    "tipo": "J",
    "estado": "A",
    "razonsocial": "XXXXXXXXX XXXX XX XXXXXXXXXX XXXXXXXXXXX XXXXX",
    "formajuridica": 86,
    "mescierre": 12,
    "contratosocial": "2000-07-31",
    "inscripcion": {
        "registro": 1,
        "numero": 112345
    },
    "ds": "2008-01-21"
}
```

---

### persona.impuestos

#### Estructura de la key

`per:{id}#imp:{impuesto}`

#### Estructura de los ítems

| nombre       | tipo            | enum           | min | max    | req |
| ------------ | --------------- | -------------- | --- | ------ | --- |
| impuesto     | integer         |                | 1   | 9999   | x   |
| estado       | string          | AC, NA, BD, EX |     |        | x   |
| periodo      | #periodomensual |                |     |        | x   |
| dia          | integer         |                | 1   | 31     |     |
| motivo       | object          |                |     |        |     |
| motivo.id    | integer         |                | 1   | 999999 | x   |
| motivo.desde | #fecha          |                |     |        |     |
| motivo.hasta | #fecha          |                |     |        |     |
| inscripcion  | #fecha          |                |     |        |     |
| ds           | #fecha          |                |     |        |     |

Aclaraciones:

- **`estado`**: AC: Activo, NA: No alcanzado, BD: Baja definitiva, EX: Exento
- **`motivo.desde/hasta`**: Solamente los impuestos provinciales correspondientes regimenes simplificado de IIBB tendrán valores en estos campos

#### Ejemplo: Impuesto activo (estado AC)

`per:20000000168#imp:20`

```json
{
    "impuesto": 20,
    "periodo": 200504,
    "estado": "AC",
    "dia": 19,
    "motivo": {
        "id": 44
        },
    "inscripcion": "2005-04-20",
    "ds": "2015-12-30"
}
```

#### Ejemplo: Impuesto con baja definitiva (estado BD)

`per:20000000168#imp:5243`

```json
{
    "impuesto": 5243,
    "periodo": 201807,
    "estado": "BD",
    "dia": 31,
    "motivo": {
        "id": 44,
        "desde":"2018-04-20"
        },
    "inscripcion": "2018-06-07",
    "ds": "2018-07-10"
}
```

---

### persona.domicilios

#### Estructura de la key

`per:{id}#dom:{org}.{tipo}.{orden}`

#### Estructura de los ítems

| nombre         | tipo          | enum | min | max     | req |
| -------------- | ------------- | ---- | --- | ------- | --- |
| org            | #organización |      |     |         | x   |
| tipo           | integer       |      | 1   | 3       | x   |
| orden          | integer       |      | 1   | 9999    | x   |
| estado         | integer       |      | 1   | 99      |     |
| calle          | string        |      |     | 200     |     |
| numero         | integer       |      | 1   | 999999  |     |
| piso           | string        |      |     | 5       |     |
| sector         | string        |      |     | 200     |     |
| manzana        | string        |      |     | 200     |     |
| torre          | string        |      |     | 200     |     |
| unidad         | string        |      |     | 5       |     |
| provincia      | integer       |      | 0   | 24      |     |
| localidad      | string        |      |     | 200     |     |
| cp             | string        |      |     | 8       |     |
| nomenclador    | string        |      |     | 9       |     |
| nombre         | string        |      |     | 200     |     |
| adicional      | object        |      |     |         |     |
| adicional.tipo | integer       |      | 1   | 99      | x   |
| adicional.dato | string        |      |     | 200     | x   |
| baja           | #fecha        |      |     |         |     |
| partido        | integer       |      | 1   | 999     |     |
| partida        | integer       |      | 1   | 9999999 |     |
| ds             | #fecha        |      |     |         |     |

Aclaraciones:

- **`unidad`** es "Oficina, Departamento o Local"
- **`nombre`** es "Nombre de Fantasia"
- **`partido`** es el código del partido provincial
- **`partida`** es el número de partida inmobiliaria
- **`tipo`** indica el tipo de domicilio para AFIP. Una persona puede tener un único domicilio `tipo 1` (Fiscal para AFIP), un único domicilio `tipo 2` (Real para AFIP) y 0 a N domicilios `tipo 3`; Los domicilios informados por las jurisdiccionales (`org > 1`) siempre tienen `tipo 3`
- **`orden`** comienza desde `1` para cada `org` y `tipo`

#### Ejemplo: Domicilio fiscal para AFIP

`per:20000000168#dom:1.1.1`

```json
{
    "org": 1,
    "tipo": 1,
    "orden": 1,
    "estado": 1,
    "provincia": 1,
    "localidad": "MAR DEL PLATA SUR",
    "cp": "7600",
    "nomenclador": "1345",
    "calle": "XXXXX",
    "numero": 1000,
    "adicional": {
        "tipo": 3,
        "dato": "XXXXXXX XXXX"
    },
    "nombre": "XX XXXXXX XXXXXX",
    "ds": "2008-01-18"
}
```

---

### persona.domisroles

#### Estructura de la key #dor

`per:{id}#dor:{org}.{tipo}.{orden}.{rol}`

#### Estructura de #dor

| nombre | tipo          | enum | min | max  | req |
| ------ | ------------- | ---- | --- | ---- | --- |
| org    | #organización |      | 1   | 924  | x   |
| tipo   | integer       |      | 1   | 3    | x   |
| orden  | integer       |      | 1   | 9999 | x   |
| rol    | integer       |      | 1   | 99   | x   |
| ds     | #fecha        |      |     |      |     |

Aclaraciones:

**rol**: rol o clasificación de domicilio.

| rol | descripción                    |
| --- | ------------------------------ |
| 1   | Fiscal (Jurisdicción Sede)     |
| 2   | Principal de Actividades       |
| 3   | Fiscal (en la Jurisdicción)    |
| 4   | Otros Domicilios con Actividad |
| 11  | Sin Actividad                  |

#### Ejemplo: Rol "Fiscal Jurisdiccional" asignado por DGR Córdoba al domicilio orden 20

`per:20000000168#dor:904.3.20.3`

```json
{
    "org": 904,
    "tipo": 3,
    "orden": 20,
    "rol": 3,
    "ds":"2019-05-15"
}
```

---

### persona.categorias

#### Estructura de la key #cat

`per:{id}#cat:{impuesto}.{categoria}`

#### Estructura de #cat

| nombre    | tipo            | enum   | min    | max    | req |
| --------- | --------------- | ------ | ------ | ------ | --- |
| impuesto  | integer         |        | 1      | 9999   | x   |
| categoria | integer         |        | 1      | 999    | x   |
| estado    | string          | AC, BD |        |        | x   |
| periodo   | #periodomensual |        | 100000 | 999912 | x   |
| motivo    | #motivo         |        | 1      | 999999 |     |
| ds        | #fecha          |        |        |        |     |

#### Ejemplo

`per:20000000168#cat:20.1`

```json
{
    "impuesto": 20,
    "categoria": 1,
    "periodo": 200004,
    "estado": "AC",
    "ds": "2003-04-14"
}
```

---

### persona.contribmunis

#### Estructura de la key #con

`per:{id}#con:{impuesto}.{municipio}`

#### Estructura de #con

| nombre    | tipo    | enum | min | max  | req |
| --------- | ------- | ---- | --- | ---- | --- |
| impuesto  | integer |      | 1   | 9999 | x   |
| municipio | integer |      | 1   | 9999 | x   |
| provincia | integer |      | 0   | 24   | x   |
| desde     | #fecha  |      |     |      | x   |
| hasta     | #fecha  |      |     |      |
| ds        | #fecha  |      |     |      |

#### Ejemplo

`per:20000000168#con:5244.98`

```json
{
    "impuesto": 5244,
    "municipio": 98,
    "provincia": 3,
    "desde": "2018-06-01",
    "hasta": "2018-07-31",
    "ds": "2018-07-10"
}
```

---

### persona.actividades

#### Estructura de la key #act

`per:{id}#act:{org}.{actividad}[.{articulo}]`

#### Estructura de #act

| nombre    | tipo          | regexp pattern            | min | max | req |
| --------- | ------------- | ------------------------- | --- | --- | --- |
| org       | #organización |                           |     |     | x   |
| actividad | string        | `^[0-9]{1,3}-[0-9]{3,8}$` |     |     | x   |
| orden     | integer       |                           | 1   | 999 | x   |
| desde     | #fecha        |                           |     |     | x   |
| hasta     | #fecha        |                           |     |     |     |
| articulo  | integer       |                           | 2   | 13  |     |
| ds        | #fecha        |                           |     |     |     |

Aclaraciones:

- **`actividad`**: compuesto por codigo de nomenclador y codigo de actividad, separados por guión medio.
- **`articulo`**: utilizado solamente para las actividades de la org `900` COMARB, puede tener valor 2 o entre 6 y 13, en cuyo caso forma parte de la key.

#### Ejemplo: Actividad primaria (orden 1) para AFIP

`per:20000000168#act:1.883-772099`

```json
{
    "org": 1,
    "actividad": "883-772099",
    "orden": 1,
    "desde": 201805,
    "ds": "2018-06-07"
}
```

#### Ejemplo: Actividad secundaria (orden > 1) para COMARB

`per:20000000168#act:900.900-302000.7`

```json
{
    "org": 900,
    "actividad": "900-302000",
    "articulo": "7",
    "orden": 6,
    "desde": 201507,
    "ds": "2015-07-22"
}
```

---

### persona.etiquetas

#### Estructura de la key #eti

`per:{id}#eti:{etiqueta}`

#### Estructura de #eti

| nombre   | tipo           | enum   | min      | max      | req |
| -------- | -------------- | ------ | -------- | -------- | --- |
| etiqueta | integer        |        | 1        | 9999     | x   |
| periodo  | #periododiario |        | 10000000 | 99991231 | x   |
| estado   | string         | AC, BD |          |          | x   |
| ds       | #fecha         |        |          |          |     |

#### Ejemplo

`per:20000000168#eti:160`

```json
{
    "etiqueta": 160,
    "periodo": 19940801,
    "estado": "AC",
    "ds": "2003-04-11"
}
```

---

### persona.telefonos

#### Estructura de la key #tel

`per:{id}#tel:{orden}`

#### Estructura de #tel

| nombre | tipo    | enum | min | max             | req |
| ------ | ------- | ---- | --- | --------------- | --- |
| orden  | integer |      | 1   | 999999          | x   |
| pais   | integer |      | 1   | 9999            |     |
| area   | integer |      | 1   | 9999            |     |
| numero | integer |      | 1   | 999999999999999 | x   |
| tipo   | integer |      | 1   | 99              |     |
| linea  | integer |      | 1   | 999             |     |
| ds     | #fecha  |      |     |                 |     |

#### Ejemplo

`per:20000000168#tel:1`

```json
{
    "orden": 1,
    "pais": 200,
    "area": 11,
    "numero": 99999999,
    "tipo": 2,
    "linea": 1,
    "ds": "2013-12-16"
}
```

---

### persona.emails

#### Estructura de la key #ema

`per:{id}#ema:{orden}`

#### Estructura de #ema

| nombre    | tipo    | enum | min | max | req |
| --------- | ------- | ---- | --- | --- | --- |
| orden     | integer |      | 1   | 999 | x   |
| direccion | string  |      |     | 100 | x   |
| tipo      | integer |      | 1   | 99  |     |
| estado    | integer |      | 1   | 99  |     |
| ds        | #fecha  |      |     |     |     |

#### Ejemplo

`per:20000000168#ema:1`

```json
{
    "orden": 1,
    "direccion": "XXXXXXXXXXXXXX@XXXXX.XXX.XX",
    "tipo": 1,
    "estado": 2,
    "ds": "2016-10-20"
}
```

---

### persona.relaciones

#### Estructura de la key

`per:{id}#rel:{persona}.{tipo}.{subtipo}`

#### Estructura de los ítems

| nombre  | tipo    | enum | min | max | req |
| ------- | ------- | ---- | --- | --- | --- |
| persona | #cuit   |      |     |     | x   |
| tipo    | integer |      | 1   | 999 | x   |
| subtipo | integer |      | 0   | 999 | x   |
| desde   | #fecha  |      |     |     | x   |
| ds      | #fecha  |      |     |     |     |

Aclaraciones:

- **`tipo: 3`**: relaciones societarias.
- **`tipo: 18`**: administrador de relaciones.

#### Ejemplo: Administrador de Relaciones de una Sociedad

`per:30120013439#rel:20012531001.18.0`

```json
{
    "persona": 20012531001,
    "tipo": 18,
    "subtipo": 0,
    "desde": "2009-01-12",
    "ds": "2014-04-30"
}
```

---

### persona.jurisdicciones

#### Estructura de la key

`per:{id}#jur:{org}.{provincia}`

#### Estructura de los ítems

| nombre    | tipo          | enum | min | max | req |
| --------- | ------------- | ---- | --- | --- | --- |
| org       | #organización |      |     |     | x   |
| provincia | integer       |      | 0   | 24  | x   |
| desde     | #fecha        |      |     |     | x   |
| hasta     | #fecha        |      |     |     |     |
| ds        | #fecha        |      |     |     |     |

#### Ejemplo: Jurisdicción CABA informada por COMARB

`per:30120013439#jur:900.0`

```json
{
    "org": 900,
    "provincia": 0,
    "desde": "2019-03-01",
    "ds": "2019-05-15"
}
```

---

### persona.cmsedes

#### Estructura de la key

`per:{id}#cms:{org}.{provincia}`

#### Estructura de los ítems

| nombre    | tipo    | enum   | min | max | req |
| --------- | ------- | ----   | --- | --- | --- |
| org       | integer | 1, 900 |     |     | x   |
| provincia | integer |        | 0   | 24  | x   |
| desde     | #fecha  |        |     |     | x   |
| hasta     | #fecha  |        |     |     |     |
| ds        | #fecha  |        |     |     |     |

#### Ejemplo

`per:30120013439#cms:900.3`

```json
{
    "org": 900,
    "provincia": 3,
    "desde": "2019-06-23",
    "hasta": "2019-04-14",
    "ds": "2019-04-14"
}
```

---

### persona.archivos

#### Estructura de la key

`per:{id}#arc:{orden}`

#### Estructura de los ítems

| nombre    | tipo    | enum | min | max | req |
| --------- | ------- | ---- | --- | --- | --- |
| orden     | integer |      | 1   | 999 | x   |
| tipo      | integer |      | 1   | 99  | x   |
| ds        | #fecha  |      |     |     |     |

tipo | descripción |
--- | ---
 1|CIE - Contrato Idioma Original                           |
 2|CIE - Contrato Traducido Español                         |
 3|CIE - Autorización del AR                                |
 4|CIE - Integrantes                                        |
 5|SAS - Estatutos                                          |
 6|PERSONA FISICA - DNI FRENTE                              |
 7|PERSONA FISICA - DNI CONTRAFRENTE                        |
 8|PERSONA FISICA - COMPROBANTE DOMICILIO                   |
 9|FORMULARIO INSCRIPCION                                   |
10|CIE - Datos Administrador de la Sucesión                 |
11|CIE - Copia del Certificado de Defunción Idioma Origen   |
12|CIE - Copia del Certificado de Defunción Idioma Español  |
13|CIE - Copia de Carátula del Juicio Sucesorio             |
14|CIE - Documento de Identidad del Exterior                |
15|CIE - Integrantes Sociedad del País                      |
16|RESIDENCIA FISCAL - Certificado de Residencia            |
17|Temporario                                               |
18|PERSONA FISICA - FOTO CARNET                             |
19|PERSONA FISICA - Corrección Datos Formulario Cuit Digital|
20|PERSONA JURIDICA - Planchuela                            |

#### Ejemplo

`per:30120013439#arc:1`

```json
{
    "orden": 1,
    "tipo": 16,
    "ds": "2019-04-14"
}
```

---

### persona.testigo

#### Estructura de la key

`per:{id}#wit`

#### Estructura de los ítems

`1`

#### Ejemplo

`per:30120013439#wit`

```json
1
```

---

## Ejemplo completo

`30643202812`

| key                                    | value                                                                                                                                                                                        |
| :------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `per:30643202812#wit`                  | `1`                                                                                                                                                                                          |
| `per:30643202812#per`                  | `{"tipo":"J","id":30643202812,"tipoid":"C","estado":"A","ds":"2013-11-29","razonsocial":"XXXXX XX XXXXXXXX XXXX","formajuridica":86,"mescierre":12,"contratosocial":"1976-08-21"}`           |
| `per:30643202812#act:1.883-702091`     | `{"org":1,"actividad":"883-702091","orden":1,"desde":"2002-01-01","ds":"2019-06-03"}`                                                                                                        |
| `per:30643202812#act:900.900-949100`   | `{"org":900,"actividad":"900-949100","orden":1,"articulo":2,"desde":"2002-01-01","ds":"2019-06-03"}`                                                                                         |
| `per:30643202812#cms:900.1`            | `{"org":900,"provincia":1,"desde":"2002-01-01","ds":"2019-06-03"}`                                                                                                                      |
| `per:30643202812#dom:1.1.1`            | `{"orden":1,"org":1,"tipo":1,"estado":6,"calle":"XX XXXXXX","numero":3371,"provincia":1,"localidad":"VILLA LYNCH","cp":"1672","nomenclador":"104","ds":"2007-10-31"}`                        |
| `per:30643202812#dom:1.2.1`            | `{"orden":1,"org":1,"tipo":2,"estado":6,"calle":"XX XXXXXX","numero":1097,"provincia":1,"localidad":"VILLA LYNCH","cp":"1672","nomenclador":"104","ds":"2007-10-31"}`                        |
| `per:30643202812#dom:900.3.1`          | `{"orden":1,"org":900,"tipo":3,"estado":6,"calle":"14","numero":4745,"piso":"1","unidad":"B","provincia":1,"localidad":"VILLA LYNCH (PDO. GRAL. SAN MARTIN)","cp":"1672","ds":"2019-06-03"}` |
| `per:30643202812#dom:900.3.2`          | `{"orden":2,"org":900,"tipo":3,"estado":6,"calle":"14","numero":4745,"piso":"1","unidad":"B","provincia":1,"localidad":"VILLA LYNCH (PDO. GRAL. SAN MARTIN)","cp":"1672","ds":"2019-06-03"}` |
| `per:30643202812#dom:900.3.3`          | `{"orden":3,"org":900,"tipo":3,"estado":6,"calle":"14","numero":4745,"piso":"1","unidad":"B","provincia":1,"localidad":"VILLA FIGUEROA ALCOR","cp":"B1672WAA","ds":"2019-06-03"}`            |
| `per:30643202812#dor:900.3.1.1`        | `{"orden":1,"org":900,"tipo":3,"rol":1,"ds":"2019-06-03"}`                                                                                                                                   |
| `per:30643202812#dor:900.3.2.3`        | `{"orden":2,"org":900,"tipo":3,"rol":3,"ds":"2019-06-03"}`                                                                                                                                   |
| `per:30643202812#dor:900.3.3.2`        | `{"orden":3,"org":900,"tipo":3,"rol":2,"ds":"2019-06-03"}`                                                                                                                                   |
| `per:30643202812#imp:10`               | `{"impuesto":10,"inscripcion":"1964-01-01","estado":"EX","dia":3,"periodo":200804,"motivo":{"id":173},"ds":"2008-09-10"}`                                                                    |
| `per:30643202812#imp:32`               | `{"impuesto":32,"inscripcion":"2000-01-24","estado":"AC","dia":21,"periodo":200001,"motivo":{"id":44},"ds":"2003-03-26"}`                                                                    |
| `per:30643202812#imp:5900`             | `{"impuesto":5900,"inscripcion":"2015-01-01","estado":"AC","dia":1,"periodo":201504,"motivo":{"id":40},"ds":"2015-09-10"}`                                                                   |
| `per:30643202812#jur:900.1`            | `{"provincia":1,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                           |
| `per:30643202812#jur:900.11`           | `{"provincia":11,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.12`           | `{"provincia":12,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.16`           | `{"provincia":16,"desde":"2010-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.18`           | `{"provincia":18,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.19`           | `{"provincia":19,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.20`           | `{"provincia":20,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.22`           | `{"provincia":22,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                          |
| `per:30643202812#jur:900.3`            | `{"provincia":3,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                           |
| `per:30643202812#jur:900.4`            | `{"provincia":4,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                           |
| `per:30643202812#jur:900.5`            | `{"provincia":5,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                           |
| `per:30643202812#jur:900.6`            | `{"provincia":6,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                           |
| `per:30643202812#jur:900.7`            | `{"provincia":7,"desde":"2002-01-01","org":900,"ds":"2019-06-03"}`                                                                                                                           |
| `per:30643202812#rel:20083309424.18.0` | `{"persona":20083309424,"tipo":18,"subtipo":0,"desde":"2007-09-24","ds":"2007-09-24"}` |
| `per:30643202812#arc:1`                | `{"orden":1,"tipo":16,"ds":"2007-09-24"}` |

## Atributo organización

**#organización** conforma los registros de tipo:

- [domicilios](#personadomicilios)
- [domisroles](#personadomisroles)
- [actividades](#personaactividades)
- [jurisdicciones](#personajurisdicciones)
- [cmsedes](#personacmsedes)

En todos los caso, un valor mayor que 1(uno) indica que los datos del registro provienen directamente de la migración de datos efectuada por la organización identificada con ese valor, mientras que un valor igual a 1(uno) indica que los datos del registro fueron generados por una aplicación o proceso de AFIP.

Las organizaciones son:

| coddigo | nombre                    | provincia | MSP
| :-----: | ------------------------- | --------: | ---
|    1    | AFIP                      |           | AFIP
|   900   | COMISION ARBITRAL         |           | COMARB
|   901   | AGIP - CABA               |         0 | org901
|   902   | ARBA - BUENOS AIRES       |         1 | ARBA
|   903   | AGR - CATAMARCA           |         2 | org903
|   904   | RENTAS CORDOBA            |         3 | CBA
|   905   | DGR - CORRIENTES          |         4 | org905
|   906   | DGR - CHACO               |        16 | org906
|   907   | DGR - CHUBUT              |        17 | org907
|   908   | ATER - ENTRE RIOS         |         5 | org908
|   909   | DGR - FORMOSA             |        18 | org909
|   910   | DPR - JUJUY               |         6 | org910
|   911   | DGR - LA PAMPA            |        21 | org911
|   912   | DGIP - LA RIOJA           |         8 | org912
|   913   | ATM - MENDOZA             |         7 | org913
|   914   | DGR - MISIONES            |        19 | org914
|   915   | DPR - NEUQUEN             |        20 | org915
|   916   | DGR - RIO NEGRO           |        22 | org916
|   917   | DGR - SALTA               |         9 | org917
|   918   | DGR - SAN JUAN            |        10 | org918
|   919   | DPIP - SAN LUIS           |        11 | org919
|   920   | ASIP - SANTA CRUZ         |        23 | org920
|   921   | API - SANTA FE            |        12 | org921
|   922   | DGR - SANTIAGO DEL ESTERO |        13 | org922
|   923   | DGR - TIERRA DEL FUEGO    |        24 | org923
|   924   | DGR - TUCUMAN             |        14 | org924
