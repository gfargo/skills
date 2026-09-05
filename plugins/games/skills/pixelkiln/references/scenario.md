# Scenario

Use Scenario when the project needs a hosted third-party model or a reusable
project-specific model. BFL Flux 2 Dev is live-tested through authentication,
CU preflight, paid generation, human review, and durable recovery. Other model
schemas remain unverified. Require a one-asset smoke before widening a batch.

## Safety boundary

- Require `SCENARIO_SDK_API_KEY` and `SCENARIO_SDK_API_SECRET`. Never print or
  commit either value.
- Require `providerOptions.scenario.modelId` and a positive per-asset
  `maxComputeUnits`.
- Run `doctor --dry-run` and `plan` before authenticated `doctor` or generation.
- Planning is offline. At submit, PixelKiln sends the identical request with
  `dryRun=true` and refuses missing, malformed, or over-ceiling CU quotes.
- Pass `--budget <compute-units>` for a Scenario-only run. In a mixed run, use
  `--budget scenario=<compute-units>` alongside every other provider ceiling.
- Leave `numOutputs` above one to the local `pick` workflow. Selection reuses an
  existing Scenario asset and must not trigger another generation.
- Treat `scenario://` job and asset references as the recovery contract.
  Temporary signed Scenario URLs must not remain in a settled lockfile.
- Verify the chosen model returns PNG. The MVP rejects other media formats,
  style-image uploads, animation, and tiles.

Scenario model parameters vary. The MVP supplies prompt, 128–2048px dimensions
in multiples of 16, one to four outputs, and optional seed. Put only documented
model inputs in `parameters`; PixelKiln rejects attempts to override its prompt,
dimensions, seed, output count, project routing, or CU ceiling.

PixelKiln validates the job, bytes, hashes, and recovery path. It does not prove
the output is good pixel art. Inspect one result at 1× before approving a batch.

See `docs/SCENARIO.md` for setup, manifest examples, unsupported account
operations, and the live validation checklist.
