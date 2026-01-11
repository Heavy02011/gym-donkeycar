# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

gym-donkeycar provides Gymnasium (formerly OpenAI Gym) environments for the Donkey Car simulator. It enables reinforcement learning and autonomous driving research by connecting Python code to a Unity-based simulator via TCP socket communication.

**Note**: This codebase recently migrated from OpenAI Gym to Gymnasium (v0.29.0+). See `tests/UPDATE_GYM2GYMNASIUM.md` for complete migration details.

## Development Commands

### Installation
```bash
pip install -e .                    # Install in editable mode
pip install -e .[tests]            # Install with test dependencies
pip install -e .[docs]             # Install with documentation dependencies
```

**Note**: If you encounter `fixture 'mocker' not found` errors when running tests, install test dependencies manually:
```bash
pip install pytest-mock pytest-cov pytest-env pytest-xdist ruff isort black pytype
```

### Testing
```bash
make test                          # Run tests with pytest
pytest -v tests/                   # Run tests with verbose output
pytest tests/test_gym_donkeycar.py # Run a specific test file
pytest tests/core/test_fps.py      # Run a specific test
```

### Code Quality
```bash
make format                        # Format code with black and isort
make check-codestyle              # Check code formatting (black + isort)
make lint                         # Lint with ruff
make type                         # Type check with pytype
make commit-checks                # Run format, type, and lint (run before committing)
```

### Documentation
```bash
make docs                         # Build Sphinx documentation
```

## Architecture

### Core Communication Layer (`gym_donkeycar/core/`)

The simulator communication uses a multi-threaded TCP socket architecture:

- **`client.py` (SDClient)**: Base TCP client with threaded message handling. Uses `select()` for non-blocking socket I/O. The `proc_msg()` method runs in a separate thread to handle bidirectional message flow.

- **`sim_client.py` (SimClient)**: Wraps SDClient to handle JSON message serialization/deserialization. Connects a message handler (IMesgHandler) to the socket.

- **`message.py` (IMesgHandler)**: Interface defining the message handler protocol (`on_connect`, `on_disconnect`, `on_recv_message`).

- **`donkey_sim.py` (DonkeyUnitySimHandler)**: Main handler implementing IMesgHandler. Routes incoming messages (telemetry, scene selection, car config) to appropriate handlers via the `fns` dictionary. Manages simulator state including images, sensor data (position, velocity, gyro, accel, lidar), and episode termination logic.

**Key Flow**:
1. `DonkeyEnv` creates `DonkeyUnitySimContoller`
2. Controller creates `SimClient` with `DonkeyUnitySimHandler`
3. SimClient spawns background thread for socket I/O
4. Handler processes telemetry messages and updates observation state
5. `env.step()` calls `take_action()` -> sends control message to simulator -> `observe()` waits for new telemetry

### Environment Layer (`gym_donkeycar/envs/`)

- **`donkey_env.py` (DonkeyEnv)**: Base Gymnasium environment implementing `gym.Env`. Handles action/observation spaces, episode management, and optional subprocess control. All track-specific environments (WarehouseEnv, GeneratedTrackEnv, etc.) inherit from this base class and override the `level` parameter.

- **`donkey_proc.py` (DonkeyUnityProcess)**: Manages the Unity simulator as a subprocess. Launches the executable with specified host/port parameters.

- **`donkey_sim.py` (DonkeyUnitySimContoller)**: High-level controller that wraps the handler and client. Provides the environment interface: `reset()`, `take_action()`, `observe()`, `calc_reward()`.

**Environment Registration**: All environments are registered in `gym_donkeycar/__init__.py` using `gymnasium.envs.registration.register()`.

### Gymnasium API Implementation

**Action Space**: `Box(2,)` - `[steering, throttle]`
- Steering: continuous value (default limit ±1.0)
- Throttle: continuous value (default 0.0 to 1.0)

**Observation Space**: `Box(120, 160, 3)` - RGB camera image by default (configurable)

