![version](https://img.shields.io/badge/version-20%2B-E23089)
![platform](https://img.shields.io/static/v1?label=platform&message=mac-intel%20|%20mac-arm%20|%20win-64&color=blue)

# HDI_Get process activity

Builds a live process and user monitor on top of `Process activity` (formerly `Get process activity`), which returns the running processes and open sessions as an object. Originally published by 4D as a **HDI** (*How Do I*) example for **4D v16 R4 / v17**; converted from the binary `.4DB` to the `.4DProject` architecture so it runs on current 4D releases.

## What it demonstrates

- Snapshotting all running processes and sessions with `Process activity`, then unpacking the `processes` and `sessions` arrays with `OB GET ARRAY`.
- Refreshing two list boxes (users and processes) on a `SET TIMER` tick, so the monitor updates continuously.
- Aggregating per-session CPU time and CPU usage by matching each process's `sessionID` back to its session (`CpuActivitiesCalc`).
- Reading process fields defensively through a `GetProperty` helper that returns a typed default when a key is absent.
- Mapping raw numeric process type/state codes and host types to human-readable strings and OS icons.
- Publishing the same process snapshot as JSON over HTTP via a web-published method (`WebGetJSONData`) and `WEB SEND TEXT`.
- Running the snapshot on the server in client/server mode via an `executedOnServer` method, versus locally in standalone mode.

## Key commands

| Command | Used for |
|---|---|
| `Process activity` | Snapshot running processes and sessions as an object |
| `OB GET ARRAY` | Extract the `processes` / `sessions` object arrays |
| `Application type` | Choose the server vs standalone code path |
| `SET TIMER` | Drive periodic refresh of the monitor list boxes |
| `WEB SEND TEXT` | Return the process list as JSON to a browser |
| `JSON Stringify array` | Serialise the collected process objects for the web output |
| `Time string` | Format cumulative `cpuTime` values for display |

## How it works

The landing form `HDI` is the standard splash screen (`ObjectMethods/BtnDemo.4dm` opens the demo). The demo is the form `HDI2`.

`Forms/HDI2/method.4dm` calls `InitInfo` and `InitPicturesHostType` on load and starts a 2-second timer (`SET TIMER(120)`). On each tick it refreshes the users list (`UpdateUsers`) on page 2 and the processes list (`UpdateProcess`) on page 3; the `Tab Control` object method refreshes immediately on click.

`Methods/UpdateProcess.4dm` is the core read path: it calls `Process activity` (or `GetProcessActivityOnServer` when `Application type=4`), pulls the `processes` array with `OB GET ARRAY`, and for each process reads `name`, `sessionID`, `type`, `number`, `state`, `cpuTime` and `cpuUsage` through `GetProperty`, converting codes with `ProcessTypeToString` / `ProcessStateToString` and resolving the owning session name with `GetSessionName`. The arrays are pushed into the list box via `COPY ARRAY`.

`Methods/UpdateUsers.4dm` does the same for sessions, first calling `CpuActivitiesCalc` to sum each remote session's CPU time/usage from its processes, and mapping `hostType` to an OS icon with `HostTypeToPicture`.

`Methods/WebGetJSONData.4dm` (published to the web) rebuilds the process list into plain objects and returns it with `WEB SEND TEXT(JSON Stringify array(...); "application/json")`; the form's `Button` opens `localhost` to hit it.

## Points of interest

- The command is `Process activity` in current 4D; it was named `Get process activity` in v16/v17 (hence the repo/blog title). The source already uses the modern name.
- In client/server mode the snapshot must run on the server: `GetProcessActivityOnServer` is flagged `executedOnServer`, while standalone falls through to a direct `Process activity` call, selected by `Application type=4`.
- `cpuUsage` is a fraction; the UI multiplies by 100 and `Round`s it to a percentage.
- Sessions carry no CPU totals of their own -- `CpuActivitiesCalc` synthesises `cpuTime` / `cpuUsage` by walking the processes and matching `sessionID`, and only for `type="remote"` sessions.
- `GetProperty` guards against missing keys by returning a typed default (`""`, `0`, or `False`), so the display code never faults on an absent field.
- The user monitor is only meaningful under 4D Server; in standalone mode `UpdateUsers` just shows an error text object.

## Modernisation notes

Converted from the binary `.4DB` to a 4D project. The following branch tracks the modernisation work.

| Branch | Description | Instructions |
|--------|-------------|--------------|
| [`miyako-modernize-4d-project`](../../tree/miyako-modernize-4d-project) | Modernizes 4D project methods, startup flow, localization, menu actions, and theme-aware listbox/button styling, plus README reporting. | [method.visibility.instructions.md](.github/instructions/method.visibility.instructions.md), [localisation.instructions.md](.github/instructions/localisation.instructions.md), [variable.declarations.instructions.md](.github/instructions/variable.declarations.instructions.md), [menu.instructions.md](.github/instructions/menu.instructions.md), [startup.instructions.md](.github/instructions/startup.instructions.md), [css.instructions.md](.github/instructions/css.instructions.md), [listbox.instructions.md](.github/instructions/listbox.instructions.md), [tahoe.css.instructions.md](.github/instructions/tahoe.css.instructions.md), [readme.branches.instructions.md](.github/instructions/readme.branches.instructions.md) |

## References

- [4D blog: Create your own process and user monitoring](https://blog.4d.com/create-your-own-process-and-user-monitoring/)
- [4D documentation: Process activity](https://developer.4d.com/docs/commands/process-activity)
- Original download: [HDI_Get process activity.zip](https://download.4d.com/Demos/4D_v16_R4/HDI_Get%20process%20activity.zip)
- Index of v16/v17 HDIs: [miyako/4d-hdi](https://github.com/miyako/4d-hdi)

## Screenshots

<img width="360" height="352" alt="Screenshot 2026-07-24 at 12 27 07" src="https://github.com/user-attachments/assets/0b9d79ac-9bf3-4545-aebc-23cd2b9d47b7" />
<img width="1070" height="592" alt="Screenshot 2026-07-24 at 12 27 10" src="https://github.com/user-attachments/assets/d735842a-976a-4377-8841-0560d32eef58" />
<img width="1070" height="592" alt="Screenshot 2026-07-24 at 12 27 14" src="https://github.com/user-attachments/assets/653f78fc-c57a-4d49-be8e-dd4a4b3f6b40" />
<img width="1070" height="592" alt="Screenshot 2026-07-24 at 12 34 23" src="https://github.com/user-attachments/assets/993081e2-dd3e-4de8-9575-47f2ac18fc2f" />
