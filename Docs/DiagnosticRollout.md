# Error Reporting rollout

`Plugins/ErrorReporting.xml` pins the actual reviewed error-reporting implementation
`4629caff434e35312e8c99c776e364e392142d5d`, merged in
[error-reporting PR #2](https://github.com/CometWorks/error-reporting/pull/2).
This registration remains in a draft PR while the matching SDK is reviewed.
The source was compiled against the coordinated SDK for both net48 and net10.0;
the original template revision is not used.

Complete rollout in this order:

1. Review the matching Magnetar 2.4.3.2 SDK/launcher changes, including
   `Logger.EntryEmitted`, early required-plugin initialization and the System.Text.Json
   compiler reference. Avoid publishing the mandatory-plugin launcher before its
   hub registration is available.
2. Run `python3 test.py Plugins/` and review the manifest through the normal hub
   process. From the matching Magnetar checkout run
   `bash Build/validate-error-reporting.sh /path/to/error-reporting` to compile both
   targets and record exact SDK/plugin hashes. Linux compilation does not replace
   Windows .NET Framework runtime testing.
3. Publish this hub registration before releasing the launcher that requires it.
   Ship the same compiled plugin and matching SDK in offline/managed bundles;
   verify clean-install and offline startup before exposing the new launcher.

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
