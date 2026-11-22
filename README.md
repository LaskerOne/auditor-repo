# auditor-repo

Plugin de [Claude Code](https://claude.com/claude-code) que audita un repositorio
como lo haría un líder técnico o un reclutador técnico, y entrega un informe en
español con puntaje, fortalezas, hallazgos con evidencia y un plan de acción.

> «¿Qué le falta a este proyecto para verse profesional?» — con archivo, línea y
> comando para cada respuesta.

## Cómo funciona

```
                     ┌─ auditor-pruebas-ci ─────┐
ficha del repo ──────┼─ auditor-seguridad ──────┼──► verificador-hallazgos ──► puntaje + informe
(stack, tamaño,      ├─ auditor-arquitectura ───┤    (independiente: no ve
 comandos, foco)     └─ auditor-documentacion ──┘     el razonamiento original)
                         en paralelo, solo lectura
```

1. **Ficha**: la skill reconoce el repo una sola vez (stack, tamaño, comandos, lo
   que el repo dice de sí mismo) y se la pasa a todos.
2. **Cuatro auditores en paralelo**, cada uno en su propio contexto, en solo lectura.
   Cada hallazgo debe traer evidencia comprobable; uno sin evidencia no se reporta.
3. **Un verificador independiente** revisa cada hallazgo sin conocer el argumento
   de quien lo reportó, y lo confirma, ajusta o descarta. Los descartados se
   muestran en el informe: así se ve qué se revisó y se dio por bueno.
4. **Puntaje** con una [rúbrica](skills/auditar-repo/references/rubrica.md)
   explícita: pesos por dimensión, ajustables según el foco ("para un reclutador",
   "antes de producción"), y techos cuando hay hallazgos críticos.

### Por qué subagentes y no un solo prompt

- **Paralelismo**: cuatro lecturas distintas del repo al mismo tiempo.
- **Contextos aislados**: lo que lee el auditor de seguridad no sesga al de
  arquitectura, y el contexto principal queda limpio para la síntesis.
- **Una segunda opinión real**: quien encuentra un problema tiende a confirmarlo.
  El verificador solo ve la evidencia, no el argumento.

## Instalación

```bash
/plugin marketplace add LaskerOne/auditor-repo
/plugin install auditor-repo@auditor-repo
```

## Uso

Desde la carpeta del repositorio que quieres auditar:

```
Audita este repositorio
¿Cómo vería este repo un reclutador?
¿Está listo este proyecto para pasar a producción?
Audita C:\Repos\mi-api solo en seguridad y pruebas
```

El informe queda en `auditoria-AAAA-MM-DD.md` en la raíz del repo auditado (no se
agrega a git). En el chat recibes el puntaje global y las tres primeras acciones.

## Qué revisa

| Dimensión | Peso | Ejemplos de lo que busca |
|---|---|---|
| Pruebas y CI | 30 % | Pruebas de la lógica central, invariantes, CI con lint + pruebas + build, lockfile |
| Seguridad y secretos | 30 % | Secretos en archivos e historial, RLS/autorización, entradas sin validar, permisos del CI |
| Arquitectura y capas | 20 % | Reglas que el repo declara vs. las que cumple, dependencias entre capas, módulos desproporcionados |
| Documentación y decisiones | 20 % | README que funciona, comandos que existen, porqués escritos, historial legible |

Tiene reglas específicas para **.NET** (Clean Architecture, capas por proyecto) y
para front con **React/Angular/Vue**; en otros stacks aplica el equivalente.

## Garantías

- **Solo lectura**: no instala dependencias, no modifica código, no hace commits.
  Lo único que escribe es el informe.
- **Secretos enmascarados**: si encuentra uno, reporta archivo, línea, tipo y los
  4 primeros caracteres. Nunca el valor.

## Costo y tiempo

Una auditoría completa lanza cinco subagentes y cada uno lee buena parte del repo.
En la primera prueba, sobre un repo de ~21 mil líneas, consumió **~400 mil tokens**
y tardó **15–20 minutos**. En un plan Pro eso es una porción importante de la
ventana de uso. Para gastar menos, pide un foco: «audita solo seguridad y pruebas»
lanza dos auditores en vez de cuatro.

## Límites

- **Análisis estático.** Lee código, configuración e historial de git. No ejecuta
  la app desplegada, no consulta la base de datos real y no hace pruebas de
  penetración. Lo que no pudo revisar queda listado en el informe, en «Qué no
  cubrió esta auditoría».
- **Probado en un solo repositorio** (React + Supabase). En otros stacks puede
  reportar falsos positivos o pasar cosas por alto.
- **El verificador también se equivoca.** Reduce los falsos positivos, pero no los
  elimina: revisa el informe con tu criterio antes de actuar o de entregarlo.

## Estructura

```
.claude-plugin/
  plugin.json          manifiesto del plugin
  marketplace.json     para instalarlo con /plugin marketplace add
skills/auditar-repo/
  SKILL.md             orquestación: ficha → auditores → verificador → informe
  references/rubrica.md
agents/
  auditor-pruebas-ci.md
  auditor-seguridad.md
  auditor-arquitectura.md
  auditor-documentacion.md
  verificador-hallazgos.md
```

## Estado

`0.1.0` — experimental. Primera versión, probada contra un repositorio real (React + Supabase).
Próximo: casos de evaluación con problemas plantados para medir cuántos detecta, y
reglas más profundas para .NET y AWS.

## Licencia

MIT
