# Error Reporting rollout

`Plugins/ErrorReporting.xml.pending` is an inactive registration draft. Its empty `Commit`
is intentional: the implementation and matching PluginSdk changes require coordinated review and release.
The original error-reporting main branch contains a template, so pinning that revision would not provide
the capture API required by the new launcher. No future or nonexistent commit is represented here.
The hub's XML scan ignores the `.pending` extension.

Activate the registration only after these steps:

1. Merge and publish the reviewed `CometWorks/error-reporting` server implementation with
   `Preloader.Initialize` and bounded local capture. Record its actual complete commit SHA.
2. Merge the matching Magnetar SDK/launcher changes, including `Logger.EntryEmitted`, early
   required-plugin initialization and the System.Text.Json compiler reference. Prepare the
   corresponding launcher release without exposing a launcher whose required plugin is unavailable.
3. Replace the empty `Commit` in the draft with the real merged plugin SHA, verify that source
   builds on the matching SDK for .NET Framework and CoreCLR, and rename it to `ErrorReporting.xml`.
   Run `python3 test.py Plugins/` and review the manifest through the normal hub process.
   From the matching Magnetar source checkout, run
   `bash Build/validate-error-reporting.sh /path/to/error-reporting` to compile both targets
   and record exact SDK/plugin hashes. Compilation on Linux does not replace the Windows
   .NET Framework runtime check; the Magnetar Windows workflow runs both SDK test targets.
4. Publish the hub registration before releasing the launcher that requires it. Ship the same
   compiled plugin and matching SDK in offline/managed deployment bundles; test clean-install and
   offline startup so the mandatory plugin never relies on an unavailable source pin.

The source directory deliberately includes only `ServerPlugin`: no client template, tests or
Diagnostics transport package belongs in the server plugin compilation. The implementation uses
Magnetar's System.Text.Json reference; it has no network uploader or native dump generator.
Required installation does not grant external diagnostic or dump consent.

## LinuxCompat native-wrapper rollout

The wrapper build workflow assigns a new `v1.0.<run_number>` release on merge; do not
reuse v1.0.51 (the current latest release) or the older v1.0.46 asset currently pinned
in the hub. The diagnostics PR adds optimized runtime archives, matching split-symbol
archives and `release-manifest.json`. The native-wrapper PR must land before this
rollout is completed.

After its public release exists, take the SE1 runtime asset URL and SHA-256 from
that release's manifest and update `NativeWrappers` in `Plugins/LinuxCompat.xml` and verify the downloaded
archive's hash. `Plugins/LinuxCompatLegacyId.xml` is a compatibility registration
without a native asset declaration; preserve that contract rather than adding a
second asset loader just for diagnostics. Retain `se1-native-wrappers.symbols.tar.gz` from
the same release for analysis; a Debug build or another release's symbols will not
match. Never use a local rebuild's hash as the checksum of a future release.

LinuxCompat already loads these libraries through the existing asset contract, and
Quasar.Host captures the exact supervised process identity independently. No
LinuxCompat source/version bump is required merely to consume a newer native-wrapper
asset. If LinuxCompat source later changes, publish and pin that source separately.
Until the new wrapper release is available, this draft preserves the existing
working LinuxCompat manifests and must remain a rollout prerequisite.
