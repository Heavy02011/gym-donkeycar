# OpenAI Gym to Gymnasium Migration Documentation

This document describes the migration from OpenAI Gym to Gymnasium (the actively maintained fork by the Farama Foundation).

## Overview

OpenAI Gym has been deprecated and is no longer actively maintained. Gymnasium is the official successor, maintained by the Farama Foundation, and provides ongoing support, bug fixes, and compatibility with modern Python and NumPy versions.

## Summary of Changes

### Dependency Changes

| Before | After |
|--------|-------|
| `gym==0.22` | `gymnasium>=0.29.0` |

**File changed**: `setup.py`

### Import Changes

All imports were updated from `gym` to `gymnasium`:

```python
# Before
import gym
from gym import spaces
from gym.envs.registration import register
from gym.utils import seeding

# After
import gymnasium as gym
from gymnasium import spaces
from gymnasium.envs.registration import register
# seeding removed - now handled by numpy directly
```

### API Changes

#### 1. `step()` Method Return Values

The `step()` method now returns 5 values instead of 4:

```python
# Before (4 values)
observation, reward, done, info = env.step(action)

# After (5 values)
observation, reward, terminated, truncated, info = env.step(action)
```

- `terminated`: Episode ended due to task completion or failure (e.g., collision)
- `truncated`: Episode ended due to external constraints (e.g., time limit)
- Use `done = terminated or truncated` for backward compatibility

#### 2. `reset()` Method

The `reset()` method now returns 2 values and accepts seed parameter:

```python
# Before
observation = env.reset()

# After
observation, info = env.reset(seed=None, options=None)
```

#### 3. `render()` Method

Render mode is now set at initialization instead of per-call:

```python
# Before
env = gym.make("donkey-warehouse-v0", conf=conf)
env.render(mode="human")

# After
env = gym.make("donkey-warehouse-v0", conf=conf, render_mode="human")
env.render()  # Uses render_mode from initialization
```

#### 4. `seed()` Method Removed

The deprecated `seed()` method has been removed. Seeding is now done via `reset()`:

```python
# Before
env.seed(42)
env.reset()

# After
env.reset(seed=42)
```

#### 5. Metadata Format Change

```python
# Before
metadata = {"render.modes": ["human", "rgb_array"]}

# After
metadata = {"render_modes": ["human", "rgb_array"]}
```

### Files Changed

| File | Changes |
|------|---------|
| `setup.py` | Updated dependency from `gym==0.22` to `gymnasium>=0.29.0`, updated description |
| `gym_donkeycar/__init__.py` | Updated import from `gym.envs.registration` to `gymnasium.envs.registration` |
| `gym_donkeycar/envs/donkey_env.py` | Major API updates (step, reset, render, metadata) |
| `tests/test_gym_donkeycar.py` | Updated to use `gymnasium` and `env.unwrapped` for accessing custom attributes |
| `examples/gym_test.py` | Updated API usage |
| `examples/reinforcement_learning/ppo_train.py` | Updated API usage |
| `examples/reinforcement_learning/ddqn.py` | Updated API usage |
| `examples/genetic_alg/simple_gen_driver.py` | Updated API usage |
| `examples/test_cam_config.py` | Updated API usage |
| `README.md` | Updated documentation with new API examples |
| `docs/usage.rst` | Updated description |
| `Makefile` | Fixed ruff lint command for newer version |

### Code Formatting Changes

The following files were reformatted by black/isort (no functional changes):
- `examples/genetic_alg/simple_gen.py`
- `examples/supervised_learning/models.py`
- `examples/supervised_learning/train.py`
- `gym_donkeycar/core/client.py`
- `gym_donkeycar/core/message.py`
- `gym_donkeycar/core/sim_client.py`
- `gym_donkeycar/envs/donkey_proc.py`
- `gym_donkeycar/envs/donkey_sim.py`

## Testing

### Test Environment

- **Python version**: 3.12.3
- **pytest version**: 9.0.2
- **gymnasium version**: 1.2.3
- **Platform**: Linux

### Tests Executed

