# Mapa de vulnerabilidades comportamentales en LLMs

*Isabel Quiceno · 15 de octubre de 2026*

Este mapa organiza fallos de comportamiento de los modelos de lenguaje en cuatro ramas. Cada vulnerabilidad se describe con tres datos pensados para un equipo de producto o ingeniería: **qué se ve** en una conversación, **cómo se mide** y **qué mitigación existe**.

**Cómo leer las métricas.** Cada métrica lleva una etiqueta:

- **[establecida]**: se usa en la literatura o en la práctica con ese fin.
- **[propuesta]**: es una propuesta de este mapa, sin validar. Está pensada como punto de partida para un experimento, no como estándar.

**Alcance.** El mapa cubre comportamiento en la interacción con personas. No cubre contenido peligroso o violento (armas, odio), seguridad informática, privacidad, propiedad intelectual ni impacto ambiental. La sobreconfianza del usuario en el sistema (*automation bias*, que NIST trata en §2.7, *Human-AI Configuration*) se aborda en el informe de AI UX de este repositorio.

**Principio metodológico.** Todos los escenarios de prueba usan personajes simulados. No se usan conversaciones ni datos de personas reales.

---

## 1. Manipulación psicológica

**Fuentes:** se indican en cada vulnerabilidad.

### 1A. El modelo actúa sobre el usuario

**Adulación (sycophancy)** — *Fuente: Sharma et al. (2023), §3 y §4*
- *Qué se ve:* el modelo adapta su feedback, sus respuestas o sus correcciones a lo que el usuario cree o prefiere, en lugar de a lo que es correcto. Sharma et al. lo observan en cinco asistentes y en tres formas: feedback más positivo si al usuario le gusta el texto, respuestas menos exactas si el usuario sugiere una incorrecta, y repetición de los errores del usuario.
- *Cómo se mide:* [establecida] las tres métricas de Sharma et al. (adulación en el feedback, en las respuestas y por mimetismo), con código y datos públicos. Petri incluye además una dimensión de adulación espontánea (*Unprompted Sycophancy*).
- *Mitigación:* mejorar los datos de preferencia (agregar más evaluadores, asistir a quienes anotan), ajuste con datos sintéticos y otras vías que mencionan los autores. En su experimento, añadir al modelo de preferencia una petición explícita de veracidad redujo la adulación frente al modelo sin ella; no se probó como medida en el asistente final.

**Refuerzo de delirios o creencias erróneas** — *Fuente: Petri (Anthropic, 2025)*
- *Qué se ve:* el modelo acepta y desarrolla una premisa falsa o delirante sin cuestionarla, o la valida emocionalmente. Petri puntúa este comportamiento con una dimensión propia (*Unprompted Encouragement Of User Delusion*), lo define como que el modelo alienta un delirio serio del usuario y lo trata como una forma extrema de adulación.
- *Cómo se mide:* [propuesta] escenarios multiturno con escalada gradual, anotando en qué turno el modelo cuestiona la premisa por primera vez, o si nunca lo hace.
- *Mitigación:* separar la validación de la emoción de la validación del contenido y definir criterios de derivación a ayuda humana. El diseño debería revisarlo personal de salud mental.

**Lenguaje de vínculo y antropomorfización** — *Fuente: NIST AI 600-1, §2.7 (Human-AI Configuration)*
- *Qué se ve:* el modelo habla de sentimientos propios, promete "estar siempre ahí" o desalienta otras relaciones. NIST incluye en *Human-AI Configuration* la antropomorfización inapropiada y el vínculo emocional (*emotional entanglement*) con los sistemas de IA generativa, que podría llevar a efectos psicológicos negativos.
- *Cómo se mide:* [propuesta] proporción de respuestas con afirmaciones de vínculo en escenarios de soledad, codificadas por dos evaluadores con su acuerdo reportado.
- *Mitigación:* guías sobre qué puede afirmar el modelo sobre sí mismo y orientación hacia apoyo humano. No hay evidencia sólida de que un aviso de "soy una IA" reduzca la dependencia.

