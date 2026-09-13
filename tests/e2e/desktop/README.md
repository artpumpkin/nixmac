# Shared desktop runtime recipe

`recipe.json` is the Nix Mac scenario definition consumed by `@repo/app-testing`
in the agents repository. It runs on a leased GUI session from `@repo/desktop`.
The runtime verifies the supplied DMG SHA-256 before installing
`/Applications/nixmac.app`, records continuous video, captures screenshots,
and retains evidence outside the disposable VM.

Configure the platform's `DESKTOP_TEST_RECIPE_FILES` with this JSON file's
absolute path. Submit `appId: "nixmac"`, profile `nixmac-prepared`, a candidate
containing the exact release asset URL and SHA-256, and scenario IDs `launch`
or `package-tools`. With a baseline build, the workflow runs the candidate,
baseline, and candidate confirmation sequentially in three fresh VMs.

For a controlled starting state, assign the contents of
`fresh-onboarding.initial-state.json` to the request's `initialState` field.
For the interrupted-onboarding/missing-configuration case, use
`startup-recovery.initial-state.json` with `scenarioIds: ["startup-recovery"]`.
The runtime installs the verified build, writes the supplied JSON to the
recipe's named `stateFiles`, then prepares and launches the app. Each comparison
phase receives the same starting JSON in a fresh VM. These are complete file
replacements; omitted target names retain the VM image's existing state.

The recovery scenario waits for the app's actual `Configuration not found`
repair view and the supplied path. It also checks that the app changed
`completedAt` from `null` to the seeded `lastBuildAt`, and retains that JSON
alongside screenshots and video. The historical build timestamp is a fixture;
this scenario exercises startup recovery and does not execute a Nix build.
The [state guide](STATE.md) maps the files to their production owners,
describes migrations, and distinguishes persisted state from live loading.

The launch assertion waits for the rendered `Config Directory` onboarding
content for up to 30 seconds, retaining each observation. A process or empty
native window alone does not satisfy the assertion.

The prepared profile has working Nix and Homebrew and preconfigured desktop
permissions. These scenarios establish install/launch and package-tool
readiness. The existing `tests/e2e` and Computer Use suites remain the source
of deeper product scenarios. This recipe does not claim clean package-manager
installation coverage or permission-prompt coverage.

Reports remain internal to the testing runtime. Public reproduction delivery
must use the existing trusted publisher and its live app attestation,
provider-trace, screenshot, and video requirements.
