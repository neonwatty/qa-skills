# iOS Simulator Setup & Naming Conventions

## Naming Convention

| Pattern | Purpose | Example |
|---------|---------|---------|
| `{AppName}-Workflow-iPhone16` | Default workflow testing device | `Seatify-Workflow-iPhone16` |
| `{AppName}-Workflow-iPhone16-Pro` | Pro-specific feature testing | `Seatify-Workflow-iPhone16-Pro` |
| `{AppName}-Workflow-iPad` | iPad testing | `Seatify-Workflow-iPad` |

## Creating Simulators (Bash commands)

```bash
# Get the app/repo name
APP_NAME=$(basename $(pwd))

# List available device types
xcrun simctl list devicetypes | grep iPhone

# List available runtimes
xcrun simctl list runtimes

# Create project-specific iPhone 16 simulator
xcrun simctl create "${APP_NAME}-Workflow-iPhone16" "iPhone 16" iOS18.2

# Create project-specific iPhone 16 Pro simulator
xcrun simctl create "${APP_NAME}-Workflow-iPhone16-Pro" "iPhone 16 Pro" iOS18.2

# Erase simulator to clean state
xcrun simctl erase <udid>

# Delete simulator when done
xcrun simctl delete <udid>

# List all workflow simulators (to find project-specific ones)
xcrun simctl list devices | grep "Workflow-iPhone"
```

## Initialization Steps

1. **Determine the simulator name:** `{basename $(pwd)}-Workflow-iPhone16`
2. Call `list_simulators` to see available simulators
3. Search for existing project-specific simulator matching the naming pattern
4. If not found, create one via `xcrun simctl create`
5. Call `boot_simulator` with the UDID
6. Call `claim_simulator` with the UDID
7. Call `open_simulator` to ensure Simulator.app is visible
8. Optional: `xcrun simctl erase <udid>` for clean state (ask user first)
9. Take initial screenshot to confirm readiness
10. Store UDID for all subsequent operations
11. Record simulator info for reports: device name, iOS version, UDID, app name

## Storing Simulator State for Recovery

Store in first workflow task metadata:
```
metadata: {
    "simulatorUdid": "[UDID]",
    "simulatorName": "[AppName]-Workflow-iPhone16",
    "iosVersion": "18.2",
    "appName": "[App name]"
}
```
