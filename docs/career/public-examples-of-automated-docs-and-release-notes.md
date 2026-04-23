# Public Examples of Automated Docs and Release Notes

Yes, several top engineering teams have publicly implemented major parts of this approach.

## What is proven in public today

1. Kubernetes
   1. Automated release notes from PR metadata and labels.
   2. Automated API and reference documentation generation from source and OpenAPI.
   3. Public references:
      1. https://github.com/kubernetes/release/tree/master/cmd/release-notes
      2. https://github.com/kubernetes-sigs/reference-docs

2. GitLab
   1. Structured changelog and release entry workflow tied to merge requests.
   2. Strong docs-as-code validation and CI gating.
   3. Public reference:
      1. https://docs.gitlab.com/ee/development/changelog.html

3. Python core ecosystem
   1. Fragment-based release notes collected from individual changes, then assembled automatically.
   2. Public reference:
      1. https://github.com/python/blurb

4. OpenStack
   1. Reno-based release note fragment collection and rendering pipeline.
   2. Public reference:
      1. https://docs.openstack.org/reno/latest/

5. GitHub platform capability
   1. Native auto-generated release notes from merged PRs and labels.
   2. Public reference:
      1. https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes

## Important reality check

1. Fully autonomous generation across all long-form content types such as Overview, Features, User Guide, Installation Guide, and API Changes with zero human review is still uncommon in public.
2. The leading pattern is hybrid:
   1. Automated extraction and classification
   2. Template-driven draft generation
   3. Human approval before publish

Your direction aligns with where strong teams are heading. This is the next maturity level beyond standard release-note automation.

## Optional next step

Compile this into a short benchmark matrix with columns such as Team, What is automated, Inputs used, Human gate, and Public links.