```bash
python -m pytest tests/ -v
```

### Test Results

```
================================================= test session starts ==================================================
platform linux -- Python 3.12.3, pytest-9.0.2, pluggy-1.6.0 -- /usr/bin/python
cachedir: .pytest_cache
rootdir: /home/runner/work/gym-donkeycar/gym-donkeycar
configfile: setup.cfg
plugins: cov-7.0.0, env-1.2.0, xdist-3.8.0, mock-3.15.1
collecting ... collected 2 items

tests/core/test_fps.py::test_fps PASSED                                                                          [ 50%]
tests/test_gym_donkeycar.py::test_load_gyms PASSED                                                               [100%]

=================================================== warnings summary ===================================================
tests/test_gym_donkeycar.py::test_load_gyms
  /home/runner/.local/lib/python3.12/site-packages/gymnasium/spaces/box.py:236: UserWarning: WARN: Box low's precision lowered by casting to float32, current low.dtype=float64
    gym.logger.warn(

tests/test_gym_donkeycar.py::test_load_gyms
  /home/runner/.local/lib/python3.12/site-packages/gymnasium/spaces/box.py:306: UserWarning: WARN: Box high's precision lowered by casting to float32, current high.dtype=float64
    gym.logger.warn(

============================================ 2 passed, 3 warnings in 50.18s ============================================
```

### Test Summary

| Test | Status | Description |
|------|--------|-------------|
| `tests/core/test_fps.py::test_fps` | ✅ PASSED | Tests FPS timer functionality |
| `tests/test_gym_donkeycar.py::test_load_gyms` | ✅ PASSED | Tests loading all gym environments |

### Warnings

The float precision warnings are expected behavior in Gymnasium when using float64 values for Box space bounds. These are informational warnings and do not affect functionality.

### Linting Results

```bash
make lint
```

```
# stop the build if there are Python syntax errors or undefined names
ruff check gym_donkeycar/ tests/ docs/conf.py setup.py examples/ --select=E9,F63,F7,F82 --output-format=full
All checks passed!

# exit-zero treats all errors as warnings.
ruff check gym_donkeycar/ tests/ docs/conf.py setup.py examples/ --exit-zero
All checks passed!
```

### Code Style Results

```bash
make check-codestyle
```

```
# Sort imports
isort --check gym_donkeycar/ tests/ docs/conf.py setup.py examples/
# Reformat using black
black --check -l 127 gym_donkeycar/ tests/ docs/conf.py setup.py examples/
All done! ✨ 🍰 ✨
27 files would be left unchanged.
```

## Migration Guide for Users

### Updating Your Code

If you have existing code using `gym-donkeycar`, update it as follows:

```python
# Old code
import gym
import gym_donkeycar

env = gym.make("donkey-warehouse-v0", conf=conf)
obs = env.reset()
for _ in range(100):
    action = env.action_space.sample()
    obs, reward, done, info = env.step(action)
    if done:
        obs = env.reset()
env.close()

# New code
import gymnasium as gym
import gym_donkeycar

env = gym.make("donkey-warehouse-v0", conf=conf)
obs, info = env.reset()
for _ in range(100):
    action = env.action_space.sample()
    obs, reward, terminated, truncated, info = env.step(action)
    if terminated or truncated:
        obs, info = env.reset()
env.close()
```

### Breaking Changes

1. **Import statement**: Change `import gym` to `import gymnasium as gym`
2. **step() return values**: Now returns 5 values (add handling for `terminated` and `truncated`)
3. **reset() return values**: Now returns 2 values (observation and info dict)
4. **seed() method**: Removed - use `reset(seed=...)` instead
5. **render() method**: No longer accepts `mode` parameter - set at initialization

## Security Assessment

CodeQL security analysis was performed and found **0 vulnerabilities** in the code changes.

## Conclusion

The migration from OpenAI Gym to Gymnasium was completed successfully. All tests pass, code style checks pass, and no security vulnerabilities were introduced. The migration follows Gymnasium's official API guidelines and maintains backward compatibility for the core environment functionality.
