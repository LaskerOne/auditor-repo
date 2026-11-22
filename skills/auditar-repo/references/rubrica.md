# Rúbrica de puntaje

Cada dimensión se califica de 0 a 10. El puntaje global es el promedio ponderado,
con un decimal y coma decimal.

| Dimensión | Peso por defecto |
|---|---|
| Pruebas y CI | 30 % |
| Seguridad y secretos | 30 % |
| Arquitectura y capas | 20 % |
| Documentación y decisiones | 20 % |

**Ajuste de pesos según el foco del usuario**, y dilo en el informe:
- "Para un reclutador" o "portafolio": documentación 30 %, seguridad 20 %.
- "Antes de producción": seguridad 40 %, documentación 10 %.
- Si una dimensión no se auditó, reparte su peso proporcionalmente entre las otras.

**Techo por hallazgos críticos:** una dimensión con un hallazgo crítico confirmado
no puede pasar de 4, por bien que esté el resto. Un secreto con privilegios
expuesto deja el puntaje global con techo de 5. Lo crítico no se compensa con
pulido.

## Qué significa cada rango

| Rango | Pruebas y CI | Seguridad y secretos | Arquitectura y capas | Documentación y decisiones |
|---|---|---|---|---|
| **0–2** | No hay pruebas ni CI | Secretos expuestos o datos accesibles sin autenticación | Todo mezclado en pocos archivos; cambiar algo rompe otra cosa | No hay README útil; no se sabe cómo correrlo |
| **3–4** | Pruebas sueltas o CI que no prueba nada | Riesgos altos sin mitigar; sin control de dependencias | Capas declaradas pero violadas de forma sistemática | README mínimo; comandos que no existen |
| **5–6** | Pruebas de lo principal; CI corre algo | Sin secretos; validaciones parciales; autorización incompleta | Estructura reconocible con violaciones puntuales | Se puede correr siguiendo el README; sin porqués |
| **7–8** | Pruebas de la lógica central con invariantes; CI con lint, pruebas y build | Autorización en cada punto de acceso; dependencias revisadas; CI con permisos mínimos | Reglas declaradas que se cumplen; módulos cohesivos | README completo; decisiones grandes explicadas; historial legible |
| **9–10** | Lo anterior más cobertura medida, pruebas de integración y verificaciones locales automáticas | Lo anterior más análisis automático en CI y modelo de amenazas | Lo anterior más fronteras verificadas automáticamente | Lo anterior más ADRs, diagrama y guía de contribución |

## Veredicto global

| Puntaje | Veredicto sugerido |
|---|---|
| 0–3,9 | No está listo para mostrarse: hay riesgos o vacíos de base. |
| 4–5,9 | Funciona, pero no lo demuestra: falta evidencia verificable. |
| 6–7,4 | Sólido para un proyecto personal; con pocas acciones se ve profesional. |
| 7,5–8,9 | Se ve profesional: un revisor externo confiaría en él. |
| 9–10 | Referencia: podría usarse como ejemplo para otros. |

Adapta la frase al repo; la tabla es un punto de partida, no una plantilla.
