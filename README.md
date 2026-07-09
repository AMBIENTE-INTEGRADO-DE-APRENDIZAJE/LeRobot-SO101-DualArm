# 🦾 LeRobot SO-101 — Configuración Dual Arm (Leader/Follower A y B)

**Departamento de Ingeniería Industrial | Universidad de los Andes**\
**Ambiente Integrado de Aprendizaje (AIA)**

Guía completa para configurar y operar dos pares de brazos robóticos SO-101 con [LeRobot](https://github.com/huggingface/lerobot) de Hugging Face. Cubre desde la identificación de puertos hasta la teleoperación simultánea de ambos brazos.

---

## Requisitos previos

- Ubuntu con Conda instalado.
- Dos pares de brazos SO-101 (Leader A + Follower A, Leader B + Follower B).
- 4 puertos USB disponibles.
- LeRobot instalado (`pip install lerobot` o desde el repo oficial).

## 0. Activar el entorno

```bash
conda activate lerobot
cd ~/lerobot
```

## 1. Identificar los puertos USB

Cada brazo se conecta a un puerto serial distinto. Para identificarlos:

```bash
lerobot-find-port
```

El programa pedirá desconectar un dispositivo USB, detectará cuál desapareció y mostrará su puerto. Repetir para cada brazo.

Ejemplo de asignación (puede variar en cada conexión):

| Dispositivo | Puerto |
|---|---|
| Leader A | `/dev/ttyACM0` |
| Follower A | `/dev/ttyACM1` |
| Leader B | `/dev/ttyACM3` |
| Follower B | `/dev/ttyACM2` |

> ⚠️ **Los puertos cambian al reconectar los cables USB.** Siempre verificar con `lerobot-find-port` antes de cada sesión.

## 2. Calibración

Cada brazo necesita su propia calibración almacenada en un archivo JSON independiente (A o B). Nunca usar `None.json`.

### Dónde se guardan las calibraciones

```
~/.cache/huggingface/lerobot/calibration/
├── teleoperators/so101_leader/
│   ├── A.json
│   └── B.json
└── robots/so101_follower/
    ├── A.json
    └── B.json
```

### Procedimiento de calibración

Durante la calibración el programa pide dos cosas:

1. Llevar todas las articulaciones aproximadamente a la **mitad de su recorrido** → presionar `ENTER`.
2. Mover cada articulación por **todo su rango completo** → presionar `ENTER`.

### Calibrar Leader A

```bash
lerobot-calibrate \
  --teleop.type=so101_leader \
  --teleop.id=A \
  --teleop.port=/dev/ttyACM0
```

### Calibrar Follower A

```bash
lerobot-calibrate \
  --robot.type=so101_follower \
  --robot.id=A \
  --robot.port=/dev/ttyACM1
```

### Calibrar Leader B

```bash
lerobot-calibrate \
  --teleop.type=so101_leader \
  --teleop.id=B \
  --teleop.port=/dev/ttyACM3
```

### Calibrar Follower B

```bash
lerobot-calibrate \
  --robot.type=so101_follower \
  --robot.id=B \
  --robot.port=/dev/ttyACM2
```

## 3. Teleoperación

### Brazo A

```bash
lerobot-teleoperate \
  --teleop.type=so101_leader \
  --teleop.id=A \
  --teleop.port=/dev/ttyACM0 \
  --robot.type=so101_follower \
  --robot.id=A \
  --robot.port=/dev/ttyACM1
```

### Brazo B

```bash
lerobot-teleoperate \
  --teleop.type=so101_leader \
  --teleop.id=B \
  --teleop.port=/dev/ttyACM3 \
  --robot.type=so101_follower \
  --robot.id=B \
  --robot.port=/dev/ttyACM2
```

### Operación simultánea (dual arm)

Abrir **dos terminales** y ejecutar un brazo en cada una:

```
Terminal 1 → Brazo A (comandos de arriba)
Terminal 2 → Brazo B (comandos de arriba)
```

## 4. Gestión de calibraciones

### Consultar una calibración

```bash
# Followers
cat ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/A.json
cat ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/B.json

# Leaders
cat ~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/A.json
cat ~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/B.json
```

### Editar una calibración manualmente

```bash
nano ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/B.json
```

Ejemplo de parámetros editables:

```json
"wrist_roll": {
    "id": 5,
    "drive_mode": 0,
    "homing_offset": -100,
    "range_min": 0,
    "range_max": 3417
}
```

Guardar con `Ctrl+O` → `ENTER` → `Ctrl+X`.

### Respaldar calibraciones

```bash
mkdir -p ~/lerobot_calibration_backup

cp ~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/A.json \
   ~/lerobot_calibration_backup/leader_A.json
cp ~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/B.json \
   ~/lerobot_calibration_backup/leader_B.json
cp ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/A.json \
   ~/lerobot_calibration_backup/follower_A.json
cp ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/B.json \
   ~/lerobot_calibration_backup/follower_B.json
```

### Restaurar una calibración

```bash
cp ~/lerobot_calibration_backup/follower_A.json \
   ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/A.json
```

## 5. Troubleshooting

### "El sistema pide calibrar de nuevo"

Verificar que se esté usando el ID correcto (`--teleop.id=A` y `--robot.id=A`, no omitirlo). Sin el parámetro `id`, LeRobot busca `None.json` que no existe.

### Overload Error

```
Overload error!
```

Ocurre cuando un servo alcanza un límite mecánico o la calibración no coincide con la posición física. Recalibrar el brazo afectado o revisar el montaje.

### Missing Motor IDs

```
Missing motor IDs: 4
```

Verificar cableado, alimentación y que el puerto USB sea el correcto. Asegurar que todos los motores del daisy-chain responden.

### La garra no cierra completamente

Editar `range_min` en la sección `gripper` del archivo de calibración correspondiente.

### Wrist Flex / Wrist Roll no siguen correctamente

Revisar `homing_offset`, `range_min` y `range_max` en el JSON. Si persiste después de recalibrar, verificar el montaje mecánico del servo o el homing_offset almacenado en la EEPROM del motor.

## 6. Recomendaciones

- **Nunca usar `None.json`** — siempre especificar `--teleop.id=A` o `--robot.id=B`.
- Mantener un archivo de calibración independiente por brazo (`A.json` y `B.json`).
- Hacer backup después de cada calibración exitosa.
- Verificar puertos USB con `lerobot-find-port` antes de cada sesión.
- No recalibrar un brazo que ya funciona bien — la calibración se preserva entre sesiones.

## 📎 Referencias

- [LeRobot — Hugging Face](https://github.com/huggingface/lerobot)
- [SO-101 Assembly Guide](https://github.com/huggingface/lerobot/blob/main/examples/robots/so101/README.md)

## 📄 Licencia

Uso académico — Universidad de los Andes.
