---
name: auditar-repo
description: Audita un repositorio de código como lo haría un líder técnico o un reclutador técnico, con subagentes en paralelo por dimensión (pruebas y CI, seguridad y secretos, arquitectura y capas, documentación y decisiones) y un verificador independiente que descarta falsos positivos. Entrega un informe en español con puntaje por dimensión, fortalezas, hallazgos con evidencia y un plan de acción ordenado. Úsala siempre que pidan auditar, evaluar, calificar o revisar un repositorio o proyecto completo, saber si un repo "está listo" para mostrarse, publicarse, entregarse o pasar a producción, preparar un portafolio de GitHub, hacer due diligence técnico de un código ajeno, o preguntas como "¿qué le falta a este proyecto?", "¿cómo vería esto un reclutador?" o "¿qué tan senior se ve este repo?", aunque no digan "auditoría". No es para revisar un diff o un PR puntual.
---

# Auditar un repositorio

El objetivo es responder con evidencia una pregunta concreta: **¿qué tan bien
demuestra este repositorio que funciona, que es seguro y que se puede mantener?**
El informe tiene que poder sostenerse frente al autor del código: cada afirmación
apunta a un archivo, una línea o un comando que cualquiera puede repetir.

La auditoría es de **solo lectura**. No modifiques el repositorio auditado salvo
para escribir el informe, y no hagas commits.

## Por qué con subagentes

Cada dimensión exige leer partes distintas del repo. Repartirlas entre subagentes
(1) las corre en paralelo, (2) evita que la lectura de una contamine el juicio
sobre otra y (3) mantiene tu contexto limpio para la síntesis. El verificador va
aparte por una razón distinta: quien encontró un hallazgo tiende a confirmarlo; una
segunda mirada que no conoce su razonamiento lo juzga solo por la evidencia.

## Flujo

### 1. Ficha del repositorio (tú, rápido)

Antes de lanzar nada, arma una ficha corta que todos los subagentes recibirán, para
que no repitan el mismo reconocimiento cuatro veces:

- **Ruta absoluta** del repo (por defecto, el directorio de trabajo; si el usuario
  nombra otro, ese).
- **Stack**: lenguajes y frameworks, por los manifiestos (`package.json`,
  `*.csproj`/`*.sln`, `pom.xml`, `pyproject.toml`, `go.mod`…).
- **Tamaño**: número de archivos versionados (`git ls-files | wc -l`), commits,
  ramas.
- **Comandos** de prueba, lint y build si están declarados.
- **Lo que el repo dice de sí mismo**: primeras líneas del README y de
  CLAUDE.md/CONTRIBUTING si existen.
- **Foco del usuario**, si dio uno ("para un reclutador", "antes de producción").

Si la ruta no es un repositorio git o está vacía, dilo y detente.

### 2. Auditores en paralelo

Lanza los cuatro auditores **en un solo mensaje** con la herramienta Agent, cada uno
con la ficha completa en su prompt:

| Dimensión | subagent_type |
|---|---|
| Pruebas y CI | `auditor-repo:auditor-pruebas-ci` |
| Seguridad y secretos | `auditor-repo:auditor-seguridad` |
| Arquitectura y capas | `auditor-repo:auditor-arquitectura` |
| Documentación y decisiones | `auditor-repo:auditor-documentacion` |

Si esos tipos no están disponibles (la skill se instaló sin el plugin), usa
`general-purpose` y pega como prompt el contenido del archivo correspondiente en
`../../agents/<nombre>.md` (relativo a este SKILL.md), seguido de la ficha.

Si el usuario pidió solo algunas dimensiones, lanza solo esas.

### 3. Unificar duplicados (tú)

Las dimensiones se solapan: fijar las acciones del CI por SHA es seguridad y
también CI; un README que dice que no hay pruebas es documentación y también
pruebas. Si dos hallazgos piden **la misma acción**, quédate con uno, asígnalo a
la dimensión donde más pesa y anota el otro id como duplicado. Así el verificador
no revisa lo mismo dos veces y el informe no cuenta un problema doble.

### 4. Verificación

Junta los hallazgos ya unificados y lanza **un** subagente
`auditor-repo:verificador-hallazgos` (o `general-purpose` con
`../../agents/verificador-hallazgos.md`). Pásale la ruta y, por cada hallazgo, solo
`id`, `titulo`, `evidencia` y `severidad`; no le pases `por_que_importa` ni el
puntaje, para que juzgue la evidencia y no el argumento.

Si hay más de 25 hallazgos, reparte la verificación en dos o tres verificadores en
paralelo por dimensión.

### 5. Puntaje

Lee `references/rubrica.md` y calcula el puntaje de cada dimensión. Parte del
`PUNTAJE_SUGERIDO` del auditor y ajústalo según el verificador: un hallazgo
crítico o alto descartado como falso positivo puede subir el puntaje; uno ajustado
hacia arriba puede bajarlo. Explica cada ajuste en una línea.

### 6. Informe

Escríbelo en el repo auditado como `auditoria-AAAA-MM-DD.md` (fecha de hoy), salvo
que el usuario pida otra ubicación o solo verlo en el chat. No lo agregues a git.
Usa esta estructura:

```markdown
# Auditoría de <nombre del repo> — <fecha>

**Puntaje global: X,X / 10** — <veredicto en una frase, sin adornos>

## Lo primero que haría
1. <acción de mayor impacto> (<esfuerzo>) — <por qué primero>
2. …
3. …

## Puntaje por dimensión
| Dimensión | Puntaje | Peso | En una frase |
|---|---|---|---|

## Fortalezas
- <fortaleza> — `<evidencia>`

## Hallazgos confirmados
### Críticos
#### <ID> · <título>
- **Evidencia:** `<archivo:línea o comando>`
- **Por qué importa:** …
- **Acción:** … (esfuerzo S/M/L)
### Altos · Medios · Bajos
…

## Plan de acción
| # | Acción | Hallazgos | Esfuerzo | Impacto |
|---|---|---|---|---|
<ordenado: primero lo crítico; luego lo de mayor impacto por menor esfuerzo>

## Descartados por el verificador
| ID | Título | Por qué se descartó |
|---|---|---|

## Qué no cubrió esta auditoría
- <lo marcado como NO_REVISADO o NO_VERIFICABLE, y cómo comprobarlo>
```

La sección de descartados no es relleno: muestra que el informe se filtró y deja
que el autor vea qué se revisó y se dio por bueno.

### 7. En el chat

Responde con el puntaje global, las tres primeras acciones y la ruta del informe.
No repitas el informe completo.

## Criterio

- **Evidencia o nada.** Un hallazgo que no se puede comprobar no entra al informe.
- **Fortalezas con la misma exigencia.** Decir qué está bien hecho es parte del
  trabajo: ayuda a decidir qué conservar y hace creíbles las críticas.
- **Contra lo que el repo dice de sí mismo.** Una regla que el repo declara y no
  cumple pesa más que una práctica genérica que no adoptó.
- **Secretos enmascarados siempre.** Nunca copies un valor secreto al informe ni al
  chat; si hay uno expuesto, rotarlo es la primera acción del plan.
- **Tono de colega senior.** Directo, específico, sin adjetivos. "No hay pruebas de
  `src/lib/calculos.ts`" ayuda; "la cobertura es deficiente" no.
