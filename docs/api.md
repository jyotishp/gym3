# API

## Env Interface

```py
class Env:
    """
    An interface for reinforcement learning environments.

    :param ob_space: ValType representing the valid observations generated by the environment
    :param ac_space: ValType representing the valid actions that an agent can take in the environment
    :param num: number of simultaneous episodes
    """

    def __init__(self, ob_space: ValType, ac_space: ValType, num: int):
        ...

    def observe(self) -> Tuple[Any, Any, Any]:
        """
        Structured data that's accessible to the agent

        Can be called zero or more times per act() call; it is idempotent and does
        not affect the environment.  Returns the observation after the last act()
        call has affected the environment or the initial state if no act()
        call has been made.

        The following snippet shows what the start of a rollout looks like, and
        the usual convention for indexing the values:

        env = Env()
        # initial reward is ignored; didn't depend on actions
        _reward, ob_0, first_0 = env.observe()
        env.act(ac_0)
        reward_0, ob_1, first_1 = env.observe()
        env.act(ac_1)

        Note that the initial reward `_reward` is ignored by algorithms, since it
        didn't depend on the actions. And that first_0 is always True since the
        environment was just created.

        :returns:
            reward: (float array) last reward, shaped (num,)
            ob: observation matching ob_space, with (num,) prepended to the shape of each leaf
            first: (bool array) whether new episode just started, shaped (num,)
        """

    def get_info(self) -> List[Dict]:
        """
        Return unstructured diagnostics that aren't accessible to the agent
        Per-episode stats, rendered images, etc.
        Corresponds to same timestep as the observation from observe().

        :returns: a list of dictionaries with length `self.num`
        """

    def act(self, ac: Any) -> None:
        """
        Apply an action

        :param ac: action matching ac_space, with (num,) prepended to the shape of each leaf
        """

    def callmethod(
        self, method: str, *args: Sequence[Any], **kwargs: Sequence[Any]
    ) -> List[Any]:
        """
        Call a method on the underlying python object that offers the Gym3 interface.
        By default, this is the Env instance (self), but this can be overridden if you are
        encapsulating another object or if your wrapper wishes to handle the method call.

        :param method: name of method to call on the base object
        :param *args, **kwargs: the value of each argument must be a list with length equal to `self.num`

        :returns: A list of objects, with length equal to `self.num`
        """
```

## Important Classes/Functions

* `Env` - Abstract base class that specifies the interface for `gym3` environments.
* `Wrapper` - Base class for wrappers, the default wrapper passes everything through.  Override methods to make your own wrapper.
* `FromGymEnv`, `FromBaselinesVecEnv`, `ToGymEnv`, `ToBaselinesVecEnv` - Convert to and from other environment interfaces.
* `ConcatEnv` - Combine multiple `gym3` environments into a single one.
* `SubprocEnv` - Given a function that makes a `gym3` environment, instantiate it in a subprocess.
* `types` - `ac_space` and `ob_space` are defined using types from the `types` module, consisting of `TensorType` and `DictType`.

## Included Wrappers

* `AsynchronousWrapper` - Take an environment with a synchronous `act()` function and make it asynchronous with a thread.
* `TrajectoryRecorderWrapper` - Record episodes to pickle files
* `ExtractDictObWrapper` - For a dictionary observation space, extract a single key's value.  This is useful if your training code does not support dictionary observation spaces.
* `VideoRecorderWrapper` - If an environment produces RGB images, use this wrapper to save one video per episode.
* `ViewerWrapper` - If an environment produces RGB images, use this wrapper to pop up a window that a human can then view.

## Utilities

* `types_np` - A module containing useful functions that act on objects from `types`, such as `zeros` and `sample`, that return `numpy` arrays.
* `types_th` - The same as `types_np` but for `torch` tensors.
* `call_func` - A function that takes a python path string such as `"procgen:ProcgenEnv"` and calls the function at that path with the given keyword arguments.  Useful as a replacement for `gym.make`.
* `libenv` - A module that can be used for creating `gym3` environments written in any language that can expose a C interface (defined in [`libenv.h`](../gym3/libenv.h))
* `Interactive` - Create an interactive window where a human can operate an environment using a keyboard.  This requires that keyboard keys can be mapped to environment actions, and that the environment produces RGB images.  See the bottom of [`gym3/interactive.py`](../gym3/interactive.py) for an example.
* `testing` - A module with some useful testing tools
    * `AssertSpacesWrapper` - A wrapper that will raise an exception if actions or observations do not match `ac_space` or `ob_space`, respectively.  This is useful in test scripts for your environment to make sure you are not using actions or observations that do not match the space you defined.
    * `IdentityEnv` - A simple test environment for debugging where the observations are the correct actions.
    * `TimingEnv` - A environment for speed tests, using this lets you see how fast your RL setup would be with an environment that takes almost no time per step.
    * `FixedSequenceEnv` - A simple environment where the agent must guess a fixed sequence through its actions, with no observations to rely on.
* `unwrap` - A function that will return the `Env` instance at the bottom of a stack of wrappers.  Generally you should use `callmethod()` when possible, as it works with `SubprocEnv`, whereas `unwrap` does not.