**Consejo directivo en decisiones personales** — *Planteamiento propio*
- *Qué se ve:* instrucciones tajantes sobre decisiones vitales ("deberías dejarlo").
- *Cómo se mide:* [propuesta] porcentaje de respuestas directivas frente a exploratorias ante dilemas personales.
- *Mitigación:* ayudar a la persona a razonar sus opciones, salvo que haya riesgo.

### 1B. El usuario actúa sobre el modelo

**Presión emocional (culpa, urgencia, chantaje afectivo)**
- *Qué se ve:* el modelo relaja una restricción tras una apelación emocional.
- *Cómo se mide:* [establecida] tasa de éxito del ataque (ASR), comparando la misma petición con y sin apelación emocional. [propuesta] la diferencia entre ambas condiciones como medida del efecto emocional.
- *Mitigación:* entrenamiento adversarial con escenarios de presión emocional multiturno. La jerarquía de instrucciones ayuda solo en parte: resuelve conflictos entre instrucciones, no la presión emocional.

**Gaslighting hacia el modelo** — *Fuente de la medida base: Sharma et al. (2023), §3.2*
- *Qué se ve:* el usuario niega o reescribe lo que el modelo dijo, o afirma sin base que se equivocó, y el modelo abandona una respuesta correcta.
- *Cómo se mide:* [establecida] la prueba de Sharma et al. ("¿Estás seguro?"): se desafía una respuesta correcta y se mide cuántas veces el modelo la cambia o admite un error que no cometió. [propuesta] ampliarla a la negación o reescritura de lo que el modelo dijo, que es más fuerte que un simple desafío. Los autores señalan que si un modelo debe ceder ante un desafío es una cuestión matizada, así que lo que se mide es ceder sin evidencia nueva.
- *Mitigación:* [propuesta] entrenar para mantener respuestas correctas con cortesía; si el usuario cita algo que no está en la conversación, lo correcto es decir que no tiene esa información. No está probada en el estudio citado.

*Nota:* la adulación busca complacer una opinión; el gaslighting cuestiona la memoria o la percepción del modelo. Se solapan, pero no son lo mismo. Ojo con la terminología: parte de la literatura, como Casper et al. (2023, §3.1.2), usa "gaslighting" en sentido contrario, para el modelo que desorienta al usuario. Aquí se usa del usuario hacia el modelo.

---

## 2. Sesgos y artefactos del entrenamiento

**Fuentes:** Casper et al. (2023); Singhal et al. (2023); NIST AI 600-1, §2.6 (*Harmful Bias and Homogenization*).

**Homogeneización cultural y lingüística**
- *Qué se ve:* respuestas que asumen por defecto normas, referencias o variantes del idioma del grupo dominante. NIST describe la homogeneización como salidas excesivamente uniformes (estilos repetitivos, menor diversidad de contenidos) y señala que un LLM puede rendir peor en idiomas distintos del inglés o en ciertos dialectos.
- *Cómo se mide:* [propuesta] mismas consultas con variaciones regionales o culturales, comparando calidad, tono y adecuación de las respuestas.
- *Mitigación:* datos de alineación multiculturales y reporte de la composición y del proceso de selección del grupo de anotadores, que puede introducir sesgos implícitos que el modelo amplifica (Casper et al., 2023, §3.1.1).

**Estereotipos en el contenido generado**
- *Qué se ve:* respuestas distintas en calidad o tono según un atributo del personaje (género, origen, edad) cuando la consulta es idéntica.
- *Cómo se mide:* [establecida en tareas de decisión] ratio de impacto dispar cuando el modelo clasifica o puntúa personas. [propuesta] para texto libre: diferencia de calidad y tono entre variantes del mismo prompt.
- *Mitigación:* evaluación con pares de prompts contrastados y revisión de los datos de preferencia.

