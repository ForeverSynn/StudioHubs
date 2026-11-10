# Developer Notes

## API Endpoints

Configuration:

- GET /Plugins/StudioHubs/config
- POST /Plugins/StudioHubs/config

Studio Hubs data:

- GET/POST /Plugins/StudioHubs/studio-hubs/studios-manage
- GET/POST/DELETE /Plugins/StudioHubs/studio-hubs/collection
- GET/POST/DELETE /Plugins/StudioHubs/studio-hubs/video
- GET/DELETE /Plugins/StudioHubs/studio-hubs/logo
- GET /Plugins/StudioHubs/studio-hubs/logo/{fileName}
- GET /Plugins/StudioHubs/studio-hubs/video/{fileName}

Notes:

- Config GET is readable for dashboard loading reliability.
- Config POST remains admin-protected.

## Build

From the plugin root, run the local build/package script:

```sh
bash scripts/build-local.sh
```

The script publishes the plugin, checks the required files (and JSON syntax when `jq` is installed), creates a ZIP, and verifies the ZIP contents. It does not create a Git tag, modify release metadata, or publish a GitHub release.

Default outputs:

- `dist/local/StudioHubs-v<version>.zip` — installable plugin package
- `dist/local/StudioHubs-v<version>/` — unpacked package files
- `bin/Release/<target-framework>/publish/` — full .NET publish output

Optional arguments:

- `--configuration Debug` to use another build configuration
- `--output-dir PATH` to write artifacts somewhere else

To test the package, stop Jellyfin and move the existing Studio Hubs plugin folder out of Jellyfin's plugins directory (keep it as the backup; Jellyfin loads only one version of a plugin, and a second version folder left in place can be marked superseded or removed). Then extract the ZIP into a new plugin folder under the plugins directory and start the server. Review the Jellyfin log for plugin load errors. To roll back, stop Jellyfin, remove the test folder and move the backup folder back.

Local and release packages include:

- Jellyfin.Plugin.StudioHubs.dll
- Jellyfin.Plugin.StudioHubs.deps.json
- manifest.json
- meta.json
- StudioHubs.png

## Release Workflow

This repository includes a manually triggered GitHub Actions workflow at `.github/workflows/create_release.yml`.

When run from the Actions tab, it:

1. Computes the next `vX.Y.Z.W` tag (patch increment, or the optional `version` input) and writes it into the manifest, meta and project version.
2. Builds with `dotnet publish -c Release` and creates a ZIP release asset named after the tag (for example, `v1.0.12.0.zip`), with its checksum in the manifest.
3. Commits the updated metadata and project version and pushes `HEAD` to `main`, then creates and pushes the tag. Run it from `main` only: started from another branch it would push that branch's commits to `main` too. If the build or packaging fails, nothing is pushed.
4. Creates a GitHub release with that ZIP, using the `release_notes` input as release notes, or the manual-trigger message when it is empty.
