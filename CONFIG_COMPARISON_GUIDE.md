# Config Comparison: Jump3 vs Horse Stance (Working vs Non-Working)

## Main Issue: Missing `motion_lib_type`

### ✅ **FIXED** - Added this to Jump3 config (line 515):
```yaml
motion:
  motion_lib_type: WJX  # ← CRITICAL! Must match urcirobot.py import
  motion_file: humanoidverse/data/motions/ANTA/Jump/0-jump3_origin_inter0.5_S0-30_E60-30.pkl
```

**Why this matters:**
- Your `urcirobot.py` hardcodes: `from humanoidverse.utils.motion_lib.motion_lib_robot_WJX import MotionLibRobotWJX`
- Without `motion_lib_type: WJX`, the system tries to use the wrong motion library
- This was causing the `torch.int32` indexing errors we fixed earlier

---

## Other Key Differences

### 1. **Algorithm Type**
| Config | Algorithm | Multi-Head Rewards |
|--------|-----------|-------------------|
| Jump3 | PPO | ❌ No |
| Horse Stance | **MHPPO** | ✅ Yes (`use_vec_reward: true`) |

**Jump3 (line 28):**
```yaml
algo:
  _target_: humanoidverse.agents.ppo.ppo.PPO
```

**Horse Stance (line 28):**
```yaml
algo:
  _target_: humanoidverse.agents.mh_ppo.mh_ppo.MHPPO
  config:
    ...
    l2c2:
      enable: false
```

---

### 2. **Robot Type & Asset Paths**
| Config | Robot | Asset Root |
|--------|-------|-----------|
| Jump3 | `g1_29dof_anneal_23dof` | `humanoidverse/data/robots` |
| Horse Stance | `g1_23dof_lock_wrist` | `description/robots` |

**Jump3:**
```yaml
asset:
  robot_type: g1_29dof_anneal_23dof
  asset_root: humanoidverse/data/robots
```

**Horse Stance:**
```yaml
asset:
  robot_type: g1_23dof_lock_wrist
  asset_root: "description/robots"
```

---

### 3. **Environment Configuration**

**Horse Stance has additional termination** (line 112):
```yaml
termination:
  ...
  terminate_when_dof_far: false  # ← Extra termination type
  
termination_curriculum:
  ...
  terminate_when_dof_far_curriculum:  # ← Extra curriculum
    enable: true
    init: 3.0
    max: 3.0
    min: 1.0
```

**Horse Stance has soft dynamic correction** (line 154):
```yaml
soft_dynamic_correction:
  enable: false
  alpha: 0.1
  type: deter
  curriculum:
    enable: true
    max_alpha: 0.9
    min_alpha: 0.0
```

---

### 4. **Observations**

**Horse Stance has domain randomization observations** (lines 913-919):
```yaml
critic_obs:
  - base_lin_vel
  - base_ang_vel
  - projected_gravity
  - dof_pos
  - dof_vel
  - actions
  - ref_motion_phase
  - dif_local_rigid_body_pos
  - local_ref_rigid_body_pos
  - dr_base_com          # ← Extra DR obs
  - dr_link_mass         # ← Extra DR obs
  - dr_kp                # ← Extra DR obs
  - dr_kd                # ← Extra DR obs
  - dr_friction          # ← Extra DR obs
  - dr_ctrl_delay        # ← Extra DR obs
  - history_critic
```

**Horse Stance has motion_len** (line 996):
```yaml
post_compute_config: {}
motion_len: -1  # ← Additional config
```

---

### 5. **Domain Randomization**

**Jump3 (minimal DR):**
```yaml
domain_rand:
  push_robots: false
  randomize_base_com: false
  randomize_link_mass: false
  randomize_pd_gain: false
  randomize_friction: false
  randomize_torque_rfi: false
  randomize_ctrl_delay: false
```

**Horse Stance (extensive DR):**
```yaml
domain_rand:
  push_robots: true
  randomize_base_com: true
  randomize_link_mass: true
  randomize_link_inertia: true  # ← Extra
  randomize_pd_gain: true
  randomize_friction: true
  randomize_torque_rfi: true
  randomize_ctrl_delay: true
  use_rao: true  # ← Extra: Residual Action Offset
  rao_lim: 0.05
  _push_fixed: true
```

---

### 6. **Rewards**

**Jump3 rewards:**
```yaml
rewards:
  set_reward: Tairan
  set_reward_date: 20241025
  reward_scales:
    teleop_body_position_extend: 1.0
    teleop_vr_3point: 1.6
    teleop_body_position_feet: 2.1
    ...
    penalty_torques: -1.0e-06
    penalty_action_rate: -0.5
    penalty_feet_ori: -2.0
    ...
```

**Horse Stance rewards:**
```yaml
rewards:
  set_reward: Anonymity
  set_reward_date: 20250417
  reward_scales:
    teleop_contact_mask: 0.5  # ← Extra
    teleop_max_joint_position: 1.0  # ← Extra
    ...
    feet_air_time: 1.0  # ← Extra
    penalty_feet_contact_forces: -0.01  # ← Extra
    penalty_stumble: -2.0  # ← Extra
    collision: -30.0  # ← Extra
  
  adaptive_tracking_sigma:  # ← Extra
    enable: true
    alpha: 0.001
```

---

### 7. **Control Parameters**

**Jump3 (more compliant):**
```yaml
control:
  stiffness:
    knee: 200
    ankle_pitch: 20
    ankle_roll: 20
  damping:
    knee: 5.0
    ankle_pitch: 0.2
    ankle_roll: 0.1
```

**Horse Stance (stiffer):**
```yaml
control:
  stiffness:
    knee: 150
    ankle_pitch: 40  # ← 2x stiffer
    ankle_roll: 40   # ← 2x stiffer
  damping:
    knee: 4.0
    ankle_pitch: 2.0  # ← 10x more damping
    ankle_roll: 2.0   # ← 20x more damping
```

---

## Summary: Why Horse Stance Works

1. ✅ **Has `motion_lib_type: WJX`** (CRITICAL - now added to Jump3)
2. Uses MHPPO (Multi-Head PPO) instead of regular PPO
3. More extensive domain randomization for robustness
4. Additional observations for domain randomization states
5. More reward terms for better control
6. Different robot model (23 DOF vs 29 DOF)

---

## What You Need to Do

### ✅ **Already Fixed:**
- Added `motion_lib_type: WJX` to Jump3 config

### If Jump3 Still Doesn't Work:

**Option 1: Use Horse Stance config style (Recommended)**
- Copy the working config structure
- Replace motion file and robot type
- Keep all the DR and reward configurations

**Option 2: Minimal Changes to Jump3**
Just add what's absolutely necessary:
```yaml
motion:
  motion_lib_type: WJX  # ✅ Already added
  motion_file: humanoidverse/data/motions/ANTA/Jump/0-jump3_origin_inter0.5_S0-30_E60-30.pkl

obs:
  obs_dims:
    ...
  motion_len: -1  # Add this line
```

**Option 3: Check if you need MHPPO**
If your Jump3 was trained with regular PPO but Horse Stance uses MHPPO, the policy architectures might be incompatible. Check your exported ONNX model to see what it expects.

---

## Quick Test

```bash
cd /home/qiulm/sources/ASAP
conda activate hvgym
python humanoidverse/urci.py +checkpoint=logs/MotionTracking/.../model_15000.pt
```

If it works now, you're all set! If not, the issue might be with the algorithm type (PPO vs MHPPO) or observation dimensions.


