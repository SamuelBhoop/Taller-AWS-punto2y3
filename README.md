# Taller AWS — Punto 2: FastAPI en Amazon EC2

Repositorio del despliegue de la aplicación **FastAPI** (`test_docker_fastapi`) en una instancia **Amazon EC2**.

**Repositorio:** https://github.com/SamuelBhoop/Taller-AWS-punto2y3

## Estructura

```
test_docker_fastapi/
├── main.py              # API FastAPI
├── requirements.txt
├── Dockerfile           # Para Docker (opcional / punto Docker)
├── run_container.sh
└── fastapi-ec2.service  # Unidad systemd para arranque automático
```

## Endpoints

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/` | Mensaje de bienvenida |
| GET | `/items/{item_id}` | Consultar ítem por ID |
| POST | `/items/` | Crear ítem (JSON) |
| GET | `/docs` | Swagger UI |

---

## 1. Subir el proyecto a GitHub

Ya está en este repositorio. Si haces cambios locales:

```bash
git add .
git commit -m "Agregar test_docker_fastapi y archivos de despliegue EC2"
git push -u origin main
```

---

## 2. Crear instancia EC2 (consola AWS)

1. Inicia sesión en [AWS Console](https://console.aws.amazon.com/) → **EC2** → **Launch instance**.
2. Configuración sugerida:
   - **Name:** `fastapi-taller-aws`
   - **AMI:** Ubuntu Server 22.04 LTS (64-bit x86)
   - **Instance type:** `t2.micro` o `t3.micro` (capa gratuita si aplica)
   - **Key pair:** crea o selecciona un `.pem` y descárgalo
   - **Network settings:** permite **SSH (22)** y **HTTP personalizado TCP 8000** desde `0.0.0.0/0` (solo para el taller; en producción restringe el origen)
3. **Launch instance**.

**Captura sugerida:** pantalla de configuración de la instancia y del Security Group con puertos 22 y 8000.

---

## 3. Conectarse por SSH

```bash
ssh -i "tu-clave.pem" ubuntu@<IP_PUBLICA_EC2>
```

En Windows (PowerShell), ajusta permisos de la clave si hace falta.

**Captura sugerida:** terminal conectada por SSH a la instancia.

---

## 4. Instalar dependencias en la instancia

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv git

git clone https://github.com/SamuelBhoop/Taller-AWS-punto2y3.git
cd Taller-AWS-punto2y3/test_docker_fastapi

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Probar manualmente:

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

En el navegador: `http://<IP_PUBLICA>:8000/` y `http://<IP_PUBLICA>:8000/docs`

**Captura sugerida:** respuesta JSON en el navegador y/o Swagger.

---

## 5. Servicio systemd (arranque automático)

Edita rutas si clonaste en otra carpeta:

```bash
sudo cp fastapi-ec2.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable fastapi-ec2.service
sudo systemctl start fastapi-ec2.service
sudo systemctl status fastapi-ec2.service
```

Reinicia la instancia y verifica que la API sigue respondiendo:

```bash
sudo reboot
# Tras reconectar:
curl http://localhost:8000/
```

**Capturas sugeridas:**

- `systemctl status fastapi-ec2.service` en verde (active running)
- `curl` o navegador tras el reinicio

---

## 6. Security Group (acceso público)

En EC2 → **Security Groups** → grupo de tu instancia → **Edit inbound rules**:

| Tipo | Puerto | Origen | Uso |
|------|--------|--------|-----|
| SSH | 22 | Tu IP o 0.0.0.0/0 (taller) | Administración |
| Custom TCP | 8000 | 0.0.0.0/0 | API FastAPI |

**Captura sugerida:** reglas de entrada con puerto 8000 abierto.

Prueba desde tu PC:

```
http://<IP_PUBLICA_EC2>:8000/
```

---

## 7. Comandos útiles

```bash
# Ver logs del servicio
sudo journalctl -u fastapi-ec2.service -f

# Reiniciar la API
sudo systemctl restart fastapi-ec2.service

# Detener
sudo systemctl stop fastapi-ec2.service
```

---

## Docker (opcional, mismo proyecto)

```bash
cd test_docker_fastapi
chmod +x run_container.sh
./run_container.sh
```

O:

```bash
docker build -t fastapi-test .
docker run -d -p 8000:8000 --name fastapi fastapi-test
```

---

## Checklist del taller

- [ ] Repositorio en GitHub con `test_docker_fastapi`
- [ ] Instancia EC2 creada
- [ ] Código clonado en la instancia
- [ ] API funcionando en puerto 8000
- [ ] Capturas de configuración en la instancia
- [ ] Servicio `systemd` habilitado y activo
- [ ] Security Group con puerto 8000
- [ ] Capturas de acceso por IP pública
