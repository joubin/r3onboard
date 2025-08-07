# Fix for r3onboard ARM64 Compatibility on Radxa Systems

This document explains the issue and provides a solution for running r3onboard on ARM64 systems like the Radxa.

## The Problem

The r3onboard project fails to run on ARM64 systems (like your Radxa) with the error:

```
ValueError: dbus_fast._private.marshaller.Marshaller size changed, may indicate binary incompatibility. Expected 48 from C header, got 40 from PyObject
```

This happens because:

1. The project was locked to `dbus-fast` version 1.95.2 (via `bleak` dependency)
2. Version 1.95.2 doesn't have precompiled ARM64 wheels
3. When it tries to compile from source, it creates binary incompatible extensions

## The Solution

The newer `dbus-fast` versions (2.44.x) **do have ARM64 wheels** and work correctly on ARM64 systems.

### Step 1: Update Dependencies

The `pyproject.toml` has been updated to use compatible versions:
- `dbus-fast ^2.44.0` - has ARM64 wheels (fixes the binary incompatibility)
- `bleak ^0.22.0` - last stable 0.x version compatible with `bless` (bleak 1.x has breaking changes)

### Step 2: Update Your Environment

Run these commands on your Radxa system:

```bash
# Remove the old virtual environment
rm -rf .venv

# Update the lock file to use the new constraints (allow updates this time)
poetry lock

# Install with the new dependencies
poetry install

# Test that it works
poetry run r3onboard
```

### Step 3: Verify the Fix

After running the above commands, you should see that `dbus-fast` version 2.44.x is installed with proper ARM64 wheels:

```bash
poetry show dbus-fast
```

## Why This Works

The fix addresses three compatibility constraints:

1. **ARM64 compatibility**: `dbus-fast` 2.44.x has precompiled ARM64 wheels on PyPI
2. **bleak compatibility**: `bleak` 0.22.x supports `dbus-fast` 2.x 
3. **bless compatibility**: `bless` is incompatible with `bleak` 1.x due to module structure changes

The solution uses `bleak` 0.22.x (last stable 0.x series) which works with both `dbus-fast` 2.x and `bless`.

## Alternative: Manual Override

If you need to test this before the project is updated, you can manually add this to your `pyproject.toml`:

```toml
[tool.poetry.dependencies]
# ... existing dependencies ...
dbus-fast = "^2.44.0"  # Latest version with ARM64 wheels
bleak = "^0.22.0"      # Last 0.x version compatible with bless
```

Then run:
```bash
poetry lock
poetry install
```

## Verification

The fix is working if:
1. `poetry install` completes without errors
2. `poetry run r3onboard` starts without the marshaller error
3. `poetry show dbus-fast` shows version 2.44.x

## Technical Details

- The issue affects all ARM64 systems running Python with certain library versions
- `dbus-fast` 1.95.2 lacks ARM64 wheels, forcing source compilation
- Source compilation creates binary incompatible extensions on modern ARM64 systems
- `dbus-fast` 2.44.x includes proper ARM64 wheels that work correctly

This fix maintains compatibility with other architectures while solving the ARM64 issue. 