# Selene

<img width="1536" height="1024" alt="Selene-CLI_logo_2026_2_1" src="https://github.com/user-attachments/assets/9322847f-5950-45d7-bfa1-a2b3c6f393d7" />


# Selene-cli — Emulador de terminal con IA (Experimental, usa **g4f**) 🌙🤖

```
   _____      _                  
  / ____|    | |                 
 | (___   ___| | ___ _ __   ___  
  \___ \ / _ \ |/ _ \ '_ \ / _ \ 
  ____) |  __/ |  __/ | | |  __/ 
 |_____/ \___|_|\___|_| |_|\___| 
                                 
                                 
```

> **Selene-cli** es un emulador de terminal que integra capacidades de conversación y asistencia de IA mediante **g4f**.
> **Synapse** es la capa de *control y orquestación* que decide si una sugerencia de la IA debe ejecutarse en el sistema.
> Transparencia: Selene **no es** la IA — Selene usa g4f y Synapse controla la ejecución. 🔐

---

## ✅ Resumen rápido

* Proyecto: `selene-cli`
* Comando sugerido: `selene`
* Backend IA por defecto: **g4f** (configurable)
* Synapse = capa que inspecciona respuestas de la IA, detecta órdenes y aplica políticas (allow/confirm/deny).
* Modos: `safe`, `audit`, `full` (por defecto `safe`).
* Enfoque: **seguridad primero** — la IA puede sugerir, Synapse decide.

---

## Características principales ✨

* Interfaz REPL tipo terminal
* Integración con backends IA (g4f por defecto; pluginable)
* Synapse: detección de intención, parsing de comandos, políticas de control y confirmación

---

## Arquitectura (alto nivel)

```
[Usuario] 
   ↓
[Selene-cli REPL]  <——— UI/entrada
   ↓
[Synapse]  — parser, policies, contexto, decision
   ↓
( si Synapse permite ) 
   ↓
[Executor seguro] → [Sistema operativo]
   ↑
( logs, salida )
   ↑
[g4f / proveedor IA] ← Synapse solicita respuestas / generación
```

---

## Instalación (ejemplo)

> Recomendación:) instalar en VM/container de pruebas antes de ejecutar en un host de producción.

```bash
# clonar repo
git clone https://github.com/<tu-usuario>/selene-cli.git
cd selene-cli

# crear virtualenv (recomendado)
python -m venv .venv
source .venv/bin/activate

# instalar dependencias
pip install -r requirements.txt

# ejecutar
python -m selene
# o instalar con pip
pip install .
selene
```

`requirements.txt` mínimo sugerido:

```
requests
pyjwt
python-dotenv
# opcionales según proveedores/funcionalidades
```

---

## Synapse — la pieza clave (explicación técnica)

Synapse es responsable de:

1. Analizar la respuesta textual de la IA y detectar *intenciones de ejecución* (ej.: `run: ls -la /var/log` o texto libre con comandos embebidos).
2. Parsear posibles comandos (regex + heurísticas + parser robusto).
3. Clasificar la acción con la política `allow/confirm/deny`.
4. Gestionar confirmaciones interactivas cuando se requiere.
5. Mantener contexto (historial, variables de entorno, estado de sesión).
6. Registrar todo en el *audit log*.

---

## Código ejemplo: g4f wrapper (skeleton)

> No incluye llamadas a providers concretos; g4f es un agregador. Este wrapper es conceptual.

```python
# selene/providers/g4f_provider.py
import time

class G4FProvider:
    def __init__(self, options: dict):
        self.options = options

    def ask(self, prompt: str, context: dict = None) -> str:
        """
        Envía prompt a g4f y devuelve texto generado.
        Aquí deberías implementar el cliente `g4f` real o la llamada HTTP
        al endpoint que uses. Este es un stub.
        """
        # Ejemplo: si usas la librería 'g4f', aquí harías:
        # import g4f
        # return g4f.ChatCompletion.create(...)

        # Stub: respuesta dummy
        time.sleep(0.2)
        return "Ejemplo: `ls -la /var/log`"
```

---

## Ejemplo de flujo (usuario → ejecución)

1. Usuario escribe: `ayúdame a ver los logs de nginx`
2. Selene envía prompt a g4f (incluyendo contexto/historial)
3. g4f responde: `Puedes ejecutar: \`tail -n 200 /var/log/nginx/error.log``
4. Synapse (que es el motor interno de detección de comandos) analiza la respuesta: detecta `tail` → `allow` (si está en allow_list)
5. Si necesita confirmación, Selene pregunta: “Esto ejecutará `tail -n 200 ...`. ¿Confirmas? (y/n)”
6. Si usuario confirma → Executor ejecuta de forma segura → Salida se muestra y se registra.

---

## Privacidad y uso de g4f 🛡️

* **Lo que envías a Selene puede salir de tu control** según el proveedor; no enviar contraseñas, claves privadas ni datos sensibles en bruto.
```

---

## Prompting / Contexto (cómo enviar prompts a g4f)

* Incluye siempre: versión de Selene, modo actual, resumen de las últimas N interacciones (no demasiado largas).
* Pide a la IA que **identifique claramente** los comandos propuestos dentro de bloques de código (`` `comando` ``) para facilitar el parsing.
* Ejemplo de prompt:

```
Eres un asistente de sistema. Si propones un comando, ponlo entre backticks. 
Contexto: modo=safe, último comando de usuario="ver logs nginx".
Pregunta: ¿qué comando ejecutarías para ver los últimos 200 errores del nginx?
```

---

## Desarrollo y tests 🧪

* Tests unitarios para Synapse (clasificación), executor (simulaciones), provider (mocks).
* Integración: testear en contenedor con filesystem limitado.
* CI: ejecutar linters (flake8), tests y analizar cobertura.

---

## Troubleshooting 📋

* **La IA sugiere comandos no detectados**: mejora heurística de `detect_commands` (usar parsers más complejos, dependencias NLP).
* **Comandos no se ejecutan**: revisar `allow_list`, modo actual y permisos de usuario.
* **Salida truncada**: controlar buffers y tiempo de ejecución; usar `PAGER` o paginar manualmente.
* **Problemas con g4f**: revisa opciones del provider y proxy de red (g4f depende de proveedores).

---

## Contribuir 🤝

1. Fork y branch.
2. PR con descripción clara.
3. Tests que cubran cambios.

---

## Licencia

GNU GPLv3

---

## FAQ rápido ❓

**P:** ¿Puedo ejecutar scripts completos?
**R:** Sí, pero Synapse debería pedir confirmación según políticas; evita ejecutar scripts sin revisión.

**P:** ¿Synapse puede aprender políticas?
**R:** Puede extenderse: almacenar decisiones del usuario y proponer reglas automáticas, pero ten cuidado con la automatización sin supervisión.

**P:** ¿Qué pasa si la IA intenta inyectar un comando peligroso en texto largo?
**R:** Synapse aplica búsqueda de patrones `deny_patterns` y sanitiza. Recomendable tener múltiples capas: regex, heurísticas y validación por lista blanca.

```
# Selene-cli 🌙
**Emulador de terminal asistido por IA (experimental) — Synapse controla la ejecución.**
```

---
