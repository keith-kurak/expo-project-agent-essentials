# expo-project-agent-essentials

All the stuff I consider important when setting up an Expo project.

This repo is a reference for agents. In a new Expo project, tell the agent:

> Apply the defaults from `~/github/expo-project-agent-essentials`. Follow its `SETUP.md`.

## What you get

- **EAS Build and EAS Update**, with `runtimeVersion` policy `appVersion`
- **Bundle IDs**: `com.keithkurak.<slug>`
- **App variants**: development (default), preview, production. Set by `APP_VARIANT`. Each variant has its own bundle ID suffix (`.dev`, `.preview`) and name under the icon (`<ABBR>-DEV`, `<ABBR>-PREVIEW`).
- **eas.json profiles**: `development`, `development-simulator`, `preview`, `production`
- **EAS workflows**
  - Push to `main` → Android preview build or update. Changes only in hidden folders do not trigger it.
  - PR → publish an update and comment with a QR code
  - Manual → iOS simulator + Android device development builds
- **Agent skills** for the EAS cloud simulator
  - `eas-sim-dev`: develop on an iOS cloud simulator with Metro, driven by agent-device
  - `eas-sim-verify-pr-ios`: validate a PR on iOS, comment with a link to the run
  - `eas-sim-preview-link`: comment an iOS web preview link on a PR
  - `eas-sim-verify-pr-android`: on-demand PR validation on Android, comment with a link to the run
- **README format**: description, start the dev server, features (short bullets), TODO (next major goals)

## Layout

| Path | Contents |
| --- | --- |
| `SETUP.md` | Step-by-step instructions for the agent |
| `templates/` | `app.config.js`, `eas.json`, `README.md`, `.eas/workflows/` |
| `skills/` | Skills to copy into the project's `.agents/skills/` |

## Sources

- App variants: from `pancake-theory`, but with development as the default and `appVersion` runtime policy
- Simulator scripts: from the `eas-sim-dev` skill in `ff4-free-enterpriser`
- PR validation flow: from the `agent-verify` workflow in `pancake-theory`
