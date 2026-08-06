<!-- GSD:project-start source:PROJECT.md -->
## Project

**reconFTW**

reconFTW is a comprehensive bash-based reconnaissance automation framework used by bug bounty hunters, penetration testers, and security researchers. It orchestrates 70+ external security tools (Go, Python, Rust) across subdomain enumeration, web probing, OSINT, and vulnerability scanning, producing structured per-target output trees with optional Axiom distributed execution, AI reporting, monitor/incremental mode, and Slack/Telegram/Discord notifications.

**Core Value:** Run one command, get a complete recon picture of a target — passive, active, and vulnerability layers — with resumable checkpoints, structured output, and zero-touch tool orchestration.

### Constraints

- **Tech stack**: Bash 4.3+ — Required for `wait -n`, `mapfile`, associative arrays. macOS users must have Homebrew bash; auto re-exec is best-effort.
- **External tools**: 70+ runtime dependencies — Most install via `go install @latest` (no version pinning), which is convenient but a known supply-chain risk.
- **Single process**: All modules sourced into one shell — No subshell isolation between modules; all state shared via globals. Workflow functions must save/restore globals they override (see `passive()` pattern).
- **Resume semantics**: Checkpoint files are touch-once at `end_func` — Interrupted functions re-run from scratch on next invocation; partial outputs are not detected.
- **Single-operator**: Designed for one user per target run — No locking, no multi-user state, no concurrent runs against the same target dir.
- **Output stability**: `Recon/<domain>/` tree is a public contract — Subdirectory names and filenames are consumed by downstream pipelines, scripts, and parsers; renames are breaking changes.
- **macOS compatibility**: GNU coreutils + GNU sed + GNU getopt required — System BSD versions are not supported.
- **CI budget**: Integration-full is weekly cron — Unit + smoke are per-push; adding heavy integration tests must respect this split.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Bash 4+ - All core framework logic (`reconftw.sh`, `modules/*.sh`, `lib/*.sh`, `install.sh`)
- Python 3.7+ - Python-backed tools (each runs in isolated `uv venv`): `dorks_hunter`, `CMSeeK`, `EmailHarvester`, `Spoofy`, `SSTImap`, `gato`, `regulator`, `reconftw_ai`, `getjswords.py`
- Go (latest, min ~1.21) - Primary binary language for ~55 security tools installed via `go install @latest`
## Runtime
- Linux (Debian/Ubuntu/RHEL/Arch) or macOS (Apple Silicon / Intel)
- Docker: Ubuntu 24.04 base image (`Docker/Dockerfile`); no automated CI push — build manually with `docker build -f Docker/Dockerfile .`
- ARM64/aarch64, ARMv6l/v7l, and x86_64 all supported for Go binary installs
- `getopt` (GNU getopt required on macOS via `brew install gnu-getopt`)
- `nproc` / `sysctl -n hw.ncpu` for CPU core auto-detection
- `timeout` / `gtimeout` (macOS Homebrew `coreutils`)
- `gnu-sed` (macOS requires `brew install gnu-sed`)
- `gnu-coreutils` (macOS requires `brew install coreutils`)
- Go tools: `go install` (`GOPATH=$HOME/go`, `GOROOT=/usr/local/go`)
- Python tools: `uv tool install` from GitHub (most) or PyPI (`fray`)
- Python repo venvs: `uv venv venv && uv pip install -r requirements.txt`
- Lockfile: None (always installs `@latest`)
## Frameworks
- No framework — pure Bash with sourced module files
- Module loading order: `lib/validation.sh` → `lib/common.sh` → `lib/ui.sh` → `lib/parallel.sh` → `modules/utils.sh` → `modules/core.sh` → `modules/osint.sh` → `modules/subdomains.sh` → `modules/web.sh` → `modules/vulns.sh` → `modules/axiom.sh` → `modules/modes.sh`
- bats-core (Bash Automated Testing System)
- GNU make (`Makefile`) — test, lint, format targets
- shellcheck (error-level) — `make lint`, pre-commit hook
- shfmt (4-space indent, `-bn`, `-ci`) — `make fmt`, pre-commit hook
- pre-commit hooks defined in `.pre-commit-config.yaml`
- semgrep: run locally with `semgrep --config=auto` (no CI workflow configured)
## Key Dependencies
### Go Tools (via `go install @latest`)
`subfinder`, `github-subdomains`, `gitlab-subdomains`, `dnstake`, `puredns`, `dnsx`, `massdns` (repo clone+build), `dsieve`, `enumerepo`, `gotator`, `analyticsrelationships`, `roboxtractor`, `crt`, `asnmap`, `mapcidr`, `smap`, `tlsx`, `hakip2host`, `cdncheck`, `hakoriginfinder`, `inscope`, `csprecon`, `favirecon`, `httpx`, `katana`, `ffuf`, `subjs`, `Gxss`, `jsluice`, `sourcemapper`, `mantra`, `urlfinder`, `xnLinkFinder`, `nmapurls`, `naabu`, `VhostFinder`, `shortscan`, `nuclei`, `dalfox`, `crlfuzz`, `Web-Cache-Vulnerability-Scanner`, `TInjA`, `toxicache`, `second-order`, `s3scanner`, `misconfig-mapper`, `sj`, `grpcurl`, `nerva`, `brutus`, `julius`, `titus`, `notify`, `interactsh-client`, `gf`, `anew`, `unfurl`, `qsreplace`, `gitdorks_go`, `github-endpoints`, `cent`, `brutespray`
- `trufflehog` — `go build` from cloned repo; `go install` blocked by replace directives in go.mod
### Python Tools (via `uv tool install`)
`dnsvalidator`, `interlace`, `wafw00f`, `commix`, `waymore`, `urless`, `ghauri`, `xnLinkFinder`, `xnldorker`, `porch-pirate`, `p1radup`, `subwiz`, `arjun`, `gqlspection`, `postleaksNg`, `cewler`, `fray`
### Repo-Clone Tools (Python venvs + `$GOPATH/bin/` wrapper)
`corsy` (~/Tools/corsy), `jwt_tool` — not proper Python packages; cloned + venv + wrapper binary
### Repo-Clone Tools (Python venvs, run via `venv/bin/python3`)
`dorks_hunter`, `CMSeeK`, `EmailHarvester`, `SwaggerSpy`, `LeakSearch`, `Spoofy`, `msftrecon`, `Scopify`, `regulator`, `SSTImap`, `gato`
- `cloud_enum` — uses `pyproject.toml` (not `requirements.txt`); install via `uv pip install .`
### Repo-Clone Tools (Go build)
`ghleaks`, `nomore403`, `ffufPostprocessing`, `JSA`, `ultimate-nmap-parser`
### System-Level (apt/brew/yum)
`nmap`, `massdns`, `jq`, `exiftool`, `whois`, `sqlmap`, `testssl.sh`, `medusa`
- `shodan` CLI — `pip install shodan --break-system-packages` (uv broken by missing `pkg_resources`)
### Prebuilt Binary (GitHub Releases)
`noseyparker` (praetorian-inc/noseyparker) — x86_64 + arm64; no Go module or PyPI package
### Rust (Cargo)
`smugglex`; Rustup from `https://sh.rustup.rs`
## Configuration
- `reconftw.cfg` — sourced after CLI parsing; all feature flags, rate limits, timeouts, wordlist paths, API keys, thread counts
- `secrets.cfg` (gitignored, auto-sourced) — API keys and tokens separated from main config
- `secrets.cfg.example` — template showing all supported secret vars
- Feature flags: `OSINT=true`, `SUBDOMAINS_GENERAL=true`, `VULNS_GENERAL=false`, etc.; `SUBLOCALDB=true` / `LOCAL_DOMAIN_DB` control the local FQDN database lookup (`sub_localdb`)
- Rate limits: `HTTPX_RATELIMIT=150`, `NUCLEI_RATELIMIT=150`, `FFUF_RATELIMIT=0`
- Thread counts: auto-scaled via `AVAILABLE_CORES=$(nproc)` with multipliers per tool
- Timeouts: per-tool in seconds or minutes (`CMSSCAN_TIMEOUT=3600`, `SUBFINDER_ENUM_TIMEOUT=180`)
- Wordlist paths: `fuzz_wordlist`, `lfi_wordlist`, `subs_wordlist` etc. under `${WORDLISTS_DIR}`
- Output: `EXPORT_FORMAT`, `AI_REPORT_TYPE`, `ASSET_STORE`
- GNU `getopt` long options, parsed in `reconftw.sh` while/case loop
- All CLI overrides use `CLI_*` pattern and are re-applied after `reconftw.cfg` is sourced
- Full list from `getopt` call: `domain`, `list`, `recon`, `subdomains`, `passive`, `all`, `web`, `osint`, `zen`, `deep`, `help`, `vps`, `vps-count`, `ai`, `check-tools`, `health-check`, `quick-rescan`, `incremental`, `adaptive-rate`, `dry-run`, `parallel`, `no-parallel`, `monitor`, `monitor-interval`, `monitor-cycles`, `refresh-cache`, `gen-resolvers`, `force`, `export`, `report-only`, `no-report`, `parallel-log`, `quiet`, `verbose`, `no-color`, `log-format`, `show-cache`, `banner`, `no-banner`, `legal`
- `SHODAN_API_KEY`, `WHOISXML_API`, `PDCP_API_KEY`, `XSS_SERVER`, `COLLAB_SERVER` — preferred over config file
- `GOROOT`, `GOPATH`, `PATH` — extended by `reconftw.cfg` for Go and Rust binaries
- `LOGFILE` — per-target log path
- `config/reconftw_full.cfg` — full-scan preset
- `config/reconftw_quick.cfg` — quick-scan preset
- `config/reconftw_stealth.cfg` — low-noise preset
## Build
- Default: `go1.23.6` (fetches latest from `https://go.dev/VERSION?m=text`)
- Installed to `/usr/local/go`; set `install_golang=false` in config to skip
- Minimum: Python 3.7 (enforced in `install_yum()`)
- Virtual environments per tool via `uv venv`
- Root venv at `.venv/` for `getjswords.py` and similar helpers
## Platform Requirements
- Bash ≥ 4.3 (for `wait -n` used in `lib/parallel.sh`)
- Go ≥ 1.21 (tools use SIV module paths like `/v2`, `/v3`)
- Python ≥ 3.7
- `uv` package manager
- Rust / Cargo (for `smugglex`)
- GNU coreutils, getopt, sed (macOS only via Homebrew)
- ~5GB free disk space for Go cache, tools, and repos
- ~1GB RAM minimum (Go compilation)
- Base image: `ubuntu:24.04`
- Build arg `INSTALL_AXIOM=true` (default) installs axiom fleet tooling
- Ports 85-90 exposed (for headless browser tooling)
- Runs as root (required for raw socket operations by some tools)
- Health check: `./reconftw.sh --health-check`
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Shell Settings
- `reconftw.sh`: `set -o pipefail; set -E; set +e` — pipefail catches pipe failures; `-E` propagates ERR trap through functions; `+e` keeps the run alive so a single tool failure doesn't abort everything
- `lib/validation.sh`: `set -o pipefail` only — stricter because it's a pure-library file with no tool-execution side effects
## Source Guard Pattern
- `lib/common.sh`: `[[ -n "$_COMMON_SH_LOADED" ]] && return 0`
- `lib/parallel.sh`: `[[ -n "$_PARALLEL_SH_LOADED" ]] && return 0`
- `lib/ui.sh`: `[[ -n "${_UI_SH_LOADED:-}" ]] && return 0`
## Function Naming
- Public module functions: `snake_case` prefixed with module context (`sub_passive`, `sub_crt`, `geo_info`)
- Private helpers: `_snake_case` prefix (`_print_status`, `_print_error`, `_print_module_start`, `_parallel_emit_job_output`)
- UI layer: `ui_` prefix (`ui_init`, `ui_header`, `ui_summary`, `ui_batch_end`)
- Lifecycle wrappers: `start_func` / `end_func` (call these at top/bottom of every recon function)
- Validation functions: `validate_*` / `sanitize_*` (defined in `lib/validation.sh` and `modules/utils.sh`)
## Variable Naming
- Config flags: `SUBPASSIVE`, `SUBCRT`, `PARALLEL_MODE`, `OUTPUT_VERBOSITY`
- Runtime state: `LOGFILE`, `SCRIPTPATH`, `DIFF`, `DRY_RUN`, `AXIOM`
- Error codes: `E_SUCCESS=0`, `E_INVALID_DOMAIN=20`, `E_INVALID_IP=21` (readonly, defined in `lib/validation.sh`)
## CLI Flag Pattern
- CLI flags set `CLI_*` variables **before** sourcing `reconftw.cfg` (e.g. `CLI_DOMAIN`, `CLI_SUBDOMAINS`)
- After sourcing, `CLI_*` are re-applied in explicit `if` blocks so they always win over config values
- Pattern: `if [[ -n "${CLI_DOMAIN:-}" ]]; then domain="$CLI_DOMAIN"; fi`
- This two-pass approach means users can override any config default from the command line without editing files
## Output / UI Conventions
- `OUTPUT_VERBOSITY=0` (quiet): only errors/FAIL printed
- `OUTPUT_VERBOSITY=1` (normal, default): OK/WARN/FAIL/SKIP status lines
- `OUTPUT_VERBOSITY=2` (verbose): all of the above + INFO messages + start_func messages
## Live-Progress Cursor State Machine (`lib/ui.sh`)
- `ui_live_progress_begin` — prints `\n` to create a blank status slot, sets `_UI_LIVE_NEEDS_UP=true`
- `ui_live_progress_update` — when `_UI_LIVE_NEEDS_UP=true` uses `\033[1A` (cursor up) to return to the slot before overwriting; every frame ends with `\n` so the scrollback buffer records clean lines instead of raw escape codes; sets `_UI_LIVE_NEEDS_UP=true` on exit
- `ui_live_progress_break` — erases the slot (going up if `_UI_LIVE_NEEDS_UP=true`), sets `_UI_LIVE_NEEDS_UP=false`; static output then writes on the cleared line and the next update treats the resulting `\n`-fresh line as the new slot without going up
- `ui_live_progress_end` — same erase as break, then sets `_UI_LIVE_ACTIVE=false`
- `_UI_CACHED_WIDTH` is invalidated on `SIGWINCH` so a terminal resize never produces wrapped lines that corrupt the cursor offset
## Function Lifecycle (start_func / end_func)
- `start_func name desc` — logs to LOGFILE, sets per-function start timestamp, emits INFO at verbosity >= 2
- `end_func message name [status]` — touches checkpoint file, calculates elapsed time, calls `_print_status`
- `skip_notification reason` — emits SKIP/CACHE badge; reasons: `"disabled"`, `"mode"`, `"processed"`, `"processed-visible"`, `"noinput"`
## File Checkpointing (Resumability)
- `end_func` creates the checkpoint: `touch "$called_fn_dir/.${fn}"`
- DIFF mode (`DIFF=true`) bypasses checkpoint — forces re-execution
- Helper `should_run()` in `lib/common.sh` provides a cleaner gate: `if should_run "FLAG_VAR"; then`
## Error Handling
- `E_SUCCESS=0`, `E_GENERAL=1`, `E_MISSING_DEP=2`, `E_INVALID_INPUT=3`
## Validation Functions
| Function | Purpose |
|----------|---------|
| `validate_domain()` | RFC domain check + injection character rejection |
| `validate_ipv4()` | Octet range validation |
| `validate_integer()` | Numeric range check |
| `validate_boolean()` | Accepts `true`/`false` only (not `1`/`0`/`yes`/`no`) |
| `validate_file_readable()` | Exists + readable + is-a-file |
| `sanitize_interlace_input()` | Removes shell metacharacters from input files (canonical in `lib/validation.sh`) |
| `sanitize_domain()` | Strips URL components, lowercases, rejects injection (in `modules/utils.sh`) |
| `is_in_scope_host()` | Anchored hostname scope check (prevents substring false positives) |
| `filter_in_scope_urls()` | Python3-based URL scope check (scheme, userinfo, host) |
## Path and CWD Conventions
- `start()` calls `cd "$dir"` so all relative paths inside module functions resolve to `Recon/<domain>/`
- `reconftw.sh` captures `startdir=${PWD}` before any `cd` so relative input paths (e.g. `-l domains.txt`) resolve against the original working directory
- Module functions must never `cd` without restoring CWD — use absolute paths or subshells for one-off directory changes
## Parallel Execution
- `parallel_funcs N fn_a fn_b fn_c` — spawns each function as a background subshell
- `_throttle_jobs` uses `wait -n` (bash 4.3+) to keep at most N jobs running at once
- Each job's stdout is captured to a per-job temp file; output is replayed on completion per `PARALLEL_LOG_MODE` (summary / tail / full)
- A heartbeat loop emits live progress to the terminal at verbosity ≥ 1
- Sequential long-running tools use `run_with_heartbeat "label" [--total-queries N] [--rate-per-sec R] [interval_s] cmd args`; passing `--total-queries` (wordlist line count) and `--rate-per-sec` (tool QPS limit) enables a rate-based ETA countdown — `_bruteforce_domains` and `_resolve_domains` set these automatically from `PUREDNS_PUBLIC_LIMIT` / `DNSX_RATE_LIMIT`
## Import / Sourcing Order
- Covered by the module loading order in the Frameworks section above; no circular imports by design
## Logging
- All tool output redirected to `$LOGFILE`: `command ... 2>>"$LOGFILE" >/dev/null`
- Structured JSON logging via `log_json level func message [key=val]` (optional, `STRUCTURED_LOGGING=true`)
- `redact_secrets()` scrubs `REDACT_VARS` and `REGISTERED_SECRETS` from log lines
- `register_secret "$value"` must be called before logging any secret value
## Comments
- Only when the WHY is non-obvious: a hidden constraint, a subtle invariant, a workaround for a specific bug
- No what-it-does comments; no task/fix/caller references ("added for X", "used by Y")
- One short line max — no multi-line comment blocks
## Testing Conventions
- **Mandatory test setup boilerplate** after every `source reconftw.sh --source-only`:
  ```bash
  source "$project_root/reconftw.sh" --source-only
  set -e               # restore errexit — reconftw.sh does global set +e
  export MIN_DISK_SPACE_GB=0  # disable disk-space guard in tests
  ```
  Without `set -e`, assertion failures inside test bodies are silently swallowed.