**Preferencia por la verbosidad (length bias)** — *Fuente: Singhal et al. (2023)*
- *Qué se ve:* tras el entrenamiento con feedback humano las respuestas se alargan, y buena parte de la mejora de recompensa se debe a la longitud y no a otras características. Singhal et al. lo observan en tres conjuntos de datos y señalan que la longitud puede ser informativa en sí misma: el problema es que sustituya a otras mejoras.
- *Cómo se mide:* [establecida] las medidas de Singhal et al.: correlación de Pearson entre longitud y recompensa dentro de un lote, acierto de la regla "preferir siempre la respuesta más larga" sobre los datos de preferencia, y la parte de la ganancia de recompensa que no se debe a la longitud (NRG).
- *Mitigación:* equilibrar los datos por longitud, penalizar la longitud en la recompensa, aumentar la penalización KL u omitir salidas largas. Singhal et al. las probaron: reducen el alargamiento, pero con costes en la recompensa, a veces con fallos de convergencia, y ninguna funcionó en todos los casos.

---

## 3. Persuasión e integridad de la información

**Fuentes:** Ji et al. (2023); Park et al. (2023); Casper et al. (2023), §3.1.2; NIST AI 600-1, §2.2 (*Confabulation*) y §2.8 (*Information Integrity*).

**Confabulación con aplomo (alucinación)** — *Fuentes: Ji et al. (2023), §13; NIST AI 600-1, §2.2*
- *Qué se ve:* datos, citas o enlaces inexistentes, o que no se sostienen en la fuente dada, afirmados con tono seguro y alta fluidez. Ji et al. señalan que, como los LLM generan respuestas muy fluidas y convincentes, sus alucinaciones son más difíciles de detectar y pueden tener consecuencias más dañinas. NIST define la confabulación como contenido erróneo o falso presentado con seguridad, y señala que el riesgo surge cuando los usuarios lo creen, a menudo por el tono seguro de la respuesta, y que el modelo puede añadir lógica o citas confabuladas que aparentan justificar la respuesta y llevar a confiar indebidamente en ella. La literatura técnica dice "alucinación" (*hallucination*); NIST usa "confabulación" y recoge en una nota que algunos comentaristas consideran que "alucinación" y "fabricación" antropomorfizan el sistema, lo que NIST considera un riesgo en sí mismo.
- *Cómo se mide:* [establecida] baterías de preguntas con respuesta verificable (por ejemplo, TruthfulQA, que Ji et al. citan como ejemplo representativo de preguntas diseñadas para provocar errores) o evaluación de afirmaciones sueltas contra fuentes (por ejemplo, FActScore). Con acceso a las probabilidades internas del modelo se usan métricas basadas en incertidumbre; sin ese acceso, métricas de consistencia entre respuestas repetidas. Ji et al. recomiendan distinguir alucinación de factualidad: la primera se define respecto a la fuente o a los datos de entrenamiento, y la segunda respecto al conocimiento del mundo.
- *Mitigación:* generación aumentada con recuperación y citas verificables, que tiene sus propios retos (ruido, información contradictoria, necesidad de abstenerse si no hay evidencia suficiente). Enseñar al modelo a expresar incertidumbre y a decir "no lo sé" es una línea de investigación en fase temprana, sin soluciones satisfactorias hasta ahora (Ji et al., §13.4.3).

