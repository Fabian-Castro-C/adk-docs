¡Gracias por tu interés en contribuir al Agent Development Kit (ADK)! Damos
la bienvenida a contribuciones en los frameworks principales, la documentación y los
componentes relacionados, que se enumeran a continuación.

Esta guía proporciona información sobre cómo involucrarse.

## Preparándose para contribuir

### Elige el repositorio correcto

El proyecto ADK está dividido en varios repositorios. Encuentra el adecuado para
tu contribución:

Repositorio | Descripción | Guía Detallada
--- | --- | ---
[`google/adk-python`](https://github.com/google/adk-python) | Contiene el código fuente de la biblioteca principal de Python | [`CONTRIBUTING.md`](https://github.com/google/adk-python/blob/main/CONTRIBUTING.md)
[`google/adk-python-community`](https://github.com/google/adk-python-community) | Contiene herramientas, integraciones y scripts contribuidos por la comunidad | [`CONTRIBUTING.md`](https://github.com/google/adk-python-community/blob/main/CONTRIBUTING.md)
[`google/adk-js`](https://github.com/google/adk-js) | Contiene el código fuente de la biblioteca principal de JavaScript | [`CONTRIBUTING.md`](https://github.com/google/adk-js/blob/main/CONTRIBUTING.md)
[`google/adk-go`](https://github.com/google/adk-go) | Contiene el código fuente de la biblioteca principal de Go | [`CONTRIBUTING.md`](https://github.com/google/adk-go/blob/main/CONTRIBUTING.md)
[`google/adk-java`](https://github.com/google/adk-java) | Contiene el código fuente de la biblioteca principal de Java | [`CONTRIBUTING.md`](https://github.com/google/adk-java/blob/main/CONTRIBUTING.md)
[`google/adk-docs`](https://github.com/google/adk-docs) | Contiene el código fuente del sitio de documentación que estás leyendo actualmente | [`CONTRIBUTING.md`](https://github.com/google/adk-docs/blob/main/CONTRIBUTING.md)
[`google/adk-samples`](https://github.com/google/adk-samples) | Contiene agentes de ejemplo para ADK | [`CONTRIBUTING.md`](https://github.com/google/adk-samples/blob/main/CONTRIBUTING.md)
[`google/adk-web`](https://github.com/google/adk-web) | Contiene el código fuente para la interfaz de desarrollo `adk web` |

Estos repositorios típicamente incluyen un archivo `CONTRIBUTING.md` en la raíz del
repositorio con información más detallada sobre requisitos, pruebas, procesos
de revisión de código, etc. para ese componente en particular.

### Firma un CLA

Las contribuciones a este proyecto deben estar acompañadas por un
[Acuerdo de Licencia de Contribuidor](https://cla.developers.google.com/about) (CLA).
Tú (o tu empleador) retienes los derechos de autor de tu contribución; esto simplemente
nos da permiso para usar y redistribuir tus contribuciones como parte del
proyecto.

Si tú o tu empleador actual ya han firmado el CLA de Google (incluso si fue
para un proyecto diferente), probablemente no necesites hacerlo de nuevo.

Visita <https://cla.developers.google.com/> para ver tus acuerdos actuales o para
firmar uno nuevo.

### Revisa las pautas de la comunidad

Este proyecto sigue las
[Pautas de Comunidad de Código Abierto de Google](https://opensource.google/conduct/).

## Únete a la discusión

¿Tienes preguntas, quieres compartir ideas o discutir cómo estás usando ADK? Dirígete a
nuestras discusiones de **[Python](https://github.com/google/adk-python/discussions)**,
**[TypeScript](https://github.com/google/adk-js/discussions)**,
**[Go](https://github.com/google/adk-go/discussions)**, o
**[Java](https://github.com/google/adk-java/discussions)**!

Este es el lugar principal para:

* Hacer preguntas y obtener ayuda de la comunidad y los mantenedores.
* Compartir tus proyectos o casos de uso (`Show and Tell`).
* Discutir posibles características o mejoras antes de crear un issue formal.
* Conversación general sobre ADK.

## Cómo contribuir

Hay varias formas en las que puedes contribuir a ADK:

### Reportar problemas { #reporting-issues-bugs-errors }

Si encuentras un error en el framework o un error en la documentación:

* **Errores del Framework:** Abre un issue en [`google/adk-python`](https://github.com/google/adk-python/issues/new),
[`google/adk-js`](https://github.com/google/adk-js/issues/new),
[`google/adk-go`](https://github.com/google/adk-go/issues/new), o
[`google/adk-java`](https://github.com/google/adk-java/issues/new)
* **Errores de Documentación:** [Abre un issue en `google/adk-docs` (usa la plantilla de bug)](https://github.com/google/adk-docs/issues/new?template=bug_report.md)

### Sugerir mejoras { #suggesting-enhancements }

¿Tienes una idea para una nueva característica o una mejora a una existente?

* **Mejoras del Framework:** Abre un issue en [`google/adk-python`](https://github.com/google/adk-python/issues/new),
[`google/adk-js`](https://github.com/google/adk-js/issues/new),
[`google/adk-go`](https://github.com/google/adk-go/issues/new), o
[`google/adk-java`](https://github.com/google/adk-java/issues/new)
* **Mejoras de Documentación:** [Abre un issue en `google/adk-docs`](https://github.com/google/adk-docs/issues/new)

### Mejorar la documentación { #improving-documentation }

¿Encontraste un error tipográfico, una explicación poco clara o información faltante? Envía tus cambios directamente:

* **Cómo:** Envía un Pull Request (PR) con tus mejoras sugeridas.
* **Dónde:** [Crea un Pull Request en `google/adk-docs`](https://github.com/google/adk-docs/pulls)

### Escribir código { #writing-code }

Ayuda a corregir errores, implementar nuevas características o contribuir con ejemplos de código para la documentación:

**Cómo:** Envía un Pull Request (PR) con tus cambios de código.

* **Framework de Python:** [Crea un Pull Request en `google/adk-python`](https://github.com/google/adk-python/pulls)
* **Framework de TypeScript:** [Crea un Pull Request en `google/adk-js`](https://github.com/google/adk-js/pulls)
* **Framework de Go:** [Crea un Pull Request en `google/adk-go`](https://github.com/google/adk-go/pulls)
* **Framework de Java:** [Crea un Pull Request en `google/adk-java`](https://github.com/google/adk-java/pulls)
* **Documentación:** [Crea un Pull Request en `google/adk-docs`](https://github.com/google/adk-docs/pulls)

### Revisiones de código

* Todas las contribuciones, incluyendo las de los miembros del proyecto, pasan por un proceso
  de revisión.

* Usamos Pull Requests (PRs) de GitHub para el envío y revisión de código. Por favor
  asegúrate de que tu PR describa claramente los cambios que estás haciendo.

## Licencia

Al contribuir, aceptas que tus contribuciones estarán licenciadas bajo la
[Licencia Apache 2.0](https://github.com/google/adk-docs/blob/main/LICENSE) del
proyecto.

## ¿Preguntas?

Si te atascas o tienes preguntas, no dudes en abrir un issue en el
rastreador de issues del repositorio relevante.