# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Zuul CI jobs repository that contains job definitions, playbooks, and roles for continuous integration workflows. The repository is structured to integrate with Zuul CI/CD systems and leverages roles from the upstream zuul-jobs repository at opendev.org/zuul/zuul-jobs.

## Repository Structure

```
ci-jobs/
├── zuul.d/          # Zuul configuration files
│   ├── jobs.yaml    # Job definitions
│   └── projects.yaml # Project pipeline configuration
├── playbooks/       # Ansible playbooks for jobs
├── roles/           # Custom Ansible roles (for future use)
└── CLAUDE.md        # This file
```

## Key Concepts

### Zuul Configuration Files

**zuul.d/jobs.yaml**: Defines Zuul jobs
- Each job has a name, parent (typically `base`), description, and run playbook
- Jobs can reference roles from zuul-jobs via `required-projects`
- Example: `ci-jobs-validate-host` extends the base job

**zuul.d/projects.yaml**: Configures which jobs run in which pipelines
- `check` pipeline: Runs on new patchsets/PRs for initial validation
- `gate` pipeline: Runs on approved changes before merging
- Jobs listed here will be automatically triggered by Zuul

### Job Architecture

Jobs in this repository follow this pattern:
1. Job defined in `zuul.d/jobs.yaml` with metadata and playbook reference
2. Playbook in `playbooks/` that orchestrates roles
3. Roles either from upstream zuul-jobs or custom roles in `roles/`

### Integration with zuul-jobs

This repository uses roles from https://opendev.org/zuul/zuul-jobs:
- Reference roles by name in playbooks (e.g., `validate-host`)
- Add `required-projects: - opendev.org/zuul/zuul-jobs` to job definitions
- Zuul automatically makes these roles available during job execution

## Adding New Jobs

1. Create a playbook in `playbooks/your-job-name.yaml`
2. Define the job in `zuul.d/jobs.yaml` with:
   - Unique name (prefix with `ci-jobs-` by convention)
   - Parent job (usually `base`)
   - Description of what the job does
   - Reference to the playbook
3. Add the job to appropriate pipelines in `zuul.d/projects.yaml`

## Validation and Testing

### YAML Syntax Validation
Before committing changes, validate YAML syntax:
```bash
# Validate YAML syntax for all zuul.d files
yamllint zuul.d/

# Check Ansible playbook syntax
ansible-playbook --syntax-check playbooks/*.yaml
```

### Example Job Structure
From `zuul.d/jobs.yaml` - the standard job definition pattern:
```yaml
- job:
    name: ci-jobs-validate-host          # Unique job name
    parent: base                          # Inherit from base job
    description: |                        # Multi-line description
      What the job does...
    run: playbooks/ci-jobs-validate-host.yaml  # Playbook to execute
    required-projects:                    # External dependencies
      - opendev.org/zuul/zuul-jobs
```

### Current Jobs

**ci-jobs-validate-host**: Initial test job that validates the Zuul system works end-to-end by running the validate-host role, which logs system information (CPU, filesystem, network) about the build node.

## Important Notes

### Zuul Configuration Parsing
- Zuul reads all `.yaml` files in `zuul.d/` alphabetically
- Configuration is merged, so split files logically (jobs, projects, templates, etc.)
- Use `---` document separator at the start of YAML files
- Job and project names must be globally unique within the Zuul tenant

### Required Projects
When using roles from zuul-jobs:
- Always add `required-projects: - opendev.org/zuul/zuul-jobs` to job definitions
- Roles are automatically available in playbooks without additional imports
- Reference roles by their directory name (e.g., `validate-host`, not `zuul-jobs.validate-host`)

### Pipeline Behavior
- **check**: Runs in parallel on all open changes; can fail without blocking
- **gate**: Runs serially on approved changes; failure prevents merge
- Jobs must succeed in check before running in gate
- Same job definition runs in both pipelines unless variants are used

## Repository Information

- Main branch: `main`
- License: See LICENSE file for details
