# butterflymx-mule-ci-cd-workflows

Shared, reusable GitHub Actions workflows for ButterflyMX MuleSoft apps deploying
to CloudHub 2.0. Consumer apps keep their own thin per-branch orchestrator
workflows and call into these via `uses:`.

## Workflows

- `build_workflow.yml` — strips `-SNAPSHOT`, publishes a uniquely-versioned
  artifact to Anypoint Exchange, exposes `RELEASE_VERSION` as a job output.
- `deploy_workflow.yml` — re-applies the exact `RELEASE_VERSION` published by
  the build job and deploys it to CloudHub 2.0.

## Usage from an app repo

```yaml
jobs:
  build:
    uses: runslikebutter/butterflymx-mule-ci-cd-workflows/.github/workflows/build_workflow.yml@v1
    with:
      GITHUB_ENVIRONMENT: "Sandbox"
    secrets:
      CONNECTED_APP_CLIENT_ID: "${{ secrets.CONNECTED_APP_CLIENT_ID }}"
      CONNECTED_APP_CLIENT_SECRET: "${{ secrets.CONNECTED_APP_CLIENT_SECRET }}"

  deploy:
    needs: build
    uses: runslikebutter/butterflymx-mule-ci-cd-workflows/.github/workflows/deploy_workflow.yml@v1
    with:
      GITHUB_ENVIRONMENT: "Sandbox"
      RELEASE_VERSION: "${{ needs.build.outputs.RELEASE_VERSION }}"
      ANYPOINT_ENV: "Sandbox"
      MULE_ENV: "dev"
      APPLICATION_NAME: "<app-name>-dev"
      REGION: "cloudhub-us-east-2"
    secrets:
      CONNECTED_APP_CLIENT_ID: "${{ secrets.CONNECTED_APP_CLIENT_ID }}"
      CONNECTED_APP_CLIENT_SECRET: "${{ secrets.CONNECTED_APP_CLIENT_SECRET }}"
      DECRYPTION_KEY: "${{ secrets.DECRYPTION_KEY }}"
```

The calling app's own repo needs a `.maven/settings.xml` with the
`Repository` server entry (username `~~~Client~~~`, password
`${client.id}~?~${client.secret}`) and a `pom.xml` with a matching
`cloudhub2Deployment` block — these two workflows only invoke `mvn`, they
don't generate that config.

## Versioning

Pin consumers to a tag (`@v1`, `@v1.1`, ...), never `@main` — a change here
would otherwise immediately affect every app's pipeline with no chance to
opt in.

## Access

If this repo is private, the consuming repos need to be allow-listed under
this repo's Settings > Actions > General > Access, or the org needs to
permit cross-repo reusable workflow calls.