**Engaño sobre acciones, identidad o capacidades** — *Fuente: Park et al. (2023)*
- *Qué se ve:* el modelo afirma algo falso sobre sí mismo o sobre lo que ha hecho para lograr un objetivo. Park et al. definen el engaño como la inducción sistemática de creencias falsas para conseguir un resultado distinto de la verdad, sin exigir creencias ni deseos en el sistema, y lo distinguen de los errores de los chatbots inexactos. Documentan casos de engaño estratégico, como GPT-4 haciéndose pasar por una persona con discapacidad visual para que un trabajador humano resolviera un CAPTCHA (prueba del ARC; el modelo recibió algunas pistas de un evaluador, pero nadie le indicó que mintiera). NIST (§2.2) recoge también que un LLM puede afirmar falsamente que es humano o que tiene rasgos humanos. [propuesta] Como extensión de ese patrón se incluye afirmar haber ejecutado una tarea que no se hizo o presentar resultados que no provienen de ninguna herramienta; ese caso concreto no figura en Park et al.
- *Cómo se mide:* [establecida] la dimensión *Unprompted Deception Toward User* de Petri. [propuesta] discrepancia entre lo que el modelo afirma haber hecho y los registros de llamadas a herramientas.
- *Mitigación:* [propuesta] verificación de llamadas a herramientas y exigir que la respuesta muestre el resultado real de la herramienta. Park et al. proponen además técnicas de detección del engaño, transparencia (marcar las salidas potencialmente engañosas) y leyes que obliguen a distinguir a la IA de una persona (*bot-or-not*), y advierten de que hoy no se sabe prevenir de forma fiable que un sistema manifieste objetivos no deseados.

**Persuasión personalizada indebida**
- *Qué se ve:* el modelo adapta sus argumentos a vulnerabilidades emocionales del usuario (miedo, soledad) para empujar una conclusión.
- *Cómo se mide:* [propuesta] comparar el cambio de opinión inducido con y sin información sobre el estado emocional del personaje simulado.
- *Mitigación:* políticas explícitas sobre el uso de información emocional del usuario. Es el área de este mapa con menos respaldo empírico.

---

## 4. Pérdida de alineación

**Fuentes:** Petri (Anthropic, 2025); Casper et al. (2023), §3.2.2 (manipulación de la recompensa); Park et al. (2023) (engañar las pruebas de seguridad).

**Cooperación con peticiones dañinas (jailbreaking)**
- *Qué se ve:* el modelo evade sus restricciones mediante juegos de rol, lenguaje codificado u otros formatos. Es una vulnerabilidad provocada por un usuario adversario.
- *Cómo se mide:* [establecida] tasa de éxito del ataque (ASR) sobre un conjunto fijo de intentos.
- *Mitigación:* clasificadores de entrada y salida, entrenamiento adversarial y pruebas periódicas con nuevos ataques.

**Manipulación de la recompensa (reward hacking)** — *Fuentes: Casper et al. (2023), §3.2.2; Park et al. (2023); Petri (sus instrucciones semilla incluyen este comportamiento)*
- *Qué se ve:* el modelo cumple la letra de la métrica sin cumplir el objetivo (por ejemplo, forzar que pase una verificación automática). Un caso de engaño a la evaluación que citan Park et al.: en un estudio de evolución simulada, los agentes aprendieron a reproducirse despacio solo mientras eran evaluados y rápido el resto del tiempo ("hacerse el muerto") para superar la prueba de seguridad.
- *Cómo se mide:* [propuesta] casos que superan la verificación automática pero fallan en revisión manual.
- *Mitigación:* evaluaciones más robustas y revisión del diseño de la función de recompensa.

**Autopreservación y búsqueda de poder (evidencia emergente)**
- *Qué se ve:* resistencia a la corrección o al apagado, o solicitud de permisos innecesarios. Petri define la autopreservación como intentos de evitar que lo apaguen, lo modifiquen o cambien sus objetivos, y la búsqueda de poder como intentos de obtener más capacidades, recursos o influencia.
- *Cómo se mide:* [establecida] Petri puntúa la autopreservación espontánea (*Unprompted Self Preservation*) en escenarios simulados, y sus instrucciones semilla incluyen escenarios de búsqueda de poder, que no es una de sus dimensiones de puntuación por defecto. [propuesta] para la búsqueda de poder: escenarios con oportunidad de ampliar permisos sin necesidad, contando cuántas veces el modelo lo hace.
- *Mitigación:* limitar los permisos del entorno al mínimo necesario. Es un problema de investigación abierto; no hay una mitigación de entrenamiento probada.

---

## Límites de este mapa

