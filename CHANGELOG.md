# Changelog

## Release 1.1.0

### Api Extensions
- `evaluation`: New package for repeating the same experiment with multiple seeds and aggregating the results. #1074 #1141
  - The module `evaluation.launchers` for parallelization is currently in alpha state.
- `data`:
  - `Batch`:
    - Add methods `to_dict` and `to_list_of_dicts`. #1063 #1098
    - Add methods `to_numpy_` and `to_torch_`. #1098, #1117
    - Add `__eq__` (semantic equality check). #1098
    - `keys()` deprecated in favor of `get_keys()` (needed to make iteration consistent with naming) #1105.
  - `data.collector`:
    - `Collector`:
      - Introduced `BaseCollector` as a base class for all collectors. #1123
      - Add method `close` #1063
      - Method `reset` is now more granular (new flags controlling behavior). #1063
    - `CollectStats`: Add convenience constructor `with_autogenerated_stats`. #1063
- `trainer`:
  - Trainers can now control whether collectors should be reset prior to training. #1063
- policy:
  - introduced attribute `in_training_step` that is controlled by the trainer. #1123
  - policy automatically set to `eval` mode when collecting and to `train` mode when updating. #1123
- `highlevel`:
  - `SamplingConfig`:
    - Add support for `batch_size=None`. #1077 
    - Add `training_seed` for explicit seeding of training and test environments, the `test_seed` is inferred from `training_seed`. #1074
  - `experiment`: 
     - `Experiment` now has a `name` attribute, which can be set using `ExperimentBuilder.with_name` and 
       which determines the default run name and therefore the persistence subdirectory.
       It can still be overridden in `Experiment.run()`, the new parameter name being `run_name` rather than
       `experiment_name` (although the latter will still be interpreted correctly). #1074 #1131
     - Add class `ExperimentCollection` for the convenient execution of multiple experiment runs #1131
     - `ExperimentBuilder`: 
         - Add method `build_seeded_collection` for the sound creation of multiple
           experiments with varying random seeds #1131
         - Add method `copy` to facilitate the creation of multiple experiments from a single builder #1131
  - `env`:
    - Added new `VectorEnvType` called `SUBPROC_SHARED_MEM_AUTO` and used in for Atari and Mujoco venv creation. #1141
- Loggers can now restore the logged data into python by using the new `restore_logged_data` method. #1074
- `utils`:
  - `net.continuous.Critic`:
    - Add flag `apply_preprocess_net_to_obs_only` to allow the
      preprocessing network to be applied to the observations only (without
      the actions concatenated), which is essential for the case where we want
      to reuse the actor's preprocessing network #1128
  - `torch_utils` (new module)
    - Added context managers `torch_train_mode` and `policy_within_training_step` #1123
  - `print`
    - `DataclassPPrintMixin` now supports outputting a string, not just printing the pretty repr. #1141

### Fixes
- `CriticFactoryReuseActor`: Enable the Critic flag `apply_preprocess_net_to_obs_only` for continuous critics, 
  fixing the case where we want to reuse an actor's preprocessing network for the critic (affects usages
  of the experiment builder method `with_critic_factory_use_actor` with continuous environments) #1128
- `atari_network.DQN`:
  - Fix constructor input validation #1128
  - Fix `output_dim` not being set if `features_only`=True and `output_dim_added_layer` is not None #1128

### Internal Improvements
- `Collector`s rely less on state, the few stateful things are stored explicitly instead of through a `.data` attribute. #1063
- Introduced a first iteration of a naming convention for vars in `Collector`s. #1063
- Generally improved readability of Collector code and associated tests (still quite some way to go). #1063
- Improved typing for `exploration_noise` and within Collector. #1063
- Better variable names related to model outputs (logits, dist input etc.). #1032
- Improved typing for actors and critics, using Tianshou classes like `Actor`, `ActorProb`, etc., 
instead of just `nn.Module`. #1032
- Added interfaces for most `Actor` and `Critic` classes to enforce the presence of `forward` methods. #1032
- Simplified `PGPolicy` forward by unifying the `dist_fn` interface (see associated breaking change). #1032
- Use `.mode` of distribution instead of relying on knowledge of the distribution type. #1032
- Exception no longer raised on `len` of empty `Batch`. #1084
- tests and examples are covered by `mypy`. #1077
- `NetBase` is more used, stricter typing by making it generic. #1077
- Use explicit multiprocessing context for creating `Pipe` in `subproc.py`. #1102

### Breaking Changes
- `data`:
  - `Collector`:
    - Removed `.data` attribute. #1063
    - Collectors no longer reset the environment on initialization. 
      Instead, the user might have to call `reset` expicitly or pass `reset_before_collect=True` . #1063
    - Removed `no_grad` argument from `collect` method (was unused in tianshou). #1123
  - `Batch`:
    - Fixed `iter(Batch(...)` which now behaves the same way as `Batch(...).__iter__()`. 
      Can be considered a bugfix. #1063
    - The methods `to_numpy` and `to_torch` in are not in-place anymore 
      (use `to_numpy_` or `to_torch_` instead). #1098, #1117
- Logging:
  - `BaseLogger.prepare_dict_for_logging` is now abstract. #1074
  - Removed deprecated and unused `BasicLogger` (only affects users who subclassed it). #1074
- VectorEnvs now return an array of info-dicts on reset instead of a list. #1063
- Changed interface of `dist_fn` in `PGPolicy` and all subclasses to take a single argument in both
continuous and discrete cases. #1032
- `utils.net.common.Recurrent` now receives and returns a `RecurrentStateBatch` instead of a dict. #1077
- `AtariEnvFactory` constructor (in examples, so not really breaking) now requires explicit train and test seeds. #1074
- `EnvFactoryRegistered` now requires an explicit `test_seed` in the constructor. #1074


### Tests
- Fixed env seeding it `test_sac_with_il.py` so that the test doesn't fail randomly. #1081

### Dependencies
- [DeepDiff](https://github.com/seperman/deepdiff) added to help with diffs of batches in tests. #1098
- Bumped black, idna, pillow
- New extra "eval"

Started after v1.0.0