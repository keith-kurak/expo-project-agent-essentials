# Apply these defaults to an Expo project

Instructions for an agent. Do the steps in order, from the root of the **target** project. The files to copy are in `templates/` and `skills/` in this repo.

Some steps change the user's EAS account (project creation, environment variables) or cost money (builds). Do those only when the user asked for this setup. Ask before you start a build.

## 0. Collect the values

| Value | How to get it | Example |
| --- | --- | --- |
| `<slug>` | `slug` in `app.json` | `pancake-theory` |
| `<SLUGID>` | `<slug>` in lowercase, with all characters that are not a–z or 0–9 removed | `pancaketheory` |
| `<ABBR>` | Short abbreviation of the app name, 2–4 capital letters. Ask the user if they did not give one. | `PK` |
| EAS account | `keithco`, unless the user says a different one | `keithco` |

## 1. Install packages

```bash
npx expo install expo-dev-client expo-updates
```

## 2. Bundle ID and package

In `app.json`, set:

- `expo.ios.bundleIdentifier`: `com.keithkurak.<SLUGID>`
- `expo.android.package`: `com.keithkurak.<SLUGID>`
- `expo.scheme`: `<SLUGID>`, if there is no scheme yet

These are the production identifiers. `app.config.js` (step 5) adds `.dev` and `.preview` for the other variants.

## 3. EAS init, EAS Build, EAS Update

Do this step **before** you add `app.config.js`. The EAS CLI can write to `app.json`, but not to a dynamic config.

```bash
npx --yes eas-cli@latest init --account keithco --non-interactive
npx --yes eas-cli@latest update:configure --non-interactive
```

Then make sure that `app.json` has:

- `expo.owner` and `expo.extra.eas.projectId` (from `init`)
- `expo.updates.url`: `https://u.expo.dev/<projectId>` (from `update:configure`)
- `expo.runtimeVersion`: `{ "policy": "appVersion" }`. Set it if `update:configure` wrote a different value.

## 4. eas.json

Copy `templates/eas.json` to `eas.json`. Replace the existing file if there is one, but keep any `submit` settings that are already in it.

Profiles:

| Profile | Use | Channel | EAS environment |
| --- | --- | --- | --- |
| `development` | Dev client for devices (Android APK, iOS ad hoc) | `development` | `development` |
| `development-simulator` | Dev client for the iOS simulator and EAS cloud simulator | `development` | `development` |
| `preview` | Internal test app | `preview` | `preview` |
| `production` | Store app | `production` | `production` |

## 5. App variants

1. Copy `templates/app.config.js` to `app.config.js`. Replace `<ABBR>`.
2. Remove `"expo-dev-client"` from `plugins` in `app.json` if it is there. `app.config.js` adds it.
3. Create the `APP_VARIANT` EAS environment variables. Do **not** create one in `development`: development is the default.

   ```bash
   npx --yes eas-cli@latest env:create --environment preview --name APP_VARIANT --value preview --visibility plaintext --non-interactive
   npx --yes eas-cli@latest env:create --environment production --name APP_VARIANT --value production --visibility plaintext --non-interactive
   ```

4. Check each variant:

   ```bash
   npx expo config --type public | grep -E "name|bundleIdentifier|package"
   APP_VARIANT=preview npx expo config --type public | grep -E "name|bundleIdentifier|package"
   APP_VARIANT=production npx expo config --type public | grep -E "name|bundleIdentifier|package"
   ```

   | `APP_VARIANT` | Name under the icon | Bundle ID / package |
   | --- | --- | --- |
   | unset | `<ABBR>-DEV` | `com.keithkurak.<SLUGID>.dev` |
   | `preview` | `<ABBR>-PREVIEW` | `com.keithkurak.<SLUGID>.preview` |
   | `production` | `expo.name` | `com.keithkurak.<SLUGID>` |

Why EAS environment variables and not `env` in `eas.json`: the workflow `fingerprint` and `update` jobs read the EAS environment. With the variable in one place, builds, updates, and fingerprints always use the same variant.

## 6. EAS workflows

Copy `templates/.eas/workflows/` to `.eas/workflows/`. Replace `<ABBR>` in `update-on-pr.yaml`.

| File | Trigger | What it does |
| --- | --- | --- |
| `build-or-update-preview-android.yaml` | Push to `main`, except changes only in hidden folders | Android preview: publish an update if a build has the same fingerprint, else make a new build |
| `update-on-pr.yaml` | PR opened or updated | Publish an update to the PR branch, comment on the PR with a QR code |
| `dev-builds.yaml` | Manual | iOS simulator development build + Android device development build |

