# Secure Tool Runtime

Un entorno de ejecucion seguro (sandbox) para ejecutar codigo interpretado en Python de forma aislada y controlada. Este modulo permite dar capacidades de ejecucion de codigo e interaccion con APIs a agentes autonomos (Tool-calling) de forma segura, previniendo ataques de inyeccion de comandos, accesos no autorizados al sistema de archivos local y denegaciones de servicio por bucles infinitos.

## Arquitectura de Seguridad Multipaso

La proteccion del entorno de sandbox se articula en tres niveles:

### 1. Validacion Estatica del Codigo (AST Scanner):
*   Antes de ejecutar el script, el codigo se parsea en un Arbol de Sintaxis Abstracta (AST) usando la libreria nativa `ast` de Python.
*   Un `NodeVisitor` personalizado recorre todos los nodos del arbol levantando excepciones de seguridad instantaneas (`SecurityValidationError`) si detecta importaciones prohibidas (`os`, `sys`, `subprocess`, `socket`) o llamadas a funciones builtin bloqueadas (`open`, `eval`, `exec`, `globals`).

### 2. Aislamiento de Ejecucion (Process Sandboxing):
*   El script validado se ejecuta en un proceso de sistema independiente utilizando la libreria `multiprocessing`.
*   Se redefinen los flujos de salida `sys.stdout` y `sys.stderr` y se limpia el ambito global restringiendo la tabla de `__builtins__` a funciones matematicas e iteradores basicos inocuos, impidiendo el acceso a descriptores de archivo heredados.

### 3. Limites de Recursos (Timeout Enforcement):
*   El proceso hijo se monitoriza desde el hilo principal con un limite de tiempo estricto (por defecto 2.0 segundos).
*   Al expirar el timeout, el proceso se termina forzosamente (`process.terminate()`), impidiendo ataques de CPU y garantizando que bucles infinitos no saturen el servidor.

## Requisitos e Instalacion

*   Python 3.8 o superior
*   Pydantic

Para instalar las dependencias locales, ejecute:

```bash
pip install -r requirements.txt
```

## Pruebas y Verificacion

1.  **Ejecutar Pruebas de Validacion y Ejecucion:**
    ```bash
    python -m unittest test_runtime.py
    ```
2.  **Ejecutar Demostracion Practica:**
    ```bash
    python example.py
    ```
    El script demostrara la ejecucion exitosa de calculos matematicos, el bloqueo estatico ante intentos de lectura de `/etc/passwd` y la terminacion limpia de un script con bucle infinito mediante alarma de timeout.
