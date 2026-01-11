# gym-donkeycar Examples

This directory contains example code for using the gym-donkeycar Gymnasium environments.

## Prerequisites

### Basic Requirements
```bash
# Install gym-donkeycar
pip install -e ..

# Or install from repository
pip install git+https://github.com/tawnkramer/gym-donkeycar
```

### Simulator Setup

1. **Download the simulator** from: https://github.com/tawnkramer/gym-donkeycar/releases
2. **Launch the simulator** before running examples (recommended), OR
3. **Auto-launch**: Provide the simulator path with `--sim /path/to/simulator`

**Note**: For manual launch, use `--sim manual` in the example commands below.

## Examples Overview

### 1. Basic Testing: `gym_test.py`

A simple script to test the environment with random actions.

**Usage:**
```bash
# With simulator already running
python gym_test.py --sim manual --env_name donkey-generated-track-v0

# Test a specific environment
python gym_test.py --sim manual --env_name donkey-warehouse-v0 --port 9091

# Test all environments sequentially
python gym_test.py --sim manual --env_name all
```

**What it does:**
- Connects to the simulator
- Runs 3 episodes with random actions
- Prints episode completion info
- Tests LIDAR sensor configuration

### 2. Camera Configuration Test: `test_cam_config.py`

Tests custom camera resolution settings.

**Usage:**
```bash
python test_cam_config.py --sim manual --env_name donkey-warehouse-v0 --port 9091
```

**What it does:**
- Configures camera to 256x256x3 resolution
- Verifies the camera size matches configuration
- Runs 100 steps with straight driving

### 3. Reinforcement Learning Examples

#### PPO Training: `reinforcement_learning/ppo_train.py`

Uses [Stable-Baselines3](https://github.com/DLR-RM/stable-baselines3) for PPO training.

**Additional Dependencies:**
```bash
pip install stable-baselines3[extra]
```

**Usage:**
```bash
# Training
python reinforcement_learning/ppo_train.py --sim manual --env_name donkey-warehouse-v0

# Testing trained model
python reinforcement_learning/ppo_train.py --sim manual --env_name donkey-warehouse-v0 --test
```

**Notes:**
- Saves model as `ppo_donkey.zip`
- Training runs for 10,000 timesteps by default
- Use `--multi` for parallel environments (requires simulator path)

#### DDQN Training: `reinforcement_learning/ddqn.py`

Deep Double Q-Learning implementation.

**Additional Dependencies:**
```bash
pip install tensorflow keras numpy
```

**Usage:**
```bash
python reinforcement_learning/ddqn.py --sim manual --env_name donkey-warehouse-v0
```

**Notes:**
- Saves model checkpoints during training
- Uses replay buffer for experience replay
- Configurable learning rate, gamma, epsilon decay

### 4. Genetic Algorithm: `genetic_alg/simple_gen_driver.py`

Genetic algorithm for evolving neural network controllers.

**Additional Dependencies:**
```bash
pip install tensorflow==1.15  # Requires TensorFlow 1.x
pip install keras
```

**⚠️ Warning:** This example uses deprecated TensorFlow 1.x API and may not work with TensorFlow 2.x or Python 3.10+.

**Usage:**
```bash
python genetic_alg/simple_gen_driver.py \
    --in_model /path/to/model.h5 \
    --out_model evolved_model.h5 \
    --num_agents 8 \
    --num_iter 10 \
    --env_name donkey-warehouse-v0
```

**Known Issues:**
- Requires TensorFlow 1.x (`tf.Session`, `K.set_session`)
- Not compatible with Python 3.12+
- May require downgrading to Python 3.8 or 3.9

### 5. Supervised Learning: `supervised_learning/`

Training models from recorded driving data.

**Files:**
- `train.py`: Train a neural network from recorded data
- `evaluate.py`: Evaluate trained model performance
- `models.py`: Model architectures
- `conf.py`: Configuration settings

**Additional Dependencies:**
```bash
pip install tensorflow keras opencv-python pillow
```

**Usage:**
```bash
# Train from recorded data
python supervised_learning/train.py --data /path/to/recorded/data --model mymodel.h5

# Evaluate model
python supervised_learning/evaluate.py --model mymodel.h5 --sim manual
```

## Important Notes for Gymnasium Migration

All examples have been updated to use the Gymnasium API:

### Key Changes from OpenAI Gym

1. **Import statement:**
   ```python
   import gymnasium as gym  # Not "import gym"
   ```

2. **step() returns 5 values:**
   ```python
   obs, reward, terminated, truncated, info = env.step(action)
   done = terminated or truncated  # If needed
   ```

3. **reset() returns 2 values:**
   ```python
   obs, info = env.reset()  # Not just "obs"
   ```

4. **Accessing custom environment attributes:**
   ```python
   env.unwrapped.viewer.exit_scene()  # Use env.unwrapped
   env.unwrapped.viewer.set_reward_fn(custom_fn)
   env.unwrapped.viewer.set_episode_over_fn(custom_fn)
   ```

## Available Environments

All examples support these environment names:
- `donkey-warehouse-v0`
- `donkey-generated-roads-v0`
- `donkey-avc-sparkfun-v0`
- `donkey-generated-track-v0`
- `donkey-roboracingleague-track-v0`
- `donkey-waveshare-v0`
- `donkey-minimonaco-track-v0`
- `donkey-warren-track-v0`
- `donkey-thunderhill-track-v0`
- `donkey-circuit-launch-track-v0`

## Troubleshooting

### "fixture 'mocker' not found"
Install test dependencies:
```bash
pip install pytest-mock pytest-cov
```

### "Can't connect to server"
- Make sure the simulator is running on the correct port (default 9091)
- Check that no firewall is blocking the connection
- Verify the port with `--port 9091`

### "AttributeError: 'OrderEnforcing' object has no attribute 'viewer'"
- This means the code hasn't been updated for Gymnasium wrappers
- Use `env.unwrapped.viewer` instead of `env.viewer`
- All examples in this directory have been updated

### TensorFlow Compatibility Issues
- `genetic_alg/simple_gen_driver.py` requires TensorFlow 1.x
- Use Python 3.8 or 3.9 for TensorFlow 1.x compatibility
- Consider updating to TensorFlow 2.x (requires code changes)

### Float Precision Warnings
These warnings are expected with Gymnasium:
```
WARN: Box low's precision lowered by casting to float32
```
They don't affect functionality and can be ignored.

## Performance Tips

1. **Reduce start_delay**: Default is 5 seconds, can be reduced to 1-2 seconds:
   ```python
   conf = {"start_delay": 1, ...}
   ```

2. **Adjust max_cte**: Reduce to end episodes faster when going off-track:
   ```python
   conf = {"max_cte": 5, ...}  # Default is 8.0
   ```

3. **Frame skipping**: Skip frames to speed up training:
   ```python
   conf = {"frame_skip": 2, ...}  # Process every 2nd frame
   ```

## Contributing

When adding new examples:
1. Use the Gymnasium API (not OpenAI Gym)
2. Use `env.unwrapped` to access custom attributes
3. Include docstrings explaining the example
4. Add installation requirements to this README
5. Test with a running simulator

## References

- [Gymnasium Documentation](https://gymnasium.farama.org/)
- [Stable-Baselines3](https://stable-baselines3.readthedocs.io/)
- [Donkey Car Project](https://www.donkeycar.com/)
- [Migration Guide](../tests/UPDATE_GYM2GYMNASIUM.md)