Validate each file:

```bash
npx --yes eas-cli@latest workflow:validate .eas/workflows/<file>.yaml --non-interactive
```

The push and PR triggers and the PR comment need a GitHub repository connected to the EAS project (expo.dev → project → GitHub). Tell the user if it is not connected.

## 7. Simulator skills

Copy each folder in `skills/` to `.agents/skills/` in the project, and link it for Claude Code:

```bash
mkdir -p .agents/skills .claude/skills
for s in eas-sim-dev eas-sim-verify-pr-ios eas-sim-verify-pr-android eas-sim-preview-link; do
  cp -R "<this-repo>/skills/$s" .agents/skills/
  ln -sfn "../../.agents/skills/$s" ".claude/skills/$s"
done
chmod +x .agents/skills/eas-sim-dev/*.sh
```

Replace `<ABBR>` in `.agents/skills/eas-sim-verify-pr-android/SKILL.md`.

Add to `.gitignore` if not there:

```gitignore
.expo/
.env.eas-simulator
```

`.env.eas-simulator` holds a session token. Never commit it.

| Skill | Use |
| --- | --- |
| `eas-sim-dev` | Develop on an iOS cloud simulator with Metro and Fast Refresh, driven by agent-device. Has the shared scripts. |
| `eas-sim-verify-pr-ios` | Validate a PR on an iOS cloud simulator, comment with the result and a link to the run |
| `eas-sim-preview-link` | Start an iOS web preview of a PR, comment with the link |
| `eas-sim-verify-pr-android` | On-demand PR validation on an Android cloud emulator, comment with the result and a link to the run |

The PR skills need the `gh` CLI, logged in.

## 8. Project agent instructions

Add this section to the project's `AGENTS.md` (create it if there is none; if there is a `CLAUDE.md` with no `@AGENTS.md` line, add the section there):

```markdown
## App variants

`APP_VARIANT` selects the variant: unset = development (`<ABBR>-DEV`), `preview`, `production`.
EAS gets it from the EAS environment of the build profile. Do not add it to `eas.json`.
Runtime version policy: `appVersion`.

## Cloud simulator skills

- `eas-sim-dev`: run and drive the app on an iOS cloud simulator with Metro.
- `eas-sim-verify-pr-ios` / `eas-sim-verify-pr-android`: validate a PR and comment the result.
- `eas-sim-preview-link`: post an iOS web preview link on a PR.

## README

Keep `README.md` in this order: description, start the dev server, Features (short bullets), TODO (next major goals).
When a change adds a feature or completes a goal, update Features and TODO in the same change.
```

## 9. Project README

Format the project's `README.md` like `templates/README.md`. Use these sections, in this order:

1. **Title and description**: the app name, then one or two sentences about what the app does. No feature list here.
2. **Start the dev server**: only the commands to install packages and run `npx expo start`, and how to get a development build. Use the project's package manager (`bun`, `npm`, and so on). Do not explain variants, workflows, or skills here.
3. **Features**: short bullets, 3–8 words each. Only features that work now.
4. **TODO**: the next major goals, one bullet each. Not small fixes or bugs.

Rules:

- Keep the existing content that is still correct. Move it into these sections. Do not delete information without asking the user.
- If you do not know the description, features, or goals, ask the user. Do not make them up. For a new project from a template, write an empty Features list and ask for the TODO items.
- Keep the whole README short. Put long documentation in other files and link to it.
- Replace `<ABBR>`.

When features or goals change, update the Features and TODO sections in the same change.

## 10. First builds (ask first)

The simulator skills need development builds. If the user agrees:

```bash
npx --yes eas-cli@latest workflow:run .eas/workflows/dev-builds.yaml
```

## Final check

- [ ] `npx expo config` works for all three `APP_VARIANT` values, with the names and IDs in step 5
- [ ] `app.json` has `runtimeVersion: { "policy": "appVersion" }` and `updates.url`
- [ ] `eas.json` has the four profiles
- [ ] Each workflow passes `workflow:validate`
- [ ] Skills are in `.agents/skills/`, with links in `.claude/skills/`
- [ ] `.gitignore` has `.env.eas-simulator`
- [ ] `README.md` has the sections in step 9: description, start the dev server, features, TODO
- [ ] No `<ABBR>`, `<slug>`, or `<SLUGID>` placeholders are left: `grep -rn "<ABBR>\|<SLUGID>\|<slug>" app.config.js README.md .eas .agents`
