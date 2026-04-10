# Security Review Guidelines

Lessons from upstream review (Kicksecure/systemcheck#3).

## Use existing tools

- **`sanitize-string`** (`/usr/bin/sanitize-string` from helper-scripts) already handles
  string sanitization (length limiting, special character handling). Use it instead of
  writing custom `html_escape` or similar functions.

## canary-download runs confined

- The `canary-download` Python script runs as user **`canary`** and is **AppArmor-confined**.
- Symlink protections (`O_NOFOLLOW`) and restrictive file permissions are unnecessary here
  because AppArmor already restricts which paths the process can write to.

## Don't suppress useful debugging info

- Do not sanitize or suppress error messages that contain paths, network details, or
  tracebacks. If the system is compromised, malware already has access to this info.
  If not compromised, the detail is needed for debugging.
- Paths embedded in the source code itself are public information and safe to log.
- Prefer `traceback` module for Python exception logging to get full context.

## Local variables don't need origin validation

- If a variable is set locally within a function (e.g., `tor_consensus_status` in
  `check_anondate_do`), don't add validation guards against unexpected values. The
  variable can't be inherited externally.

## Config file permission checks: error out, don't skip

- When detecting world-writable config files, **error out entirely** (`return 1`)
  rather than silently skipping the file with `continue`.
- Do not use `|| continue` on `stat` calls in this context; if `stat` fails, the
  script should error out so the problem is visible.

## Don't add timeouts to grep on local temp files

- Adding `timeout` to `grep` on files the script itself just wrote is not useful.
  The file size is bounded by the journal output, and a timeout could cause problems
  if logs are legitimately large.

## mktemp permissions

- `mktemp --directory` creates directories with mode 0700 by default, but this is
  due to the default umask, not an inherent guarantee from mktemp. Don't remove
  existing `chmod 700` calls without verifying that the umask is explicitly set in
  the execution path.

## Input validation that IS useful

- Validating date strings match `YYYY-MM-DD` before passing to `date --date=` (even
  if the source file is trusted).
- Validating function names with `declare -F` before executing via `$FUNCTION`.
- Validating qubes-db output is a valid IPv4 address before embedding in messages.