**`step()` returns**: `observation, reward, terminated, truncated, info`
- `terminated`: Episode ended due to game conditions (collision, out of bounds, etc.)
- `truncated`: Not currently used (always False)
- `info`: Dict with keys: `pos`, `cte`, `speed`, `forward_vel`, `hit`, `gyro`, `accel`, `vel`, `lidar`, `car`, `last_lap_time`, `lap_count`

**`reset()` returns**: `observation, info`
- Sends `reset_car` message to simulator
- Waits 1 second for simulator reset
- Returns initial observation and empty info dict

### Customization Points

Users can inject custom behavior:
- **Reward function**: `env.unwrapped.viewer.set_reward_fn(custom_fn)` - Replace default reward calculation
- **Episode termination**: `env.unwrapped.viewer.set_episode_over_fn(custom_fn)` - Define custom episode end conditions

## Configuration

Environments accept a `conf` dict parameter with keys:
- `exe_path` (str): Path to Unity simulator executable (if auto-launching)
- `port` (int): TCP port (default 9091)
- `host` (str): Host address (default "localhost")
- `level` (str): Scene name - set automatically by environment class
- `max_cte` (float): Max cross-track error before episode termination (default 8.0)
- `cam_resolution` (tuple): Camera resolution as (height, width, channels) (default (120, 160, 3))
- `log_level` (int): Logging level (default logging.INFO)
- `steer_limit` (float): Steering range limit (default 1.0)
- `throttle_min`, `throttle_max` (float): Throttle range (default 0.0 to 1.0)
- `frame_skip` (int): Number of frames to skip (default 1)
- `start_delay` (float): Delay in seconds after launching simulator (default 5.0)
- `car_config` (dict): Optional car appearance settings (`body_style`, `body_rgb`, `car_name`, `font_size`)
- `cam_config` (dict): Optional camera settings (resolution, FOV, position, rotation, fisheye distortion)
- `cam_config_b` (dict): Optional second camera configuration
- `lidar_config` (dict): Optional LIDAR sensor configuration

**Important**: Camera/LIDAR config keys use snake_case (e.g., `deg_per_sweep_inc`), not CamelCase. The code will raise ValueError if old CamelCase keys are detected.

## Code Style

- **Formatter**: Black with 127 character line length
- **Import sorting**: isort
- **Linter**: ruff (checks for E9, F63, F7, F82 errors)
- **Type checking**: pytype

The codebase targets Python 3.7+ (tested on 3.7, 3.8, 3.9, 3.10, 3.12).

## Available Environments

All environments use the same observation/action spaces but differ in the simulated track:

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
- `donkey-mountain-track-v0` (registered but not in README)

## Examples

See `examples/` directory for working code samples. **All examples have been updated for Gymnasium.**

- `gym_test.py`: Basic environment test with random actions
- `test_cam_config.py`: Camera configuration testing
- `reinforcement_learning/ppo_train.py`: PPO training with Stable-Baselines3
- `reinforcement_learning/ddqn.py`: DDQN implementation
- `genetic_alg/`: Genetic algorithm (⚠️ requires TensorFlow 1.x, legacy)
- `supervised_learning/`: Supervised learning from recorded data

**Important**: See `examples/EXAMPLES_README.md` for:
- Complete installation requirements for each example
- Usage instructions and command-line options
- Troubleshooting common issues
- Notes on Gymnasium API changes

### Running Examples with Live Simulator

When testing examples with a running simulator, use `--sim manual`:
```bash
python examples/gym_test.py --sim manual --env_name donkey-generated-track-v0 --port 9091
```

### Gymnasium Wrapper Compatibility

With Gymnasium, environments are wrapped by default. To access custom gym-donkeycar attributes:
```python
env.unwrapped.viewer.exit_scene()
env.unwrapped.viewer.set_reward_fn(custom_fn)
env.unwrapped.viewer.set_episode_over_fn(custom_fn)
```

**Do not use** `env.viewer` directly - it will raise `AttributeError` with Gymnasium wrappers.

## Simulator Binary

The Unity simulator binary must be downloaded separately from: https://github.com/tawnkramer/gym-donkeycar/releases

The simulator can be:
1. Launched automatically by passing `exe_path` in config
2. Launched manually before running the environment (don't pass `exe_path`)
