# PC Pro Biller

PC Pro Biller is a desktop tool to estimate the electricity cost of your computer in real time. The app samples onboard hardware sensors (CPU / GPU / other power sensors when available), converts instantaneous power (watts) into energy (kWh) over time, and multiplies by a user-provided price per kWh to compute running cost and historical usage.

This repository contains the project planning, design notes, and (eventually) the Windows prototype. The goal is a small, cross-platform utility that can run as a background tray/widget on Windows, macOS and Linux. This README focuses on the Windows-first prototype and the recommended stack.

## Key concepts

- Instantaneous power (W) is sampled periodically (e.g., once per second).
- Energy consumed during a sample interval dt (seconds): (W * dt) / 3,600,000 gives kWh.
- Cost during interval = energy_kWh * price_per_kWh.
- Totals are accumulated and stored in a local database for history, CSV export, and basic visualization.

## Features (planned)

- Live instantaneous power (W) readout.
- User-configurable price per kWh (currency symbol optional).
- Cumulative energy and cost counters (since app start / since reset / daily totals).
- Lightweight system tray (notification area) app with a small popup for details.
- History persisted to SQLite and CSV export.
- Sensor selection (auto-detect CPU/GPU/total) with fallbacks.
- Optional moving average smoothing and configurable sampling interval.

## Why this approach

- Onboard sensors provide a quick, low-cost estimate without external hardware, and many users already run sensor apps (HWiNFO, LibreHardwareMonitor). This tool aims for convenience and reasonable accuracy — not utility-bill-grade measurement.

## Windows-first recommended tech stack

- Language & UI: C# with .NET 7/8 using WinForms or WPF (WinForms for quick scaffolding; WPF/WinUI if you want a richer UI later).
- Sensor integration:
  - LibreHardwareMonitorLib (preferred for a .NET-native library) OR
  - HWiNFO64 shared memory interface (works well if the user runs HWiNFO; broader sensor coverage).
- Persistence: SQLite using `Microsoft.Data.Sqlite` or `System.Data.SQLite`.
- Packaging: `dotnet publish` single-file or MSIX/Inno Setup for installer.

Rationale: .NET + LibreHardwareMonitor is the fastest route to a working, native-feeling Windows app with direct access to sensor values.

## Project structure (planned)

- src/pcprobiller.ui (WinForms/WPF app)
- src/pcprobiller.core (calculation, sensor adapters, DB models)
- src/pcprobiller.sensors (LibreHardwareMonitor/HWiNFO adapters)
- docs/ (design notes, API)
- tests/ (unit tests for calculations and persistence)

Note: these folders will be created as the prototype is scaffolded.

## Build & run (Windows prototype - when implemented)

Prerequisites:

- .NET 7 or newer SDK installed (download from https://dotnet.microsoft.com/)
- If using HWiNFO backend: HWiNFO64 installed and running with shared memory enabled.

Typical steps (when repo contains the prototype):

```powershell
# restore packages
dotnet restore src/pcprobiller.ui

# build
dotnet build src/pcprobiller.ui -c Release

# run (debug)
dotnet run --project src/pcprobiller.ui
```

Packaging example:

```powershell
dotnet publish src/pcprobiller.ui -c Release -r win-x64 --self-contained false /p:PublishSingleFile=true -o publish
```

If using HWiNFO shared memory, run HWiNFO64 before starting the app (enable sensors-only mode with shared memory in HWiNFO settings).

## Sensors & limitations

- Many motherboards or PSU telemetry solutions can expose a 'total system power' sensor, but it's not universal. If a total sensor is not available, the app will try to sum available sources (CPU package, GPU) and document the gap.
- Sensor values are estimates made by device drivers or monitoring tools — they are suitable for trends and rough billing estimates, but not for exact utility billing.
- Laptops: battery discharge/charge makes measuring mains draw tricky. The app will attempt to use power sensors exposed by the platform; if none are available it will rely on accessible sensors and note the limitation.

## Calculation details

- Sample watts W at interval dt seconds.
- delta_kWh = (W * dt) / 3,600,000.
- delta_cost = delta_kWh * price_per_kWh.

Implementation tip: store totals in fixed-precision decimals where money is involved (e.g., decimal in C#) to avoid floating-point rounding problems.

## Roadmap

Short term (Windows prototype):
- Scaffold WinForms tray app with settings window.
- Integrate LibreHardwareMonitorLib adapter with a fallback HWiNFO path.
- Implement sampling loop, cost calculation, SQLite persistence, and CSV export.
- Provide simple daily summary UI and reset/export options.

Medium term (cross-platform):
- Define sensor adapter interface and implement macOS/Linux backends. On Linux, use lm-sensors, Intel RAPL (for CPU), and vendor tools for GPU (nvidia-smi). On macOS, use powermetrics and SMC readings where possible.
- Consider Tauri for a cross-platform UI or continue with native toolkits per OS.

Long term:
- Add background syncing to cloud (opt-in) for multi-device cost aggregation.
- Add small desktop widgets for each OS and platform-specific installers.

## Contributing

Contributions are welcome. If you want to help:

1. Open an issue describing the feature or bug.
2. Fork the repo and create a branch for your change.
3. Add tests for new behavior where applicable.
4. Submit a pull request with a clear description and screenshots if UI changes are involved.

## License

This repository will use an open-source license. If you have a preference (MIT, Apache-2.0), please indicate it or open an issue to discuss.

## Contact / discussions

Open issues for discussion on sensor backends, UI ideas, and platform-specific constraints. If you'd like, we can start by scaffolding the Windows prototype and shipping a minimal working build.
