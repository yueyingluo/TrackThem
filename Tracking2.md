Read [](file:///home/user/catch_it/configs/env/DcmmCfg.py#1-1), lines 1 to 100

Here is a detailed plan to implement the `Tracking2` task where the robot tracks two balls released in sequence.

### Overview
The core idea is to modify the environment to support two objects (`object` and `object2`) and implement a sequential logic where the robot focuses on the first object until it is successfully tracked (touched), and then switches focus to the second object. The observation space will remain relative to the *current* target object, allowing the existing PPO agent to be reused.

### Implementation Plan

#### 1. Modify MujocoDcmm.py
We need to allow the `MJ_DCMM` class to dynamically switch its target object so that internal calculations (like inverse kinematics or state retrieval) focus on the correct ball.

*   **Add `set_target_object` method**:
    *   Create a new method `set_target_object(self, object_name)` in the `MJ_DCMM` class.
    *   This method should update `self.object_name` and re-fetch `self.object_id` using `mujoco.mj_name2id`.

#### 2. Modify DcmmVecEnv.py
This file requires the most changes to handle the logic of two objects.

*   **Update `__init__`**:
    *   Add `"Tracking2"` to the list of valid tasks.
    *   Initialize a list of target names, e.g., `self.target_names = ['object', 'object2']`.

*   **Update `_reset_object`**:
    *   Currently, this method randomizes a single object in the XML.
    *   Modify it to:
        1.  Find the `body` element named `object`.
        2.  Randomize it as usual.
        3.  **Clone** this body element to create a second object.
        4.  Rename the clone to `object2`.
        5.  Append the clone to the `worldbody` of the XML tree.
        6.  Ensure both objects have valid joints and geometries.

*   **Update `reset`**:
    *   Initialize a tracking index, e.g., `self.target_idx = 0`.
    *   Call `self.Dcmm.set_target_object("object")` to focus on the first ball.
    *   Randomize positions/velocities for **both** objects.
    *   Set the state of `object2` to be "waiting" (e.g., static or out of view) until it is its turn to be released.

*   **Update `_step_mujoco_simulation`**:
    *   Implement the sequential logic:
        *   Check if the current target (`self.target_names[self.target_idx]`) has been successfully tracked (touched).
        *   If **Target 1** is touched:
            *   Increment `self.target_idx` to 1.
            *   Call `self.Dcmm.set_target_object("object2")`.
            *   Update `self.object_name` and `self.object_id` in the environment class as well.
            *   "Release" `object2` (apply its throw velocity/physics).
            *   Reset `self.step_touch` to `False` so the agent has to touch the new target.
    *   Ensure `_get_obs` and `_get_info` use the *current* target object (which they should if they reference `self.Dcmm.object_name`).

*   **Update `step`**:
    *   Modify the termination condition.
    *   The episode should only be considered "successful" (truncated) if **Target 2** is successfully tracked.
    *   If Target 1 is missed (falls), the episode terminates as failure.

#### 3. Modify train_DCMM.py
*   **Update Task Handling**:
    *   Add `Tracking2` to the condition that selects the PPO agent.
    *   You can likely reuse `PPO_Track` since the observation space (relative to the active target) is consistent.
    *   Example:
        ```python
        PPO = PPO_Track if config.task in ['Tracking', 'Tracking2'] else ...
        ```

### Summary of Files to Edit
1.  MujocoDcmm.py: Add helper to switch target object.
2.  DcmmVecEnv.py: Implement dual-object XML generation and sequential game logic.
3.  train_DCMM.py: Enable the new task configuration.