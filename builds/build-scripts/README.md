# builds/build-scripts

One directory per build variant.

Each variant directory contains a concise `README.md`, the PBS build script,
the build shell script when applicable, and an `outputs/` directory. Raw PBS
stdout/stderr are authoritative evidence and are never overwritten on retries.