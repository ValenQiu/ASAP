# Integrating Third-Party Repos with Different isaac_utils Paths

## Problem

When integrating code from another repository that uses `isaac_utils`, you may encounter import errors due to:
1. **Different file locations**: 
   - ASAP: `./ASAP/isaac_utils/` 
   - Other repo: `humanoidverse/isaac/`
2. **Missing functions**: Functions that exist in one version but not the other

## What Was Fixed

### Error Encountered
```
ImportError: cannot import name 'calc_yaw_heading_quat_inv' from 'isaac_utils.rotations'
```

### Root Cause
The file `humanoidverse/deploy/urcirobot.py` was importing `calc_yaw_heading_quat_inv`, which didn't exist in ASAP's version of `isaac_utils`.

### Solution Applied ✅
Added the missing function to `/home/qiulm/sources/ASAP/isaac_utils/isaac_utils/rotations.py`:

```python
@torch.jit.script
def calc_yaw_heading_quat_inv(yaw, w_last: bool = True):
    # type: (Tensor, bool) -> Tensor
    """
    Calculate heading quaternion inverse from yaw angle
    Args:
        yaw: yaw angle (scalar or tensor)
        w_last: whether the quaternion format is [x, y, z, w] (True) or [w, x, y, z] (False)
    Returns:
        heading quaternion inverse
    """
    axis = torch.zeros(yaw.shape[0], 3, dtype=yaw.dtype, device=yaw.device)
    axis[..., 2] = 1  # z-axis
    heading_q = quat_from_angle_axis(-yaw, axis, w_last=w_last)
    return heading_q
```

---

## General Solutions for isaac_utils Integration Issues

### Option 1: Add Missing Functions (Recommended) ✅
**When to use:** You have a few missing functions

**Steps:**
1. Identify the missing function from the import error
2. Check the other repo's `isaac_utils` for the function implementation
3. Add it to ASAP's `isaac_utils/isaac_utils/rotations.py`
4. Make sure to use `@torch.jit.script` decorator for consistency

**Pros:** Clean, maintains ASAP's structure, no path conflicts  
**Cons:** Need to manually port each missing function

---

### Option 2: Replace ASAP's isaac_utils
**When to use:** Many functions are different/missing, or major version mismatch

**Steps:**
```bash
# Backup ASAP's version
cd /home/qiulm/sources/ASAP
mv isaac_utils isaac_utils_backup

# Copy from the other repo
cp -r /path/to/other/repo/humanoidverse/isaac ./isaac_utils

# Reinstall
cd isaac_utils
pip install -e .
```

**Pros:** Gets all functions at once  
**Cons:** May break ASAP's existing code if there are API differences

---

### Option 3: Use Both Versions with Different Import Paths
**When to use:** You want to keep both versions available

**Steps:**
1. Keep ASAP's `isaac_utils` at `./ASAP/isaac_utils/`
2. Copy the other version to `./ASAP/isaac_utils_alt/`
3. In your integration code, use specific imports:
```python
from isaac_utils.rotations import calc_heading_quat  # ASAP version
from isaac_utils_alt.rotations import calc_yaw_heading_quat_inv  # Other version
```

**Pros:** Both versions available  
**Cons:** More complex, potential for confusion

---

### Option 4: Modify Import Paths in Integrated Code
**When to use:** The other repo expects `humanoidverse.isaac` path

**Steps:**
Create a symlink or modify Python path:
```bash
cd /home/qiulm/sources/ASAP/humanoidverse
ln -s ../isaac_utils isaac
```

Then the other repo's imports like `from humanoidverse.isaac.rotations import ...` will work.

**Pros:** Minimal code changes  
**Cons:** Can cause confusion with multiple paths to same code

---

## How to Identify Missing Functions

### Method 1: Check the Import Error
The error message tells you exactly what's missing:
```
ImportError: cannot import name 'calc_yaw_heading_quat_inv' from 'isaac_utils.rotations'
```

### Method 2: Compare Function Lists
```bash
# List all functions in ASAP's version
grep "^def " /home/qiulm/sources/ASAP/isaac_utils/isaac_utils/rotations.py

# List all functions in the other repo's version
grep "^def " /path/to/other/repo/humanoidverse/isaac/rotations.py

# Compare them
diff <(grep "^def " /home/qiulm/sources/ASAP/isaac_utils/isaac_utils/rotations.py | sort) \
     <(grep "^def " /path/to/other/repo/humanoidverse/isaac/rotations.py | sort)
```

### Method 3: Check All Imports in the File
```bash
# Find all imports from isaac_utils in your integration code
grep "from isaac_utils" /home/qiulm/sources/ASAP/humanoidverse/deploy/urcirobot.py
```

---

## Testing the Fix

After adding missing functions, test:

```bash
cd /home/qiulm/sources/ASAP
conda activate hvgym

# Test the specific import
python -c "from isaac_utils.rotations import calc_yaw_heading_quat_inv; print('✅ Import successful!')"

# Test the full integration file
python humanoidverse/urci.py --help
```

---

## Current Status

✅ **Fixed:** `calc_yaw_heading_quat_inv` has been added to ASAP's `isaac_utils`  
✅ **Verified:** `humanoidverse/urci.py` imports successfully  
✅ **Location:** `/home/qiulm/sources/ASAP/isaac_utils/isaac_utils/rotations.py` (line 293-306)

---

## Future Integration Tips

1. **Always check imports first** before running integrated code
2. **Use virtual environments** to avoid package conflicts
3. **Keep a backup** of ASAP's original `isaac_utils` 
4. **Document changes** when adding functions from other repos
5. **Test thoroughly** - some functions may have subtle API differences (e.g., `w_last` parameter)

---

## Function Comparison Reference

| Function | ASAP isaac_utils | Typical Other Repo | Notes |
|----------|------------------|-------------------|-------|
| `calc_heading_quat` | ✅ Yes | ✅ Yes | Standard |
| `calc_heading_quat_inv` | ✅ Yes | ✅ Yes | Standard |
| `calc_yaw_heading_quat_inv` | ✅ **Added** | ✅ Yes | Takes yaw angle, not quat |
| `quat_mul` | ✅ Yes | ✅ Yes | Standard |
| `my_quat_rotate` | ✅ Yes | ✅ Yes | Standard |

Your integration is now ready to use! 🎉


