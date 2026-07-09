# Referencia Rápida — SO-101 Dual Arm

## Comandos esenciales

```bash
# Activar entorno
conda activate lerobot && cd ~/lerobot

# Encontrar puertos
lerobot-find-port

# ── CALIBRAR ─────────────────────────────────────────
# Leader A
lerobot-calibrate --teleop.type=so101_leader --teleop.id=A --teleop.port=/dev/ttyACM0

# Follower A
lerobot-calibrate --robot.type=so101_follower --robot.id=A --robot.port=/dev/ttyACM1

# Leader B
lerobot-calibrate --teleop.type=so101_leader --teleop.id=B --teleop.port=/dev/ttyACM3

# Follower B
lerobot-calibrate --robot.type=so101_follower --robot.id=B --robot.port=/dev/ttyACM2

# ── TELEOPERAR ───────────────────────────────────────
# Brazo A
lerobot-teleoperate \
  --teleop.type=so101_leader --teleop.id=A --teleop.port=/dev/ttyACM0 \
  --robot.type=so101_follower --robot.id=A --robot.port=/dev/ttyACM1

# Brazo B
lerobot-teleoperate \
  --teleop.type=so101_leader --teleop.id=B --teleop.port=/dev/ttyACM3 \
  --robot.type=so101_follower --robot.id=B --robot.port=/dev/ttyACM2

# ── CALIBRACIONES ────────────────────────────────────
# Ver
cat ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/A.json

# Editar
nano ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/B.json

# Backup
mkdir -p ~/lerobot_calibration_backup
cp ~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/*.json ~/lerobot_calibration_backup/
cp ~/.cache/huggingface/lerobot/calibration/robots/so101_follower/*.json ~/lerobot_calibration_backup/
```

## Checklist antes de cada sesión

- [ ] `conda activate lerobot`
- [ ] Verificar puertos con `lerobot-find-port`
- [ ] Confirmar que `A.json` y `B.json` existen en las rutas de calibración
- [ ] Probar teleoperación de cada brazo por separado antes de dual arm
