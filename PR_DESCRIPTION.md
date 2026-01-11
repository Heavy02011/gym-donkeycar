# Pull Request: Migrate from OpenAI Gym to Gymnasium

## Summary

This PR migrates gym-donkeycar from the deprecated OpenAI Gym to Gymnasium (the actively maintained successor by the Farama Foundation).

**Why this migration is important:**
- OpenAI Gym is deprecated and no longer maintained
- Gymnasium is the official successor with ongoing support and bug fixes
- Ensures compatibility with modern Python (3.7+) and NumPy versions
- Aligns with the broader RL community's migration path

## Changes Overview

### 1. Core Dependency Update
- **Before:** `gym==0.22`
- **After:** `gymnasium>=0.29.0`
- Updated in `setup.py`

### 2. API Changes

#### Updated `step()` Return Values (5 instead of 4)
```python
# Before
observation, reward, done, info = env.step(action)

# After  
observation, reward, terminated, truncated, info = env.step(action)
```
- `terminated`: Episode ended due to task conditions (collision, out of bounds, etc.)
- `truncated`: Not currently used (always False in this implementation)
- Backward compatibility: `done = terminated or truncated`

#### Updated `reset()` Return Values (2 instead of 1)
```python
# Before
observation = env.reset()

# After
observation, info = env.reset(seed=None, options=None)
```

#### Environment Registration Updated
```python
# Changed from gym.envs.registration to gymnasium.envs.registration
from gymnasium.envs.registration import register
```

### 3. Code Changes

**Files modified:**
- `gym_donkeycar/__init__.py` - Updated imports and environment registration
- `gym_donkeycar/envs/donkey_env.py` - Updated base environment API
- `gym_donkeycar/envs/donkey_sim.py` - Updated controller interface
- All example files in `examples/` directory
- All test files in `tests/` directory

**Key implementation details:**
- All imports changed from `import gym` to `import gymnasium as gym`
- Spaces imported from `gymnasium.spaces`
- Metadata format updated to use `"render_modes"` instead of `"render.modes"`
- Removed deprecated `seed()` method (seeding now via `reset(seed=...)`)
- Updated all examples to use new API

### 4. Documentation Updates

- `README.md` - Updated examples and API documentation
- `docs/` - All Sphinx documentation updated
- `HISTORY.rst` - Added comprehensive migration notes for v1.3.1
- `CONTRIBUTING.rst` - Updated references
- `tests/UPDATE_GYM2GYMNASIUM.md` - Complete migration guide for users
- `examples/EXAMPLES_README.md` - Updated usage instructions

### 5. Examples Updated

All examples have been updated and tested with the new Gymnasium API:
- `examples/gym_test.py` - Basic environment test
- `examples/test_cam_config.py` - Camera configuration
- `examples/reinforcement_learning/ppo_train.py` - PPO training with Stable-Baselines3
- `examples/reinforcement_learning/ddqn.py` - DDQN implementation  
- `examples/supervised_learning/` - Supervised learning examples
- `examples/genetic_alg/` - Genetic algorithm (legacy, requires TensorFlow 1.x)

## Testing

✅ All tests passing:
```bash
pytest -v tests/
# 2 passed, 0 failed
```

✅ Code quality checks passing:
- Black formatting: ✅ Compliant (127 char line length)
- isort: ✅ Import ordering correct
- Ruff linting: ✅ No errors (E9, F63, F7, F82 checks)

⚠️ Known Issue (Pre-existing):
- `gym_donkeycar/test/client.test.py` uses deprecated `asyncore` (removed in Python 3.12)
- This is a legacy test file not part of the main test suite
- Does not block the migration
- Should be addressed in a future PR

## Backward Compatibility

**Breaking Changes:**
- Python code using the old Gym API will need to update to the new Gymnasium API
- Users will need to install `gymnasium>=0.29.0` instead of `gym`
- See `tests/UPDATE_GYM2GYMNASIUM.md` for detailed migration guide

**Migration Path for Users:**
1. Update dependency: `pip install gymnasium>=0.29.0`
2. Update imports: `import gymnasium as gym`
3. Update `step()` calls to handle 5 return values
4. Update `reset()` calls to handle 2 return values
5. See migration guide for complete details

## Environments Supported

All existing environments continue to work with the new API:
- donkey-warehouse-v0
- donkey-generated-roads-v0
- donkey-avc-sparkfun-v0
- donkey-generated-track-v0
- donkey-roboracingleague-track-v0
- donkey-waveshare-v0
- donkey-minimonaco-track-v0
- donkey-warren-track-v0
- donkey-thunderhill-track-v0
- donkey-circuit-launch-track-v0
- donkey-mountain-track-v0

## Related Documentation

- Gymnasium GitHub: https://github.com/Farama-Foundation/Gymnasium
- Gymnasium Docs: https://gymnasium.farama.org/
- Migration Guide: https://gymnasium.farama.org/content/migration-guide/
- This PR's detailed migration notes: `tests/UPDATE_GYM2GYMNASIUM.md`

## Checklist

- [x] Code migrated from Gym to Gymnasium
- [x] All tests passing
- [x] Code style checks passing (black, isort, ruff)
- [x] Documentation updated
- [x] Examples updated and working
- [x] Migration guide created for users
- [x] HISTORY.rst updated with version notes
- [x] Backward compatibility notes documented

## Author Notes

This migration ensures gym-donkeycar remains compatible with the actively maintained RL ecosystem. Gymnasium provides ongoing support, security updates, and compatibility with modern Python versions.

The migration is complete and tested. All existing functionality is preserved, with only API signature changes required by the Gymnasium library.
