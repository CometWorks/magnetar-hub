# Error Reporting rollout

`Plugins/ErrorReporting.xml.pending` is an inactive registration draft. Its empty `Commit`
is intentional: the implementation and matching PluginSdk changes are still uncommitted work.
The existing error-reporting repository HEAD contains a template, so pinning it would not provide
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
