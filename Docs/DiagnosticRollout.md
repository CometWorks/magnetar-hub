# Error Reporting rollout

`Plugins/ErrorReporting.xml` pins the actual reviewed error-reporting implementation
`4629caff434e35312e8c99c776e364e392142d5d`, merged in
[error-reporting PR #2](https://github.com/CometWorks/error-reporting/pull/2).
The registration and matching Magnetar SDK/launcher have been merged.
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

The merged wrapper diagnostics changes are published as **v1.0.55**, source commit
`393f5b07cb30ef494c4b955fbbb39864a39561f7`. `Plugins/LinuxCompat.xml` pins
`https://github.com/CometWorks/linux-native-wrappers/releases/download/v1.0.55/se1-native-wrappers.tar.gz`
with SHA-256 `39635a2c33a28cf0c5fafe9a2ba7d1224caf77994b79de1dfb3ba6a75e30005c`.
The downloaded runtime archive matches both the release manifest and the GitHub
asset digest. It replaces the previous v1.0.46 asset.

Retain the same release's `se1-native-wrappers.symbols.tar.gz` for crash analysis;
its SHA-256 is `2e078a9bb3fde3f077b59e8c1c5b9b1a5c98965daa08c547092b4b716fb4afbe`.
The symbols match the optimized runtime build; another release or a Debug build
is unsuitable. `Plugins/LinuxCompatLegacyId.xml` remains a compatibility
registration without a native asset declaration.

LinuxCompat already loads these libraries through the existing asset contract, and
Quasar.Host captures the exact supervised process identity independently. No
LinuxCompat source/version bump is required merely to consume a newer native-wrapper
asset. If LinuxCompat source later changes, publish and pin that source separately.
Publish this hub asset-pin update to roll the released native binaries into LinuxCompat installations.
