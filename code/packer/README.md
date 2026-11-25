# Packer audit module

Copy the block into a review that includes Packer.

````markdown
## Optional Packer module

<!-- BEGIN PACKER MODULE -->

Apply these checks to Packer within the current review scope. Work read-only, cite the revision and file plus line or symbol, and check for a guard or platform setting that could disprove each claim. Preserve the main audit’s finding IDs and evidence labels when supplied. Give the smallest sound fix and one decisive check. Distinguish inspected source, observed output, and open questions.

### PK1. Identify the image that ships

Start at the pipeline’s Packer command. Record template format, Packer and plugin versions, variable files and overrides, build name, selected sources, and target architecture, region, and account. Follow each `source` through its `build` block and `only` or `except` filters. Check which builders receive each provisioner. A passing build for one image family can leave another without its agent or configuration.

### PK2. Reconstruct image inputs

Trace a base image to its owner, identifier or filter, architecture, and release channel. Determine what a “most recent” filter can select on a later build. Inspect plugin constraints, downloaded checksums, package repositories, and version pins against the project’s reproducibility promise. Compare template-relative paths with runner working-directory paths. Check which scripts, files, and secrets the CI runner can actually supply in a clean checkout.

### PK3. Trace provisioning across the host and guest

Identify where each command runs, its user, interpreter, working directory, and environment. In particular, `shell-local` executes on the Packer host. For guest shell or PowerShell provisioners, follow upload, permissions or elevation, environment quoting, execution, and cleanup. Inspect exit-code allowlists and retry behavior for scripts that partially succeeded. Exercise the first provisioner after a reboot in a disposable guest if available. An expected disconnect can hide a failed step if the following provisioner never checks readiness.

### PK4. Verify the artifact and its first boot

Trace the builder output through individual post-processors and chained `post-processors` blocks to the manifest and the identifier consumed by deployment. Check which intermediate artifacts survive, whether parallel builds collide on a name, and whether replication, sharing, and encryption target the intended accounts. Inspect image contents for provisioning credentials, temporary keys, host-specific identity, and machine-local paths. Test the first-boot contract under the runtime identity, including service startup, generated identity where required, health, and reading packaged assets.

### PK5. Distinguish template checks from an image test

For HCL templates, `packer fmt -check .` checks formatting and `packer validate -syntax-only .` checks syntax. For a full configuration check, use the CI variable set and trusted plugins in a disposable copy. Inspect data sources and any external calls first. A cloud `packer build` provisions resources. For an audit, use existing build logs, manifests, or an authorized disposable artifact for boot checks. Record the selected builder, base identifier, consumer image identifier, and exact check. For an unbuilt candidate, specify which first-boot assertion would settle the question.

<!-- END PACKER MODULE -->
````

[Packer validation](https://developer.hashicorp.com/packer/docs/commands/validate) · [HCL formatting](https://developer.hashicorp.com/packer/docs/commands/fmt) · [Guest shell](https://developer.hashicorp.com/packer/docs/provisioners/shell) · [Host shell](https://developer.hashicorp.com/packer/docs/provisioners/shell-local) · [Post-processor sequences](https://developer.hashicorp.com/packer/docs/templates/hcl_templates/blocks/build/post-processors) · [Manifest](https://developer.hashicorp.com/packer/docs/post-processors/manifest)
