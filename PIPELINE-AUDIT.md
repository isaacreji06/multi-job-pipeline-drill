# Pipeline Audit

## lint
- Intended purpose: validate the codebase with ESLint before any other work runs.
- Current issue: it has no dependency on any preceding job, so it is not explicitly ordered with the rest of the pipeline.
- Correct fix: keep it as the first job and make the later validation jobs depend on it.

## unit-tests
- Intended purpose: run the unit test suite after lint has passed.
- Current issue: it starts in parallel with no dependency on lint or build, so it does not reflect the intended validation order.
- Correct fix: make it run only after lint succeeds.

## build
- Intended purpose: produce the distributable build output in dist/ for downstream jobs.
- Current issue: it does not upload the build artifact, so later jobs cannot consume the generated files.
- Correct fix: upload the dist/ directory as an artifact named app-build.

## integration-tests
- Intended purpose: verify the built app artifact exists and is usable.
- Current issue: it runs without waiting for the build job, and it never downloads the build artifact, so the integration test will fail.
- Correct fix: make it depend on build and download the app-build artifact before running tests.

## deploy-staging
- Intended purpose: deploy the validated build to staging on the main branch.
- Current issue: it runs on every push and has no dependency on the validation jobs, so it can deploy before checks have completed.
- Correct fix: require unit-tests and integration-tests to succeed, restrict execution to the main branch, and add a timeout.

## deploy-production
- Intended purpose: deploy the validated build to production after staging succeeds.
- Current issue: it runs on every push and does not wait for the staging deployment or the validation jobs.
- Correct fix: require deploy-staging to succeed, restrict execution to the main branch, and add a timeout.

## notify
- Intended purpose: report the overall pipeline result regardless of success or failure.
- Current issue: it is skipped when an upstream job fails because it lacks always() behavior.
- Correct fix: run it with if: always() so it always reports the final pipeline state.