- **Re-export `CACHE_DIR` after sourcing** — `modules/utils.sh` unconditionally sets `CACHE_DIR="${SCRIPTPATH}/.cache"` at module load time, overwriting any value set before sourcing. Tests that need a custom cache directory must re-export it after the `source` call.
- **`declare -gA` in test bodies for associative arrays** — `declare -A` inside a bats `setup()` function creates a *local* array that vanishes when setup returns. Use `declare -gA VAR=()` inside the test body itself (or at module scope in production code) to ensure the array survives into the test.
- **`$BASHPID` not `$$` for unique per-process IDs** — `$$` is the same in all subshells (it's the top-level shell PID). `$BASHPID` is unique per process and must be used when generating directory names or tokens that must not collide across concurrent subshells.
- **`anew` mock pattern** — tests that exercise functions which pipe output through `anew` must mock it. Standard mock:
  ```bash
  cat > "$MOCK_BIN/anew" <<'SH'
  #!/usr/bin/env bash
  quiet=false
  if [[ "${1:-}" == "-q" ]]; then quiet=true; shift; fi
  outfile="$1"; mkdir -p "$(dirname "$outfile")"; touch "$outfile"
  while IFS= read -r line; do
    [[ -z "$line" ]] && continue
    if ! grep -Fxq -- "$line" "$outfile"; then
      printf '%s\n' "$line" >> "$outfile"
      [[ "$quiet" != true ]] && printf '%s\n' "$line"
    fi
  done
  SH
  chmod +x "$MOCK_BIN/anew"
  ```
- **`run_with_heartbeat_shell` resets PATH in tests** — `lib/common.sh:run_with_heartbeat_shell` calls `/bin/bash -lc`, which sources login profiles and resets `PATH`, losing any `MOCK_BIN` prepended by the test. Override it inside the test body: `run_with_heartbeat_shell() { bash -c "$2"; }`.
- **Don't pre-seed files that functions wipe on entry** — some functions (e.g. `swagger_check`) start with `: >.tmp/somefile` to clear state. Pre-populating that file in the test is ineffective. Instead supply input through whichever source the function reads in its Phase 2 (e.g. `nuclei_output/info_json.txt`).
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## System Overview
```text
CLI (reconftw.sh)
  └─ loads libs: validation → common → ui → parallel
  └─ loads modules: utils → core → osint → subdomains → web → vulns → axiom → modes
  └─ parses getopt → sources reconftw.cfg → re-applies CLI_* overrides
  └─ dispatches to modes.sh workflow function
       └─ start() creates Recon/<domain>/ tree, sets $dir / $called_fn_dir
       └─ module functions called (via run_command / parallel_funcs)
       └─ end() triggers AI report, export, notifications, summary
```
## Component Responsibilities
| Component | Responsibility | File |
|-----------|----------------|------|
| Entry point & CLI parser | getopt argument parsing, config sourcing, mode dispatch | `reconftw.sh` |
| Mode orchestration | `start`/`end`, workflow functions (`recon`, `passive`, `all`, `vulns`, `osint`, `subs_menu`, `webs_menu`, `zen_menu`, `monitor_mode`, `attack_map`); `generate_attack_map()` produces `attack-map.html` + `attack-map.md` | `modules/modes.sh` |
| Function lifecycle | `start_func`/`end_func`, checkpointing, logging, notifications, reporting, plugins, health check | `modules/core.sh` |
| Subdomain enumeration | All `sub_*` functions, `subtakeover`, `zonetransfer`, `s3buckets`, `geo_info` | `modules/subdomains.sh` |
| Web analysis | `webprobe_full`, `screenshot`, `nuclei_check`, `fuzz`, `jschecks`, `urlchecks`, `waf_checks`, and 20+ others | `modules/web.sh` |
| Vulnerability scanning | `xss`, `ssrf_checks`, `sqli`, `crlf_checks`, `lfi`, `ssti`, `smuggling`, `fuzzparams`, `nuclei_dast`, `cors_checks`, `jwt_checks`, `open_redirect`, `test_ssl`, `4xxbypass`, `webcache`, `spraying`, `command_injection`, `fray_checks`; passive DAST: `dast_passive` (security headers / cookie flags / info disclosure / MIME mismatch), `exposed_files` (backup & sensitive file detection), `http_methods` (TRACE/PUT/DELETE) | `modules/vulns.sh` |
| OSINT collection | `domain_info`, `ip_info`, `emails`, `google_dorks`, `github_leaks`, `github_actions_audit`, `cloud_enum_scan`, etc. | `modules/osint.sh` |
| Shared utilities | `run_command`, `sed_i`, `deleteOutScoped`, `validate_config`, `cache_*`, `checkpoint_*`, `circuit_breaker_*`, rate-limit adaption | `modules/utils.sh` |
| Axiom/distributed mode | `axiom_launch`, `axiom_shutdown`, `axiom_selected`, `resolvers_update`, `ipcidr_target` | `modules/axiom.sh` |
| Parallel execution | `parallel_funcs`, `_throttle_jobs`, job heartbeat, progress live display, log mode output | `lib/parallel.sh` |
| Input validation/sanitization | `sanitize_domain`, `sanitize_ip`, `validate_domain`, `validate_integer`, `_sanitize_list_entry` | `lib/validation.sh` |
| Shared file/counter utilities | `ensure_dirs`, `ensure_webs_all`, `safe_backup`, `count_lines`, incident tracking | `lib/common.sh` |
| UI presentation layer | `_print_status`, `_print_msg`, `_print_section`, `_print_rule`, `ui_header`, `ui_summary`, TTY detection, color management, JSONL output | `lib/ui.sh` |
| Configuration | All runtime settings (~350 variables); sourced after CLI parse | `reconftw.cfg` |
## Pattern Overview
- Single process: `reconftw.sh` sources all libraries and modules at startup; every function lives in the same shell environment
- No subshell isolation between modules — all state is shared via global variables
- File-based checkpointing: `called_fn_dir/.funcname` sentinel files prevent re-running completed functions across invocations
- CLI-over-config: `reconftw.cfg` provides defaults; CLI flags set `CLI_*` variables that are re-applied after config sourcing to guarantee they cannot be overwritten
- All external tool invocations go through `run_command()` which handles dry-run mode, adaptive rate limiting, axiom dispatch, and debug logging
## Layers
- Purpose: Bootstrap, macOS re-exec, module loading, getopt CLI parsing, config sourcing, CLI override re-application, mode dispatch
- Location: `reconftw.sh`
- Contains: `normalize_vps_count_args()`, the main `while/case` getopt loop, the config `source` sequence, CLI override if-blocks, the final `case $opt_mode` dispatch
- Depends on: All libraries (sourced first), all modules (sourced second)
- Used by: End user / CI
- Purpose: Reusable utilities with no side effects; loadable independently for tests
- Location: `lib/validation.sh`, `lib/common.sh`, `lib/ui.sh`, `lib/parallel.sh`
- Contains: Input sanitization, file helpers, UI/color/progress, parallel job management
- Depends on: Nothing (source-guarded with `_*_LOADED` pattern)
- Used by: All modules and reconftw.sh
- Purpose: Implement all scanning, analysis, and orchestration functions
- Location: `modules/`
- Contains: All recon, vuln, OSINT, web, subdomain functions
- Depends on: Libraries (always loaded first), `reconftw.cfg` variables, external tools on PATH
- Used by: modes.sh orchestrates all others; reconftw.sh dispatches to modes.sh
- Purpose: Default runtime values for ~350 flags/paths/limits; can be overridden by `secrets.cfg` and custom config
- Location: `reconftw.cfg`, optionally `secrets.cfg`, optionally `$CUSTOM_CONFIG`
- Contains: Module enable/disable flags, tool flags, API key env-var references, paths, parallelism settings, verbosity, Axiom settings
- Depends on: Nothing
- Used by: Sourced by `reconftw.sh` between CLI parse and CLI override re-application
- Purpose: Store per-target findings in a stable directory hierarchy
- Location: `Recon/<domain>/` (created at `start()` time by `modules/modes.sh`)
- Contains: Standard subdirectories listed below
- Depends on: `start()` in modes.sh creates the directory tree
## Data Flow
### Primary Recon Request Path (`-r` / `--recon`)
`reconftw.sh` → parse getopt → source `reconftw.cfg` → re-apply `CLI_*` → dispatch to `recon()` in `modes.sh` → `start()` (creates output tree) → `parallel_funcs` over [osint, subs_menu, webs_menu, vulns groups] → `end()` (AI report, export, summary)

### Function Execution Path (every leaf module function)
checkpoint guard (`[[ ! -f "$called_fn_dir/.$fn" ]] || [[ $DIFF == true ]]`) → `start_func` (log, timestamp) → `run_command <tool> <args>` → pipe output through `anew` into result file → `end_func` (writes checkpoint sentinel, logs elapsed time, emits status badge)

### Parallel Execution Path
`parallel_funcs N fn_a fn_b …` → background subshells launched → `_throttle_jobs` (wait -n loop) caps concurrency at N → each job writes to a temp log → on completion, output replayed per `PARALLEL_LOG_MODE` → failures increment `RECON_*_PARALLEL_FAILURES`

### Axiom Distributed Scan Flow
`axiom_selected()` checks `AXIOM=true` → `run_command` routes through `axiom_run_command()` → uploads input list to fleet → runs tool across distributed instances → downloads merged output → `run_module_with_axiom_failover` retries locally on fleet failure
- Global bash variables throughout (no encapsulation). Config vars, target vars (`domain`, `dir`, `called_fn_dir`, `LOGFILE`), and result counters are all globals
- `passive()` saves/restores module-enable globals before overriding them (`modules/modes.sh:549-611`)
## Key Abstractions
- Purpose: Lifecycle wrapper around every leaf scanning function
- Examples: Used in every function in `modules/subdomains.sh`, `modules/web.sh`, `modules/vulns.sh`, `modules/osint.sh`
- Pattern: `start_func "${FUNCNAME[0]}" "description"` at top; `end_func "output path" "${FUNCNAME[0]}"` at bottom; creates checkpoint file on end
- Purpose: Prevent re-running completed functions across multiple invocations of the same target
- Location: `Recon/<domain>/.called_fn/.funcname`
- Pattern: Each function tests `[[ ! -f "$called_fn_dir/.${FUNCNAME[0]}" ]] || [[ $DIFF == true ]]`; `end_func` writes the sentinel via `touch "$called_fn_dir/.${fn}"`
- Purpose: Universal external-tool gate for dry-run preview, axiom dispatch, adaptive rate limiting, and debug logging
- Location: `modules/utils.sh:468`
- Pattern: All tool calls inside module functions use `run_command <binary> <args>` rather than direct invocation
- Purpose: Allow modules to be sourced multiple times (test re-sourcing, `--source-only`) without re-executing
- Pattern: `[[ -n "$_FOO_LOADED" ]] && return 0` at top of each lib file (`lib/common.sh:6`, `lib/parallel.sh:6`, `lib/ui.sh:5`, `lib/validation.sh` — validation uses error-code guards instead)
- Purpose: Run independent module functions concurrently up to `PARALLEL_MAX_JOBS`
- Pattern: `parallel_funcs N func_a func_b func_c` — each function spawned as a background subshell; used in `recon()`, `osint()`, `vulns()` for independent groups
- Purpose: Transparent axiom/local fallback wrapper — if axiom fails during a module, retries locally
- Location: `modules/modes.sh:656`
- Pattern: All module calls inside `subs_menu`, `webs_menu`, `recon`, `passive` use this wrapper
## Entry Points
- Location: `reconftw.sh`
- Triggers: Direct execution (`./reconftw.sh -d example.com -r`)
- Responsibilities: Bootstrap, all module loading, CLI parse, config source, mode dispatch
- Location: `reconftw.sh:123-125`
- Triggers: `./reconftw.sh --source-only` (used by bats test `setup()` blocks)
- Responsibilities: Sources all modules without executing any recon
- Location: `modules/modes.sh:13`
- Triggers: Called at the top of most workflow functions (`recon`, `subs_menu`, `passive`, `osint`, `zen_menu`)
- Responsibilities: Create output directory tree, init LOGFILE, init cache/incremental/DNS/plugins, set global `dir` and `called_fn_dir`
- Location: `modules/modes.sh:286`
- Triggers: Called at the bottom of most workflow functions
- Responsibilities: AI report, cleanup, Faraday, screenshot diffs, plugin events, hotlist, `export_reports()`, timing summary
## Output Directory Structure
```
Recon/<domain>/
├── .called_fn/       # checkpoint sentinels (.funcname per completed function)
├── .log/             # per-run log files
├── .tmp/             # temporary working files
├── subdomains/       # subdomain enumeration results
├── webs/             # web probe results, JS, URLs
├── hosts/            # port scan / service results
├── vulns/            # vulnerability findings
│   ├── missing_headers.txt      # dast_passive: missing CSP/HSTS/X-Frame-Options/etc
│   ├── cookie_issues.txt        # dast_passive: Set-Cookie missing Secure/HttpOnly/SameSite
│   ├── info_disclosure.txt      # dast_passive: Server version, X-Powered-By, debug headers
│   ├── mime_mismatch.txt        # dast_passive: Content-Type vs file extension mismatches
│   ├── exposed_files.txt        # exposed_files: accessible backup/source/sensitive files
│   └── http_methods.txt         # http_methods: TRACE/PUT/DELETE enabled hosts
├── osint/            # OSINT data (emails, dorks, cloud, leaks)
├── screenshots/      # webpage screenshots
└── ai_result/        # AI-generated report output
```
## Verbosity and Output Controls
- `0` (quiet): Only errors and final summary printed to terminal; banner suppressed
- `1` (normal, default): Errors + warnings printed; `notification()` info/good suppressed
- `2` (verbose): All `notification()` calls, PID info, full parallel output, `start_func` messages, `print_timing_summary`
- `summary`: One badge line per completed parallel job
- `tail`: Last `PARALLEL_TAIL_LINES` (default 20, doubled on failure) from each job's log
- `full`: Complete captured stdout from each job
- `jsonl-strict`: Forces `OUTPUT_VERBOSITY=0`, emits only machine-readable JSONL
## Architectural Constraints
- **Threading:** Single-threaded bash with optional background subshells via `parallel_funcs`; `wait -n` (bash 4.3+) used for job throttling in `_throttle_jobs`
- **Global state:** All config vars, `domain`, `dir`, `called_fn_dir`, `LOGFILE`, `start`, `runtime`, `DIFF`, `AXIOM`, and hundreds of module-enable flags are module-level globals; any sourced function can read or mutate them
- **Circular imports:** None by design — reconftw.sh sources libs first, then modules in explicit dependency order (`utils.sh` → `core.sh` → `osint.sh` → `subdomains.sh` → `web.sh` → `vulns.sh` → `axiom.sh` → `modes.sh`)
- **macOS bash version:** reconftw.sh re-execs itself under Homebrew bash ≥ 4 on macOS (system bash is 3.2); `lib/parallel.sh` requires bash 4.3+ for `wait -n`
- **Working directory:** `start()` calls `cd "$dir"` (the per-target output dir) before any module function runs; all relative paths inside modules resolve against the target dir. `reconftw.sh` captures `startdir=${PWD}` before this
- **No subshell isolation per module:** Modules are sourced functions, not subprocess commands. A `return` inside a module returns from the function; an `exit` would kill the whole shell
## DAST Passive Functions Pattern
Three functions in `modules/vulns.sh` mirror what Invicti / OWASP ZAP / Burp Suite passive scanning produces. All run as part of `vulns()` parallel group 4 and are included in `attack_map` mode.

### `dast_passive` (flag: `DAST_PASSIVE=true`)
One httpx `-json` probe across `webs/webs_all.txt` feeds four checks:
- **Missing security headers**: X-Content-Type-Options, CSP, HSTS (HTTPS-only), X-Frame-Options, Referrer-Policy, Permissions-Policy, COOP → `vulns/missing_headers.txt`
- **Cookie flag issues**: Set-Cookie missing Secure (HTTPS-only), HttpOnly, SameSite → `vulns/cookie_issues.txt`
- **Info disclosure**: Server with version digit, X-Powered-By, ASP.NET version, debug headers (x-debug-token, x-generator, x-drupal-cache, x-envoy-upstream-service-time) → `vulns/info_disclosure.txt`
- **MIME mismatches**: `.json`/`.xml`/`.js`/`.css` served as `text/html`; script extensions served as `application/octet-stream` → `vulns/mime_mismatch.txt`

### `exposed_files` (flags: `EXPOSED_FILES=true`, `EXPOSED_FILES_URL_LIMIT=400`)
Two-phase candidate generation then single httpx probe (mc 200/206, filter-length 0):
1. Backup variants (`.bak .old .orig .backup .copy .tmp .swp .save .bkp ~`) of crawled PHP/ASP/config/script URLs (capped at `EXPOSED_FILES_URL_LIMIT`)
2. ~40 hardcoded sensitive paths per live host: `.env` variants, SQL dumps, `.git/config`, `id_rsa`, `phpinfo.php`, `adminer.php`, `docker-compose.yml`, `composer.json`, log files, `swagger.json`, etc.
→ `vulns/exposed_files.txt`

### `http_methods` (flag: `HTTP_METHODS=true`)
- OPTIONS probe → parses `Allow` / `Public` / `Access-Control-Allow-Methods` for PUT/DELETE/TRACE/CONNECT
- Direct TRACE probe (mc 200) for XST confirmation
→ `vulns/http_methods.txt`

### `generate_attack_map` (`modules/modes.sh`)
Produces `attack-map.html` and `attack-map.md` summarising a complete target. HTML features:
- Stat tile grid (`repeat(auto-fill, minmax(160px, 1fr))`) — severity-colored tiles including all DAST passive counts
- Conditional two-col sections (only rendered when data is non-empty): Takeovers, High-Value Subdomains/Tech, Services, Vulnerabilities, Passive DAST (headers/cookies/disclosure), Exposed Files/HTTP Methods/MIME, CVEs/JS Secrets, Cloud/Priority
- Dark mode via `@media (prefers-color-scheme: dark)` + `[data-theme]` overrides

## Anti-Patterns
### Direct external tool calls without `run_command`
### Writing checkpoint files manually
### Skipping the `[[ ! -f "$called_fn_dir/.${FUNCNAME[0]}" ]] || [[ $DIFF == true ]]` guard
### Overriding config globals without save/restore in workflow functions
### `if ! cmd; then rc=$?` to capture a command's exit code
- Under `reconftw.sh`'s global `set +e`, `$?` after `if ! cmd; then` is the exit status of the `!` expression — always `0` when `cmd` fails, never the original non-zero code. Use `cmd || rc=$?` instead.
### `declare -A` at module scope for arrays that must be globally visible
- `declare -A` inside a sourced file creates a local array scoped to the sourcing call. Use `declare -gA` so the array is placed in the global environment regardless of how or where the file is sourced.
### Parsing space-delimited strings with `read` when ambient IFS may differ
- `reconftw.sh` sets `IFS=$'\n\t'` globally. A bare `read -r a b c <<<"$space_string"` puts the entire string into `a`. Use `IFS=' ' read -r a b c <<<"$space_string"` to override IFS inline.
### Using anchored `$` regex on external data files without stripping CRLF first
- External wordlists and FQDN databases often use Windows CRLF (`\r\n`) line endings. When piped through grep, each line retains a trailing `\r`, so `grep -E "pattern$"` silently matches nothing — the `$` anchor sits before `\n` but the last character is `\r`, not the final character of the pattern. Always pipe through `tr -d '\r'` before any anchored regex: `grep -iF "..." file | tr -d '\r' | grep -iE "pattern$"`. This applies to `sub_localdb` and any future function that reads user-supplied data files.
### Writing raw `\r`-only progress lines to the terminal
- `printf "\r  text\033[K"` (no trailing `\n`) leaves the line in a half-rendered state in the scrollback buffer; raw escape codes appear as literal characters when the user scrolls or selects text. Always use `ui_live_progress_update` for in-place status — it manages `_UI_LIVE_NEEDS_UP` and terminates every frame with `\n`.
### Printing static output while `_UI_LIVE_ACTIVE=true` without calling `ui_live_progress_break` first
- Printing static output directly to stdout while live progress is active leaves the cursor at the wrong position; the next heartbeat `\033[1A` goes to the wrong line and overwrites output with escape codes. Always call `ui_live_progress_break` (which clears the slot and resets `_UI_LIVE_NEEDS_UP=false`) before any `printf` to stdout when `_UI_LIVE_ACTIVE` may be true. `_print_status`, `ui_batch_start`, and `ui_batch_end` all do this correctly.
### DataForSEO: wrong endpoint causes silent 40400
- `/v3/dataforseo_labs/google/domain_overview/live` returns 40400 (Not Found) for domain traffic queries. The correct endpoint is `/v3/dataforseo_labs/google/domain_rank_overview/live`. Result data is nested at `.tasks[0].result[0].items[0].metrics`, not `.tasks[0].result[0].metrics`. Without this, a future session adding DataForSEO calls will get silent empty output with no error message.
### Module functions write relative paths — test harness must `cd` to the output dir
- All module functions write output to relative paths (e.g. `osint/foo.txt`) because `start()` does `cd "$dir"` before any function runs. A bare `--source-only` test that doesn't `cd "$dir"` will write output to the repo root instead, creating stray dirs. Always `(cd "$dir" && fn)` in test harnesses.
## Error Handling
- ERR trap in `start()` logs function name, line number, and command to `$LOGFILE` and calls `explain_err()` (`modules/modes.sh:140`)
- Non-zero exit from `parallel_funcs` increments `RECON_OSINT_PARALLEL_FAILURES` and sets `RECON_PARTIAL_RUN=true`
- `run_module_with_axiom_failover` catches axiom mid-run failures and retries locally
- Circuit-breaker helpers (`circuit_breaker_is_open`, `circuit_breaker_record_failure`) in `modules/utils.sh:1190` for persistent tool failures
## Cross-Cutting Concerns
- **Notifications**: `notify` (projectdiscovery/notify) called by `notification()` in `core.sh`; triggered on function completion, errors, and key findings; provider config at `~/.config/notify/provider-config.yaml`
- **Rate limiting**: `ADAPTIVE_RATE_LIMIT` adjusts per-tool rates on 429/503 errors; `circuit_breaker_*` helpers in `utils.sh` cut off tools that fail repeatedly
- **Secrets**: `redact_secrets()` / `register_secret()` scrub sensitive values from log lines before writing; `secrets.cfg` is auto-sourced and gitignored
- **Plugins**: event hooks in `core.sh` fire `start`/`end`/`finding` events to scripts in `plugins/`
- **Caching**: `cache_get` / `cache_set` in `utils.sh`; TTL controlled per resource type by `CACHE_MAX_AGE_DAYS_*` vars
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

## Claude Code Git Workflow

- **Commit after completing each discrete task** — once a logical unit of work is done (bug fixed, feature added, docs updated), create a commit. Don't batch unrelated changes into one commit.
- **Push directly — never open PRs** — push to `origin/main` immediately after each commit. Never create pull requests; direct push is always the right approach for this repo.
- **Commit message format** — use conventional commits style: `fix:`, `feat:`, `docs:`, `refactor:`, `test:`, `chore:`. Keep the subject line under 72 characters; add a body when the why is non-obvious.
- **Always include co-author trailer**:
  ```
  Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
  ```
