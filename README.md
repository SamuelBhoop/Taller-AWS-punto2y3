# Taller AWS — Punto 2

API REST sencilla con **FastAPI**, desplegada en una instancia **EC2** (Ubuntu) y empaquetada con **Docker**. El objetivo del taller es publicar un servicio accesible por HTTP y documentarlo con Swagger.

## Estructura del repositorio

```
test_docker_fastapi/
  main.py              # Aplicación FastAPI
  requirements.txt     # Dependencias Python
  Dockerfile           # Imagen para ejecutar la API en contenedor
  run_container.sh     # Build + run local del contenedor
  fastapi-ec2.service  # Unidad systemd para mantener uvicorn en EC2
```

## Funcionamiento de la API

La aplicación expone estos endpoints:

| Método | Ruta | Descripción |
|--------|------|-------------|
| `GET` | `/` | Mensaje de bienvenida (`Universidad EIA`) |
| `GET` | `/items/{item_id}` | Devuelve un ítem por ID y un parámetro opcional `query` |
| `POST` | `/items/` | Crea un ítem enviando JSON (`name`, `description`, `price`) |

Los modelos de entrada se validan con **Pydantic** (`Item`). La documentación interactiva está en:

- Swagger UI: `http://<host>:8000/docs`
- ReDoc: `http://<host>:8000/redoc`

## Flujo en EC2 (sin Docker)

1. Clonar el repositorio en la instancia.
2. Crear un entorno virtual e instalar dependencias:

   ```bash
   cd test_docker_fastapi
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. Probar en primer plano:

   ```bash
   uvicorn main:app --host 0.0.0.0 --port 8000
   ```

4. Para dejar el servicio permanente, copiar `fastapi-ec2.service` a systemd, habilitar e iniciar:

   ```bash
   sudo cp fastapi-ec2.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable fastapi-ec2
   sudo systemctl start fastapi-ec2
   ```

5. En el **security group** de la instancia, abrir el puerto **TCP 8000** (además del 22 para SSH).

## Flujo con Docker

Desde `test_docker_fastapi`:

```bash
docker build -t fastapi-test .
docker run -p 8000:8000 fastapi-test
```

O usar el script:

```bash
chmod +x run_container.sh
./run_container.sh
```

El `Dockerfile` usa `python:3.11-slim`, instala `requirements.txt` y arranca **uvicorn** escuchando en `0.0.0.0:8000`.

## Arquitectura resumida

```
Cliente (navegador / curl)
        │
        ▼  HTTP :8000
   EC2 (Ubuntu)
        │
        ├── uvicorn + FastAPI  (systemd o manual)
        └── opcional: contenedor Docker (misma API)
```

## Requisitos

- Instancia EC2 con acceso SSH
- Python 3.11+ (en EC2) o Docker, según el modo de despliegue
- Puerto 8000 permitido en el security group

## Repositorio relacionado

El **punto 3** del taller (S3, RDS, ECR, Lambda) está en un repositorio aparte: `repo-taller-parte3`.
