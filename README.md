# Reinforcement Learning con DQN en ROS2 Flatland

## Descripción
Implementación de un algoritmo de Reinforcement Learning diferente a PPO para entrenar
al robot SERP a navegar un pasillo con giro de 90° usando ROS2 y el simulador Flatland.

## Algoritmo seleccionado: DQN (Deep Q-Network)

Se seleccionó **DQN** en lugar de PPO por las siguientes razones técnicas:

- El entorno tiene un **espacio de acción discreto** (3 acciones: avanzar, girar izquierda,
  girar derecha). SAC y TD3 no son compatibles con acciones discretas, por lo que DQN
  es la elección correcta.
- DQN usa **exploración epsilon-greedy**: al inicio toma acciones casi aleatorias
  (`exploration_rate ≈ 0.65`) y gradualmente las reduce (`exploration_rate → 0.05`)
  a medida que aprende, lo que permite una exploración más balanceada que A2C.
- DQN es **off-policy** y usa un **replay buffer**, lo que lo hace más sample-efficient
  que algoritmos on-policy como PPO o A2C.

## Cambio realizado en el código

Archivo modificado: `serp_rl/serp_rl/__init__.py`

**Línea 18** — Importación del algoritmo:
```python
# Original
from stable_baselines3 import PPO

# Modificado
from stable_baselines3 import DQN
```

**Línea 274** — Creación del agente:
```python
# Original
agent = PPO("MlpPolicy", self, verbose=1)

# Modificado
agent = DQN("MlpPolicy", self, verbose=1, learning_starts=500)
```

El parámetro `learning_starts=500` indica que DQN espera acumular 500 experiencias
en el replay buffer antes de comenzar a actualizar la red, lo que estabiliza el
aprendizaje inicial.

## Entorno Gymnasium

- **Entorno**: Robot SERP en Flatland (pasillo con giro de 90°)
- **Espacio de observación**: 9 lecturas del LiDAR (`Box(0, 2, shape=(9,))`)
- **Espacio de acción**: 3 acciones discretas (`Discrete(3)`)
  - 0: avanzar
  - 1: girar izquierda
  - 2: girar derecha
- **Recompensas**:
  - Colisión: -200
  - Timeout (200 pasos): -300
  - Llegar al objetivo: +400 + pasos restantes
  - Avanzar: +2

## Estrategia de entrenamiento

El código entrena en bloques de **5,000 steps** y evalúa el modelo con 20 episodios
de prueba. El entrenamiento continúa hasta alcanzar una precisión del **80%**
(16/20 episodios exitosos).

## Evidencia de resultados

El modelo fue entrenado hasta alcanzar el **80% de precisión**.

Métricas observadas durante el entrenamiento con DQN:

| Iteración | ep_rew_mean | exploration_rate | n_updates |
|-----------|-------------|------------------|-----------|
| Inicio    | -157        | 0.65             | 0         |
| 500 steps | -150        | 0.05             | 7         |
| 1000 steps| -128        | 0.05             | 67        |
| 5000 steps| -88         | 0.05             | 1069      |

La `exploration_rate` bajó de 0.65 a 0.05 en los primeros 500 steps, indicando que
DQN exploró el entorno activamente antes de comenzar a explotar lo aprendido.

## Cómo ejecutar

```bash
# 1. Clonar e instalar dependencias
pip install stable-baselines3

# 2. Compilar el workspace
cd ~/ros2_ws/ros2_ws
source /opt/ros/humble/setup.bash
colcon build --parallel-workers 1
source install/setup.bash

# 3. Lanzar la simulación
ros2 launch serp_rl serp_rl.launch.py
```

## Dependencias

- ROS2 Humble
- Flatland Simulator
- Stable-Baselines3
- Gymnasium

## Evidencia visual
![Screenshot 1](screenshots/SS_1.png)
![Screenshot 2](screenshots/SS_2.png)
![Screenshot 3](screenshots/SS_3.png)
![Screenshot 4](screenshots/SS_4.png)