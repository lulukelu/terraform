## 0.12.19 (Unreleased)
## 0.12.18 (December 11, 2019)

NOTES:

* cli: Our darwin releases for this version and up will be signed and notarized according to Apple's requirements.

    Prior to this release, MacOS 10.15+ users attemping to run our software [reported](https://github.com/hashicorp/terraform/issues/23033) seeing the error: "'terraform' cannot be opened because the developer cannot be verified." This error affected all MacOS 10.15+ users who downloaded our software directly via web browsers, and was caused by [changes to Apple's third-party software requirements](https://developer.apple.com/news/?id=04102019a).

    [Our recommended approach to install and interact with the Terraform CLI can be found here](https://learn.hashicorp.com/terraform/getting-started/install).

    MacOS 10.15+ users should plan to upgrade to 0.12.18+.

UPGRADE NOTES:

* Inside `provisioner` blocks that have `when = destroy` set, and inside any `connection` blocks that are used by such provisioner blocks, it is now deprecated to refer to any objects other than `self`, `count`, and `each`.

    Terraform has historically allowed this but doing so tends to cause downstream problems with dependency cycles or incorrect destroy ordering because it causes the destroy phase of one resource to depend on the existing state of another. Although this is currently only a warning, we strongly suggest seeking alternative approaches for existing configurations that are hitting this warning in order to avoid the risk of later problems should you need to replace or destroy the related resources.
    
    This deprecation warning will be promoted to an error in a future release.

ENHANCEMENTS:

* provisioners: Warn about the deprecation of non-self references in destroy-time provisioners, both to allow preparation for this later becoming an error and also as an extra hint for the "Cycle" errors that commonly arise when such references are used. ([#23559](https://github.com/hashicorp/terraform/issues/23559))
* cli: The `terraform plan` and `terraform apply` commands (and some others) now accept the additional option `-compact-warnings`. If set, and if Terraform produces warnings that are not also accompanied by errors, then the warnings will be presented in the output in a compact form that includes only the summary information, thus providing a compromise to avoid warnings overwhelming the output if you are not yet ready to resolve them. ([#23632](https://github.com/hashicorp/terraform/issues/23632))

BUG FIXES:

* backend/s3: Fix for users with >1000 workspaces ([#22963](https://github.com/hashicorp/terraform/issues/22963))
* cli: Allow moving indexed resource instances to new addresses that that don't yet exist in state ([#23582](https://github.com/hashicorp/terraform/issues/23582))
* cli: Improved heuristics for log level filtering with the `TF_LOG` environment variable, although it is still not 100% reliable for levels other than `TRACE` due to limitations of Terraform's internal logging infrastructure. Because of that, levels other than `TRACE` will now cause the logs to begin with a warning about potential filtering inaccuracy. ([#23577](https://github.com/hashicorp/terraform/issues/23577))
* command/show: Fix panic on show plan ([#23581](https://github.com/hashicorp/terraform/issues/23581))
* config: Fixed referencing errors generally involving `for_each` ([#23475](https://github.com/hashicorp/terraform/issues/23475))
* provisioners: The built-in provisioners (`local-exec`, `remote-exec`, `file`, etc) will no longer fail when the `TF_CLI_ARGS` environment variable is set. ([#17400](https://github.com/hashicorp/terraform/issues/17400))

## 0.12.17 (December 02, 2019)

SECURITY NOTES:

* If you are using the Azure remote state backend and you are using a SAS Token for authentication, please refer to [the Azure remote state backend security advisory](https://github.com/hashicorp/terraform/security/advisories/GHSA-4rvg-555h-r626).

    Prior versions of the backend may have transmitted your state to the storage service using cleartext HTTP unless you specifically requested HTTPS when generating your SAS Token. This does not affect any other backends, and does not affect the Azure backend when using other authentication mechanisms.

NEW FEATURES:

*  lang/funcs: Add `trim*` functions

ENHANCEMENTS:

* cli: Terraform will now consolidate many warnings with the same summary text into fewer warning items, in order to avoid excessive amounts of warnings making it hard to read other output from Terraform commands. ([#23425](https://github.com/hashicorp/terraform/issues/23425))
* core: The upgrade logic for moving from the Terraform 0.11 to the Terraform 0.12 state snapshot format (internally, format version 3 to version 4) will now tolerate and ignore dependencies with invalid addresses, which tend to be left behind when following the `terraform 0.11checklist` directive to rename resources whose names start with digits prior to upgrading to Terraform 0.12. This should allow upgrading the state for a configuration that in the past had digit-prefixed resource names, once those names have been fixed in the configuration and state using the instructions given by `terraform 0.11checklist` in Terraform 0.11.14. ([#23443](https://github.com/hashicorp/terraform/issues/23443))

BUG FIXES:

* command/jsonplan, command/jsonstate: fix panic with null values ([#23492](https://github.com/hashicorp/terraform/issues/23492))
* backend/azure: Use HTTPS to talk to the storage API, even if using a SAS token that does not require it. ([#23496](https://github.com/hashicorp/terraform/issues/23496))

## 0.12.16 (November 18, 2019)

BUG FIXES:

* command/0.12upgrade: fix panic when int value is out of range ([#23394](https://github.com/hashicorp/terraform/issues/23394))
* core: fix cycle between dependencies with create_before_destroy ([#23399](https://github.com/hashicorp/terraform/issues/23399))
* backend/remote: default .terraformignore paths will now work on Windows ([#23311](https://github.com/hashicorp/terraform/issues/23311))

## 0.12.15 (November 14, 2019)

BUG FIXES:

* various commands: Fixed errant error "Initialization required. Please see the error message above." ([#23383](https://github.com/hashicorp/terraform/issues/23383))

    The error was produced on some of Terraform's subcommands (in particular `terraform show` and `terraform output`, but possibly others) if a warning was emitted during configuration loading and if a `backend` block was present. This issue has been present since v0.12.0 for any configuration that produces configuration-related deprecation warnings, but it became more visible in v0.12.14 due to the addition of several more situations that could produce warnings.

## 0.12.14 (November 13, 2019)

UPGRADE NOTES:

* Terraform v0.12.0 included several changes to the Terraform language involving making expressions, type constraints, keywords, and references first-class in the language syntax, removing the need for placing thee items either in quoted strings or in interpolation syntax. Terraform v0.11 required these items to be quoted because the underlying language could not represent them any other way, while Terraform v0.12 expects them to be unquoted in order to improve readability.

    We have been accepting both forms for backward-compatibility with existing configurations and examples since the inititial v0.12.0 release. Having maintained compatibility for both forms for several versions we are now beginning the deprecation cycle for the old usage by having Terraform emit deprecation warnings.
    
    Terraform will still accept the older forms in spite of these warnings, so no immediate action is required. If your modules are targeting Terraform v0.12.0 and later exclusively, you can silence the warnings by removing the quotes, as directed in the warning message. In a future major version of Terraform, some of these warnings will be elevated to be errors.
    
    The summary of the warning for these situations will be one of the following:
    
    * **Interpolation-only expressions are deprecated:** an expression like `"${foo}"` should be rewritten as just `foo`.
    * **Quoted type constraints are deprecated:** In a `variable` block, a type constraint `"map"` should be written as `map(string)`, `"list"` as `list(string)`, and `"string"` as just `string`.
    * **Quoted keywords are deprecated:** In certain contexts that expect special keywords, such as `when` in `provisioner` blocks, the keyword should be unquoted.
    * **Quoted references are deprecated:** In the `depends_on` and `ignore_changes` meta-arguments, quoted references like `"aws_instance.foo"` should be rewritten without the quotes, e.g. as `aws_instance.foo`.
    
    The above changes are made automatically by the upgrade tool for users who are [upgrading from Terraform 0.11](https://www.terraform.io/upgrade-guides/index.html). These warnings are intended to help those who are using Terraform for the first time at Terraform 0.12 but who may have found examples online that are written for older versions of Terraform, in order to guide towards the modern Terraform style.

* The `terraform output` command would formerly treat no outputs at all as an error, exiting with a non-zero status. Since it's expected for some root modules to have no outputs, the command now returns with success status zero in this situation, but still returns the error on stderr as a compromise to provide an explanation for why nothing is being shown.

ENHANCEMENTS:

* config: Redundant interpolation syntax for attribute values and legacy (0.11-style) variable type constrants will now emit deprecation warnings. ([#23348](https://github.com/hashicorp/terraform/issues/23348))
* config: Keywords and references in `depends_on`, `ignore_changes`, and in provisioner `when` and `on_failure` will now emit deprecation warnings. ([#23329](https://github.com/hashicorp/terraform/issues/23329))
* command/output: Now treats no defined outputs as a success case rather than an error case, returning exit status zero instead of non-zero. ([#23008](https://github.com/hashicorp/terraform/issues/23008)] [[#21136](https://github.com/hashicorp/terraform/issues/21136))
* backend/artifactory: Will now honor the `HTTP_PROXY` and `HTTPS_PROXY` environment variables when appropriate, to allow sending requests to the Artifactory endpoints via a proxy. ([#18629](https://github.com/hashicorp/terraform/issues/18629))

BUG FIXES:

* backend/remote: Filter environment variables when loading context for remote backend ([#23283](https://github.com/hashicorp/terraform/issues/23283))
* command/plan: Previously certain changes to lists would cause the list diff in the plan output to miss items. Now `terraform plan` will show those items as expected. ([#22695](https://github.com/hashicorp/terraform/issues/22695))
* command/show: When showing a saved plan file not in JSON mode, use the same presentation as `terraform plan` itself would've used. ([#23292](https://github.com/hashicorp/terraform/issues/23292))
* command/force-unlock: Return an explicit error when the local-filesystem lock implementation receives the wrong lock id. Previously it was possible to see either an incorrect error or no error at all in that case. ([#23336](https://github.com/hashicorp/terraform/issues/23336))
* core: Store absolute instance dependencies in state to allow for proper destroy ordering ([#23252](https://github.com/hashicorp/terraform/issues/23252))
* core: Ensure tainted status is maintained when a destroy operation fails ([#23304](https://github.com/hashicorp/terraform/issues/23304))
* config: `transpose` function will no longer panic when it should produce an empty map as its result. ([#23321](https://github.com/hashicorp/terraform/issues/23321))
* cli: When running Terraform as a sub-process of itself, we will no longer produce errant prefixes on the console output. While we don't generally recommend using Terraform recursively like this, it was behaving in this strange way due to an implementation detail of how Terraform captures "panic" crashes from the Go runtime, and that subsystem is now updated to avoid that strange behavior. ([#23281](https://github.com/hashicorp/terraform/issues/23281))
* provisioners: Sanitize output to filter invalid utf8 sequences ([#23302](https://github.com/hashicorp/terraform/issues/23302))

## 0.12.13 (October 31, 2019)

UPGRADE NOTES:

* **Remote backend local-only operations:** Previously the remote backend was not correctly handling variables marked as "HCL" in the remote workspace when running local-only operations like `terraform import`, instead interpreting them as literal strings as described in [#23228](https://github.com/hashicorp/terraform/issues/23228).

    That behavior is now corrected in this release, but in the unlikely event that an existing remote workspace contains a variable marked as "HCL" whose value is not valid HCL syntax these local-only commands will now fail with a syntax error where previously the value would not have been parsed at all and so an operation not relying on that value may have succeeded in spite of the problem. If you see an error like "Invalid expression for var.example" on local-only commands after upgrading, ensure that the remotely-stored value for the given variable uses correct HCL value syntax.
    
    This _does not_ affect true remote operations like `terraform plan` and `terraform apply`, because the processing of variables for those always happens in the remote system.

BUG FIXES:

* config: Fix regression where self wasn't properly evaluated when using for_each ([#23215](https://github.com/hashicorp/terraform/issues/23215))
* config: dotfiles are no longer excluded when copying existing modules; previously, any dotfile/dir was excluded in this copy, but this change makes the local copy behavior match go-getter behavior ([#22946](https://github.com/hashicorp/terraform/issues/22946))
* core: Ensure create_before_destroy ordering is enforced with dependencies between modules ([#22937](https://github.com/hashicorp/terraform/issues/22937))
* core: Fix some destroy-time cycles due to unnecessary edges in the graph, and remove unused resource nodes ([#22976](https://github.com/hashicorp/terraform/issues/22976))
* backend/remote: Correctly handle remotely-stored variables that are marked as "HCL" when running local-only operations like `terraform import`. Previously they would produce a type mismatch error, due to misinterpreting them as literal strings. ([#23229](https://github.com/hashicorp/terraform/issues/23229))

## 0.12.12 (October 18, 2019)

BUG FIXES:

* backend/remote: Don't do local validation of whether variables are set prior to submitting, because only the remote system knows the full set of configured stored variables and environment variables that might contribute. This avoids erroneous error messages about unset required variables for remote runs when those variables will be set by stored variables in the remote workspace. ([#23122](https://github.com/hashicorp/terraform/issues/23122))

## 0.12.11 (October 17, 2019)

ENHANCEMENTS:

* backend/s3: Support `role_arn` in AWS configuration files ([#22994](https://github.com/hashicorp/terraform/issues/22994))
* backend/remote: Remote backend will now ignore all .terraform/ (exclusive of .terraform/modules) and .git/ directories for uploads during remote plans/applies. You can exclude files from upload to TFC by adding a .terraformignore file to your configuration directory, more details at https://www.terraform.io/docs/backends/types/remote.html ([#23105](https://github.com/hashicorp/terraform/issues/23105))

BUG FIXES:

* config: Clean up orphan modules in the presence of -target ([#21313](https://github.com/hashicorp/terraform/issues/21313))
* config: Always evaluate whole resources rather than instances in expressions, so that invalid instance indexes can return a useful error rather than unknown ([#22846](https://github.com/hashicorp/terraform/issues/22846))
* command/jsonplan: fix bug with missing nested modules `planned_values` output ([#23092](https://github.com/hashicorp/terraform/issues/23092))
* command/show: Fix panic when the only resource instance is deposed ([#23027](https://github.com/hashicorp/terraform/issues/23027))
* commands: When required root module variables are not provided and interactive input is disabled (`-input=false`), produce a proper "variable not defined" error rather than falling through to an internal assertion failure. ([#23040](https://github.com/hashicorp/terraform/issues/23040))
* provisioner/puppet: fix bug when connection type was not set in config ([#23057](https://github.com/hashicorp/terraform/issues/23057))

## 0.12.10 (October 07, 2019)

ENHANCEMENTS:

* `terraform plan` and `terraform apply` will now warn when the `-target` option is used, to draw attention to the fact that the result of applying the plan is likely to be incomplete, and to remind to re-run `terraform plan` with no targets afterwards to ensure that the configuration has converged. ([#22783](https://github.com/hashicorp/terraform/issues/22783))
* config: New function `parseint` for parsing strings containing digits as integers in various bases. ([#22747](https://github.com/hashicorp/terraform/issues/22747))
* config: New function `cidrsubnets`, which is a companion to the existing function `cidrsubnet` which can allocate multiple consecutive subnet prefixes (possibly of different prefix lengths) in a single call. ([#22858](https://github.com/hashicorp/terraform/issues/22858))
* backend/google: The GCS backend now supports OAuth2 token authentication. ([#21772](https://github.com/hashicorp/terraform/issues/21772))
* provisioner/habitat: Multiple updates and fixes, see PR for details ([#22705](https://github.com/hashicorp/terraform/issues/22705))

BUG FIXES:

* backend/manta: fix panic when `insecure_skip_tls_verify` was not set ([#22918](https://github.com/hashicorp/terraform/issues/22918))

## 0.12.9 (September 17, 2019)

NOTES:
* core: `ignore_changes` is now processed (in addition to existing behaviors) before the provider plan is run. This means that users may see fewer planned changes when using `ignore_changes`, as before this change, changes to ignored attributes were still being sent to CustomizeDiff in providers (which could mean cascading changes for some resources). This should be indicative that providers are no longer getting changes that were marked as ignored, but if unexpected plans are seen while using `ignore_changes`, investigate the settings in the `ignore_changes` block to ensure the appropriate attributes are set. ([#22520](https://github.com/hashicorp/terraform/issues/22520))

ENHANCEMENTS:
* provisioners/habitat: `accept_license` argument available to automate accepting the EULA, now required by this client ([#22745](https://github.com/hashicorp/terraform/issues/22745))
* config: add source addressing to unknown value errors in `for_each` ([#22760](https://github.com/hashicorp/terraform/issues/22760))

BUG FIXES:
* command/console: support -var and -var-file flags ([#22145](https://github.com/hashicorp/terraform/issues/22145))
* command/show: Fixed bug with wrong errors being returned or swallowed. ([#22772](https://github.com/hashicorp/terraform/issues/22772))
* config: The `cidrhost`, `cidrsubnet`, and `cidrnetmask` functions now behave correctly with IPv6 prefixes that are short enough for the host portion to be greater than 64-bit or 32-bit (depending on the target architecture). ([#22505](https://github.com/hashicorp/terraform/issues/22505))
* config: Fixed bug on empty sets with `for_each` ([#22281](https://github.com/hashicorp/terraform/issues/22281))

## 0.12.8 (September 04, 2019)

NEW FEATURES:
* lang/funcs: New `fileset` function, for finding static local files that match a glob pattern. ([#22523](https://github.com/hashicorp/terraform/issues/22523))

ENHANCEMENTS:
* remote-state/pg: add option to skip schema creation ([#21607](https://github.com/hashicorp/terraform/issues/21607))

BUG FIXES:
* command/console: use user-supplied `-plugin-dir` ([#22616](https://github.com/hashicorp/terraform/issues/22616))
* config: ensure sets are appropriately known for `for_each` ([#22597](https://github.com/hashicorp/terraform/issues/22597))

## 0.12.7 (August 22, 2019)

NEW FEATURES:
* New functions `regex` and `regexall` allow applying a regular expression pattern to a string and retrieving any matching substring(s) ([#22353](https://github.com/hashicorp/terraform/issues/22353))

ENHANCEMENTS:
* lang/funcs: `lookup()` can work with maps of lists, maps and objects ([#22269](https://github.com/hashicorp/terraform/issues/22269))
* SDK: helper/acctest: Add function to return random IP address ([#22312](https://github.com/hashicorp/terraform/issues/22312))
* SDK: httpclient: Introduce composable `UserAgent(version)` ([#22272](https://github.com/hashicorp/terraform/issues/22272))
* connection/ssh: Support certificate authentication ([#22156](https://github.com/hashicorp/terraform/issues/22156))

BUG FIXES:
* config: reduce MinItems and MaxItems  validation during decoding, to allow for use of dynamic blocks ([#22530](https://github.com/hashicorp/terraform/issues/22530))
* config: don't validate MinItems and MaxItems in CoerceValue, allowing providers to set incomplete values ([#22478](https://github.com/hashicorp/terraform/issues/22478))
* config: fix panic on tuples with `for_each` ([#22279](https://github.com/hashicorp/terraform/issues/22279))
* config: fix references to `each` of `for_each` in 
s ([#22289](https://github.com/hashicorp/terraform/issues/22289))
* config: fix panic when using nested dynamic blocks ([#22314](https://github.com/hashicorp/terraform/issues/22314))
* config: ensure consistent evaluation when moving between single resources and `for_each` in addressing ([#22454](https://github.com/hashicorp/terraform/issues/22454))
* core: only start a single instance of each required provisioner ([#22553](https://github.com/hashicorp/terraform/issues/22553))
* command: fix issue where commands occasionally exited before the error message printed ([#22373](https://github.com/hashicorp/terraform/issues/22373))
* command/0.12upgrade: use user-supplied plugin-dir ([#22306](https://github.com/hashicorp/terraform/issues/22306))
* command/hook_ui: Truncate the ID considering multibyte characters ([#18823](https://github.com/hashicorp/terraform/issues/18823))
* command/fmt: Terraform fmt no longer inserts spaces after % ([#22356](https://github.com/hashicorp/terraform/issues/22356))
* command/state: Allow moving resources to modules not yet in state ([#22299](https://github.com/hashicorp/terraform/issues/22299))
* backend/google: Now using the OAuth2 token endpoint on `googleapis.com` instead of `google.com`. These endpoints are equivalent in functionality but `googleapis.com` hosts are resolvable from private Google Cloud Platform VPCs where other connectivity is restricted. ([#22451](https://github.com/hashicorp/terraform/issues/22451))

## 0.12.6 (July 31, 2019)

NOTES:

* backend/s3: After this update, the AWS Go SDK will prefer credentials found via the `AWS_PROFILE` environment variable when both the `AWS_PROFILE` environment variable and the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables are statically defined. Previously the SDK would ignore the `AWS_PROFILE` environment variable, if static environment credentials were also specified. This is listed as a bug fix in the AWS Go SDK release notes. ([#22253](https://github.com/hashicorp/terraform/issues/22253))

NEW FEATURES:
* backend/oss: added support for assume role config ([#22186](https://github.com/hashicorp/terraform/issues/22186))
* config: Resources can now use a for_each meta-argument ([#17179](https://github.com/hashicorp/terraform/issues/17179))

ENHANCEMENTS:
* backend/s3: Add support for assuming role via web identity token via the `AWS_WEB_IDENTITY_TOKEN_FILE` and `AWS_ROLE_ARN` environment variables ([#22253](https://github.com/hashicorp/terraform/issues/22253))
* backend/s3: Support automatic region validation for `me-south-1`. For AWS operations to work in the new region, the region must be explicitly enabled as outlined in the [AWS Documentation](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html#rande-manage-enable) ([#22253](https://github.com/hashicorp/terraform/issues/22253))
* connection/ssh: Improve connection debug messages ([#22097](https://github.com/hashicorp/terraform/issues/22097))

BUG FIXES:
* backend/remote: remove misleading contents from error message ([#22148](https://github.com/hashicorp/terraform/issues/22148))
* backend/s3: Load credentials via the `AWS_PROFILE` environment variable (if available) when `AWS_PROFILE` is defined along with `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` ([#22253](https://github.com/hashicorp/terraform/issues/22253))
* config: Improve conditionals to returns the correct type when dynamic values are present but unevaluated ([#22137](https://github.com/hashicorp/terraform/issues/22137))
* config: Fix panic when mistakingly using `dynamic` on an attribute ([#22169](https://github.com/hashicorp/terraform/issues/22169))
* cli: Fix crash with reset connection during init ([#22146](https://github.com/hashicorp/terraform/issues/22146))
* cli: show all deposed instances and prevent crash in `show` command ([#22149](https://github.com/hashicorp/terraform/issues/22149))
* configs/configupgrade: Fix crash with nil hilNode ([#22181](https://github.com/hashicorp/terraform/issues/22181))
* command/fmt: now formats correctly in presence of here-docs ([#21434](https://github.com/hashicorp/terraform/issues/21434))
* helper/schema: don't skip deprecation check during validation when attribute value is unknown ([#22262](https://github.com/hashicorp/terraform/issues/22262))
* plugin/sdk: allow MinItems > 1 when dynamic blocks ([#22221](https://github.com/hashicorp/terraform/issues/22221))
* plugin/sdk: fix reflect panics in helper/schema validation ([#22236](https://github.com/hashicorp/terraform/issues/22236))

## 0.12.5 (July 18, 2019)

ENHANCEMENTS:
* command/format: No longer show no-ops in `terraform show`, since nothing will change ([#21907](https://github.com/hashicorp/terraform/issues/21907))
* backend/s3: Support for assuming role using credential process from the shared AWS configuration file (support profile containing both `credential_process` and `role_arn` configurations) ([#21908](https://github.com/hashicorp/terraform/issues/21908))
* connection/ssh: Abort ssh connections when the server is no longer responding ([#22037](https://github.com/hashicorp/terraform/issues/22037))
* connection/ssh: Support ssh diffie-hellman-group-exchange-sha256 key exchange ([#22037](https://github.com/hashicorp/terraform/issues/22037))

BUG FIXES:
* backend/remote: fix conflict with normalized config dir and vcs root working directory ([#22096](https://github.com/hashicorp/terraform/issues/22096))
* backend/remote: be transparent about what filesystem prefix Terraform is uploading to the remote system, and why it's doing that ([#22121](https://github.com/hashicorp/terraform/issues/22121))
* configs: Ensure diagnostics are properly recorded from nested modules ([#22098](https://github.com/hashicorp/terraform/issues/22098))
* core: Prevent inconsistent final plan error when using dynamic in a set-type block ([#22057](https://github.com/hashicorp/terraform/issues/22057))
* lang/funcs: Allow null values in `compact` function ([#22044](https://github.com/hashicorp/terraform/issues/22044))
* lang/funcs: Pass through empty list in `chunklist` ([#22119](https://github.com/hashicorp/terraform/issues/22119))

## 0.12.4 (July 11, 2019)

NEW FEATURES: 

* lang/funcs: new `abspath` function returns the absolute path to a given file ([#21409](https://github.com/hashicorp/terraform/issues/21409))
* backend/swift: support for user configured state object names in swift containers ([#17465](https://github.com/hashicorp/terraform/issues/17465))

BUG FIXES:

* core: Prevent crash when a resource has no current valid instance ([#21979](https://github.com/hashicorp/terraform/issues/21979))
* plugin/sdk: Prevent empty strings from being replaced with default values ([#21806](https://github.com/hashicorp/terraform/issues/21806))
* plugin/sdk: Ensure resource timeouts are not lost when there is an empty plan ([#21814](https://github.com/hashicorp/terraform/issues/21814))
* plugin/sdk: Don't add null elements to diagnostic paths when validating config ([#21884](https://github.com/hashicorp/terraform/issues/21884))
* lang/funcs: Add missing map of bool support for `lookup` ([#21863](https://github.com/hashicorp/terraform/issues/21863))
* config: Fix issue with downloading BitBucket modules from deprecated V1 API by updating go-getter dependency ([#21948](https://github.com/hashicorp/terraform/issues/21948))
* config: Fix conditionals to evaluate to the correct type when using null ([#21957](https://github.com/hashicorp/terraform/issues/21957))

## 0.12.3 (June 24, 2019)

ENHANCEMENTS:

* config: add GCS source support for modules ([#21254](https://github.com/hashicorp/terraform/issues/21254))
* command/format: Reduce extra whitespaces & new lines ([#21334](https://github.com/hashicorp/terraform/issues/21334))
* backend/s3: Support for chaining assume IAM role from AWS shared configuration files ([#21815](https://github.com/hashicorp/terraform/issues/21815))

BUG FIXES:

* configs: Can now use references like `tags["foo"]` in `ignore_changes` to ignore in-place updates to specific keys in a map ([#21788](https://github.com/hashicorp/terraform/issues/21788))
* configs: Fix panic on missing value for `version` attribute in `provider` blocks. ([#21825](https://github.com/hashicorp/terraform/issues/21825))
* lang/funcs: Fix `merge` panic on null values. Now will give an error if null used ([#21695](https://github.com/hashicorp/terraform/issues/21695))
* backend/remote: Fix "Conflict" error if the first state snapshot written after a Terraform CLI upgrade has the same content as the prior state. ([#21811](https://github.com/hashicorp/terraform/issues/21811))
* backend/s3: Fix AWS shared configuration file credential source not assuming a role with environment and ECS credentials ([#21815](https://github.com/hashicorp/terraform/issues/21815))

## 0.12.2 (June 12, 2019)

NEW FEATURES:

* provisioners: new provisioner: `puppet` ([#18851](https://github.com/hashicorp/terraform/issues/18851))
* `range` function for generating a sequence of numbers as a list ([#21461](https://github.com/hashicorp/terraform/issues/21461))
* `yamldecode` and *experimental* `yamlencode` functions for working with YAML-serialized data ([#21459](https://github.com/hashicorp/terraform/issues/21459))
* `uuidv5` function for generating name-based (as opposed to pseudorandom) UUIDs ([#21244](https://github.com/hashicorp/terraform/issues/21244))
* backend/oss: Add support for Alibaba OSS remote state ([#16927](https://github.com/hashicorp/terraform/issues/16927))

ENHANCEMENTS:

* config: consider build metadata when interpreting module versions ([#21640](https://github.com/hashicorp/terraform/issues/21640))
* backend/http: implement retries for the http backend ([#19702](https://github.com/hashicorp/terraform/issues/19702))
* backend/swift: authentication mechanisms now more consistent with other OpenStack-compatible tools ([#18671](https://github.com/hashicorp/terraform/issues/18671))
* backend/swift: add application credential support ([#20914](https://github.com/hashicorp/terraform/pull/20914))

BUG FIXES:

* command/show: use the state snapshot included in the planfile when rendering a plan to json ([#21597](https://github.com/hashicorp/terraform/issues/21597))
* config: Fix issue with empty dynamic blocks failing when usign ConfigModeAttr ([#21549](https://github.com/hashicorp/terraform/issues/21549))
* core: Re-validate resource config during final plan ([#21555](https://github.com/hashicorp/terraform/issues/21555))
* core: Fix missing resource timeouts during destroy ([#21611](https://github.com/hashicorp/terraform/issues/21611))
* core: Don't panic when encountering an invalid `depends_on` ([#21590](https://github.com/hashicorp/terraform/issues/21590))
* backend: Fix panic when upgrading from a state with a hash value greater than MaxInt ([#21484](https://github.com/hashicorp/terraform/issues/21484))

## 0.12.1 (June 3, 2019)

BUG FIXES:

* core: Always try to select a workspace after initialization ([#21234](https://github.com/hashicorp/terraform/issues/21234))
* command/show: fix inconsistent json output causing a panic ([#21541](https://github.com/hashicorp/terraform/issues/21541))
* config: `distinct` function no longer panics when given an empty list ([#21538](https://github.com/hashicorp/terraform/issues/21538))
* config: Don't panic when a `version` constraint is added to a module that was previously initialized without one ([#21542](https://github.com/hashicorp/terraform/issues/21542))
* config: `matchkeys` function argument type checking will no longer fail incorrectly during validation ([#21576](https://github.com/hashicorp/terraform/issues/21576))
* backend/local: Don't panic if an instance in the state only has deposed instances, and no current instance ([#21575](https://github.com/hashicorp/terraform/issues/21575))

## 0.12.0 (May 22, 2019)

---

This is the aggregated summary of changes compared to v0.11.14. If you'd like to see the incremental changelog through each of the v0.12.0 prereleases, please refer to [the v0.12.0-rc1 changelog](https://github.com/hashicorp/terraform/blob/v0.12.0-rc1/CHANGELOG.md).

---

The focus of v0.12.0 was on improvements to the Terraform language made in response to all of the feedback and experience gathered on prior versions. We hope that these language improvements will help to make configurations for more complex situations more readable, and improve the usability of re-usable modules.

However, an overhaul of this kind inevitably means that 100% compatibility is not possible. The updated language is designed to be broadly compatible with the 0.11 language as documented, but some of the improvements required a slightly stricter parser and language model in order to resolve ambiguity or to give better feedback in error messages.

If you are upgrading to v0.12.0, we strongly recommend reading [the upgrade guide](https://www.terraform.io/upgrade-guides/0-12.html) to learn the recommended upgrade process, which includes a tool to automatically upgrade many improved language constructs and to indicate situations where human intuition is required to complete the upgrade.

### Incompatibilities and Notes

* As noted above, the language overhaul means that several aspects of the language are now parsed or evaluated more strictly than before, so configurations that employ workarounds for prior version limitations or that followed conventions other than what was shown in documentation may require some updates. For more information, please refer to [the upgrade guide](https://www.terraform.io/upgrade-guides/0-12.html).

* In order to give better feedback about mistakes, Terraform now validates that all variable names set via `-var` and `-var-file` options correspond to declared variables, generating errors or warnings if not. In situations where automation is providing a fixed set of variables to all configurations (whether they are using them or not), use [`TF_VAR_` environment variables](https://www.terraform.io/docs/commands/environment-variables.html#tf_var_name) instead, which are ignored if they do not correspond to a declared variable.

* The wire protocol for provider and provisioner plugins has changed, so plugins built against prior versions of Terraform are not compatible with Terraform v0.12. The most commonly-downloaded providers already had v0.12-compatible releases at the time of v0.12.0 release, but some other providers (particularly those distributed independently of the `terraform init` installation mechanism) will need to make new releases before they can be used with Terraform v0.12 or later.

* The index API for automatic provider installation in `terraform init` is now provided by the Terraform Registry at `registry.terraform.io`, rather than the indexes directly on `releases.hashicorp.com`. The "releases" server is still currently the distribution source for the release archives themselves at the time of writing, but that may change over time.

* The serialization formats for persisted state snapshots and saved plans have changed. Third-party tools that parse these artifacts will need to be updated to support these new serialization formats.

  For most use-cases, we recommend instead using [`terraform show -json`](https://www.terraform.io/docs/commands/show.html#json-output) to read the content of state or plan, in a form that is less likely to see significant breaking changes in future releases.

* [`terraform validate`](https://www.terraform.io/docs/commands/validate.html) now has a slightly smaller scope than before, focusing only on configuration syntax and type/value checking. This makes it safe to run in unattended scenarios, such as on save in a text editor.

### New Features

The full set of language improvements is too large to list them all out exhaustively, so the list below covers some highlights:

* **First-class expressions:** Prior to v0.12, expressions could be used only via string interpolation, like `"${var.foo}"`. Expressions are now fully integrated into the language, allowing them to be used directly as argument values, like `ami = var.ami`.

* **`for` expressions:** This new expression construct allows the construction of a list or map by transforming and filtering elements from another list or map. For more information, refer to [the _`for` expressions_ documentation](https://www.terraform.io/docs/configuration/expressions.html#for-expressions).

* **Dynamic configuration blocks:** For nested configuration blocks accepted as part of a resource configuration, it is now possible to dynamically generate zero or more blocks corresponding to items in a list or map using the special new `dynamic` block construct. This is the official replacement for the common (but buggy) unofficial workaround of treating a block type name as if it were an attribute expecting a list of maps value, which worked sometimes before as a result of some unintended coincidences in the implementation.

* **Generalised "splat" operator:** The `aws_instance.foo.*.id` syntax was previously a special case only for resources with `count` set. It is now an operator within the expression language that can be applied to any list value. There is also an optional new splat variant that allows both index and attribute access operations on each item in the list. For more information, refer to [the _Splat Expressions_ documentation](https://www.terraform.io/docs/configuration/expressions.html#splat-expressions).

* **Nullable argument values:** It is now possible to use a conditional expression like `var.foo != "" ? var.foo : null` to conditionally leave an argument value unset, whereas before Terraform required the configuration author to provide a specific default value in this case. Assigning `null` to an argument is equivalent to omitting that argument entirely.

* **Rich types in module inputs variables and output values:** Terraform v0.7 added support for returning flat lists and maps of strings, but this is now generalized to allow returning arbitrary nested data structures with mixed types. Module authors can specify an expected [type constraint](https://www.terraform.io/docs/configuration/types.html) for each input variable to allow early type checking of arguments.

* **Resource and module object values:** An entire resource or module can now be treated as an object value within expressions, including passing them through input variables and output values to other modules, using an attribute-less reference syntax, like `aws_instance.foo`.

* **Extended template syntax:** The simple interpolation syntax from prior versions is extended to become a simple template language, with support for conditional interpolations and repeated interpolations through iteration. For more information, see [the _String Templates_ documentation](https://www.terraform.io/docs/configuration/expressions.html#string-templates).

* **`jsondecode` and `csvdecode` interpolation functions:** Due to the richer type system in the new configuration language implementation, we can now offer functions for decoding serialization formats. [`jsondecode`](https://www.terraform.io/docs/configuration/functions/jsondecode.html) is the opposite of [`jsonencode`](https://www.terraform.io/docs/configuration/functions/jsonencode.html), while [`csvdecode`](https://www.terraform.io/docs/configuration/functions/csvdecode.html) provides a way to load in lists of maps from a compact tabular representation.

* **Revamped error messages:** Error messages relating to configuration now always include information about where in the configuration the problem was found, along with other contextual information. We have also revisited many of the most common error messages to reword them for clarity, consistency, and actionability.

* **Structual plan output:** When Terraform renders the set of changes it plans to make, it will now use formatting designed to be similar to the input configuration language, including nested rendering of individual changes within multi-line strings, JSON strings, and nested collections.

### Other Improvements

* `terraform validate` now accepts an argument `-json` which produces machine-readable output. Please refer to the documentation for this command for details on the format and some caveats that consumers must consider when using this interface. ([#17539](https://github.com/hashicorp/terraform/issues/17539))

* The JSON-based variant of the Terraform language now has a more tightly-specified and reliable mapping to the native syntax variant. In prior versions, certain Terraform configuration features did not function as expected or were not usable via the JSON-based forms. For more information, see [the _JSON Configuration Syntax_ documentation](https://www.terraform.io/docs/configuration/syntax-json.html).

* The new built-in function [`templatefile`](https://www.terraform.io/docs/configuration/functions/templatefile.html) allows rendering a template from a file directly in the language, without installing the separate Template provider and using the `template_file` data source.

* The new built-in function [`formatdate`](https://www.terraform.io/docs/configuration/functions/formatdate.html), which is a specialized string formatting function for creating machine-oriented timestamp strings in various formats.

* The new built-in functions [`reverse`](https://www.terraform.io/docs/configuration/functions/reverse.html), which reverses the order of items in a list, and [`strrev`](https://www.terraform.io/docs/configuration/functions/strrev.html), which reverses the order of Unicode characters in a string.

* A new `pg` state storage backend allows storing state in a PostgreSQL database.

* The `azurerm` state storage backend supports new authentication mechanisms, custom resource manager endpoints, and HTTP proxies.

* The `s3` state storage backend now supports `credential_source` in AWS configuration files, support for the new AWS regions `eu-north-1` and `ap-east-1`, and several other improvements previously made in the `aws` provider.

* The `swift` state storage backend now supports locking and workspaces.

### Bug Fixes

Quite a few bugs were fixed indirectly as a result of improvements to the underlying language engine, so a fully-comprehensive list of fixed bugs is not possible, but some of the more commonly-encountered bugs that are fixed in this release include:

* config: The conditional operator `... ? ... : ...` now works with result values of any type and only returns evaluation errors for the chosen result expression, as those familiar with this operator in other languages might expect.

* config: Accept and ignore UTF-8 byte-order mark for configuration files ([#19715](https://github.com/hashicorp/terraform/issues/19715))

* config: When using a splat expression like `aws_instance.foo.*.id`, the addition of a new instance to the set (whose `id` is therefore not known until after apply) will no longer cause all of the other ids in the resulting list to appear unknown.

* config: The `jsonencode` function now preserves the types of values passed to it, even inside nested structures, whereas before it had a tendency to convert primitive-typed values to string representations.

* config: The `format` and `formatlist` functions now attempt automatic type conversions when the given values do not match the "verbs" in the format string, rather than producing a result with error placeholders in it.

* config: Assigning a list containing one or more unknown values to an argument expecting a list no longer produces the incorrect error message "should be a list", because Terraform is now able to track the individual elements as being unknown rather than the list as a whole, and to track the type of each unknown value. (This also avoids any need to place seemingly-redundant list brackets around values that are already lists, which would now be interpreted as a list of lists.)

* cli: When `create_before_destroy` is enabled for a resource, replacement actions are reflected correctly in rendered plans as `+/-` rather than `-/+`, and described as such in the UI messages.

* core: Various root causes of the "diffs didn't match during apply" class of error are now checked at their source, allowing Terraform to either avoid the problem occurring altogether (ideally) or to provide a more actionable error message to help with reporting, finding, and fixing the bug.

---

For information on v0.11 and prior releases, please see [the v0.11 branch changelog](https://github.com/hashicorp/terraform/blob/v0.11/CHANGELOG.md).
