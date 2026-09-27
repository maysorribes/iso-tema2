# ISO · Tema 2 · Virtualización

Apuntes del módulo **Implantación de Sistemas Operativos** (ASIR), hechos con [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Probar en local

```bash
pip install -r requirements.txt
mkdocs serve
```

Abrir http://127.0.0.1:8000

## Estructura

```
mkdocs.yml            configuración y menú
docs/                 contenido (.md)
docs/images/          imágenes de cada apartado
docs/assets/extra.css estilos propios
```

## Añadir un apartado nuevo

1. Crear `docs/iso-ut2.2-nombre.md`
2. Guardar sus imágenes en `docs/images/nombre/`
3. Añadirlo al `nav` de `mkdocs.yml`
