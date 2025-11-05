# Testing Your Trained Policy in MuJoCo (Sim2Sim)

## Quick Setup Guide

### Step 1: Create Deployment Environment

```bash
mamba create -n asap_deploy python=3.10
mamba activate asap_deploy

# Install ROS2
conda config --env --add channels conda-forge
conda config --env --add channels robostack-staging
conda config --env --remove channels defaults
conda install ros-humble-desktop

# Install dependencies
cd /home/qiulm/sources/ASAP/sim2real
pip install -e .
pip install mujoco onnxruntime numpy scipy
```

### Step 2: Prepare Your Policy

Your policy is already exported! Located at:
```
logs/MotionTracking/20251027_021530-MotionTracking_Jump3-motion_tracking-g1_29dof_anneal_23dof/exported/model_15000.onnx
```

Copy it to the sim2real models directory:
```bash
cd /home/qiulm/sources/ASAP
mkdir -p sim2real/models/mimic/jump3_custom
cp logs/MotionTracking/20251027_021530-MotionTracking_Jump3-motion_tracking-g1_29dof_anneal_23dof/exported/model_15000.onnx \
   sim2real/models/mimic/jump3_custom/model_15000.onnx
```

### Step 3: Run Sim2Sim

**Terminal 1 - Start MuJoCo Simulation:**
```bash
cd /home/qiulm/sources/ASAP/sim2real
conda activate asap_deploy
python sim_env/base_sim.py --config=config/g1_29dof_hist.yaml
```

**Terminal 2 - Run Your Policy:**
```bash
cd /home/qiulm/sources/ASAP/sim2real
conda activate asap_deploy
python rl_policy/deepmimic_dec_loco_height.py \
  --config=config/g1_29dof_hist.yaml \
  --loco_model_path=./models/dec_loco/20250109_231507-noDR_rand_history_loco_stand_height_noise-decoupled_locomotion-g1_29dof/model_6600.onnx \
  --mimic_model_paths=./models/mimic
```

### Step 4: Control the Robot

In the policy terminal (Terminal 2), use these keyboard controls:

**Mode Switching:**
- `]` - Activate locomotion policy (walking/standing)
- `[` - Activate ASAP policy (motion tracking - your Jump3!)
- `;` - Switch between motion tracking skills
- `i` - Reset to initial position
- `o` - Emergency stop

**Locomotion Controls (when in locomotion mode):**
- `w/a/s/d` - Control linear velocity
- `q/e` - Control angular velocity
- `z` - Set all commands to zero
- `=` - Switch between standing and walking

**MuJoCo Viewer:**
- Press `9` - Release constraints

---

## Differences Summary

| Feature | vis_q_mj.py | Sim2Sim Deployment |
|---------|-------------|-------------------|
| **What it shows** | Reference motion playback | Policy execution |
| **Uses policy?** | ❌ No, just plays recorded motion | ✅ Yes, runs your trained policy |
| **Interactive?** | Limited (pause, reset) | Full control (keyboard, commands) |
| **Physics?** | Forward kinematics only | Full physics simulation |
| **Purpose** | Verify motion data quality | Test policy performance |

---

## Troubleshooting

### If you get "No module named 'rclpy'"
You need ROS2 installed in the asap_deploy environment (see Step 1).

### If you get "Cannot find locomotion model"
You need both the locomotion policy and your motion tracking policy. The locomotion policy is provided at `sim2real/models/dec_loco/`.

### If MuJoCo viewer doesn't show up
Make sure you're running in a graphical environment (not over SSH without X11 forwarding).

---

## Expected Result

When you activate your Jump3 policy (`[` key), you should see the G1 robot perform the jump motion in the MuJoCo simulator! The policy will:
1. Track the reference motion phase
2. Generate actions based on observations
3. Execute the learned jumping behavior

This is how you verify your policy works before deploying to real hardware! 🚀

