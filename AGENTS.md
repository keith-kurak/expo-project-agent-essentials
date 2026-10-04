# Agent notes for this repo

This repo is a reference, not an app. Other Expo projects copy from it.

- **Applying the defaults to another project:** follow `SETUP.md`. Run the commands in the target project, not here.
- **Editing this repo:** keep `SETUP.md`, `README.md`, `templates/`, and `skills/` consistent. If you change a template or skill, update the tables in `SETUP.md` and `README.md`.
- Placeholders in templates and skills: `<ABBR>`, `<slug>`, `<SLUGID>`. `SETUP.md` step 0 defines them.
- The simulator scripts read the scheme, bundle ID, and project ID from the app config at run time. Do not hard-code project values in them.
- Validate workflow templates with `npx --yes eas-cli@latest workflow:validate <file> --non-interactive`, run from any linked Expo project that has the same build profiles.