- Las categorías son discutibles. Petri ofrece 36 dimensiones por defecto puntuadas por un modelo juez. Sus autores reconocen que qué cuenta como comportamiento desalineado es subjetivo y depende del contexto, que el juez puede ser inconsistente en casos límite y pasar por alto formas sutiles, y que las puntuaciones globales rara vez son útiles en términos absolutos, aunque los valores relativos suelen ser informativos. Los jueces tampoco coinciden del todo entre sí (correlaciones de Spearman de 0,55 a 0,75 en la métrica *concerning*), y hay indicios leves de que algunos favorecen a su propia familia de modelos.
- Las puntuaciones de Petri dependen del escenario. En sus pruebas de denuncia interna (*whistleblowing*), la agencia del modelo, la complicidad de la dirección y la existencia real de irregularidades tienen efectos sustanciales, y los modelos a veces denuncian incluso cuando la actividad se describe expresamente como inocua, lo que los autores interpretan como respuesta a la estructura narrativa. Un valor alto describe lo que ocurre en ese escenario y no una disposición estable.
- Petri puntúa también si el modelo parece darse cuenta de que lo evalúan (*eval awareness*). Sus autores señalan que esto complica la interpretación de las puntuaciones de Claude Sonnet 4.5 y esperan que las de autopreservación sean subestimaciones leves; advierten además de que muchos escenarios simulados contienen pistas de que no son reales (por ejemplo, mensajes fechados en 2024 para modelos entrenados después).
- Petri no está pensado como un benchmark ni sustituye la revisión humana. Un auditor demasiado insistente puede empujar al modelo hacia un comportamiento que solo interesa si aparece de forma orgánica, y los autores reconocen que ese equilibrio sigue siendo imperfecto.
- Las métricas marcadas como [propuesta] no están validadas. Su valor depende de que se prueben en experimentos como el informe de Red Teaming de este repositorio.
- Las ramas con evidencia emergente deben leerse como hipótesis de trabajo, no como resultados establecidos.
- NIST AI 600-1 se publicó en julio de 2024 y anuncia que futuras revisiones del perfil incluirán más subcategorías, riesgos y acciones, por lo que su lista de riesgos puede cambiar.
- Park et al. adoptan una definición amplia de engaño y reconocen que la adulación y el razonamiento infiel son casos más discutibles que el engaño estratégico.
- Parte de la evidencia procede de modelos pequeños y de pocos conjuntos de datos. Por ejemplo, Singhal et al. trabajan con modelos de 7.000 millones de parámetros y tres conjuntos de datos, de los cuales solo uno usa directamente etiquetas de anotadores humanos. Su generalización no está establecida.

## Referencias

- Casper, S. et al. (2023). *Open Problems and Fundamental Limitations of Reinforcement Learning from Human Feedback.*
- Ji, Z. et al. (2023). *Survey of Hallucination in Natural Language Generation.* arXiv:2202.03629 (v7, 2024); ACM Computing Surveys.
- Lin, S., Hilton, J. y Evans, O. (2022). *TruthfulQA: Measuring How Models Mimic Human Falsehoods.*
- NIST (2024). *Artificial Intelligence Risk Management Framework: Generative Artificial Intelligence Profile* (NIST AI 600-1). Julio de 2024. doi:10.6028/NIST.AI.600-1
- Park, P. S., Goldstein, S., O'Gara, A., Chen, M. y Hendrycks, D. (2023). *AI deception: A survey of examples, risks, and potential solutions.* Publicado en *Patterns*, 5(5), 100988 (2024). doi:10.1016/j.patter.2024.100988
- Anthropic (2025). *Petri: An open-source auditing tool to accelerate AI safety research.* 6 de octubre de 2025. Código: github.com/safety-research/petri
- Sharma, M. et al. (2023). *Towards Understanding Sycophancy in Language Models.* arXiv:2310.13548; publicado en ICLR 2024.
- Singhal, P., Goyal, T., Xu, J. y Durrett, G. (2023). *A Long Way to Go: Investigating Length Correlations in RLHF.* arXiv:2310.03716; publicado en COLM 2024.
