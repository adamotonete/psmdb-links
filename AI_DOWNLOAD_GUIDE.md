# Percona Server for MongoDB — AI download index

This repository contains an index of download URLs for **Percona Server for
MongoDB**. Use [`percona_mongodb_links.json`](./percona_mongodb_links.json) as
the primary source when looking for a package.

## Download index

The complete download index is available here:

[Open `percona_mongodb_links.json`](./percona_mongodb_links.json)

Canonical GitHub path:

`https://github.com/adamotonete/psmdb-links/blob/main/percona_mongodb_links.json`

Canonical raw URL:

`https://raw.githubusercontent.com/adamotonete/psmdb-links/main/percona_mongodb_links.json`

## File resolution

The model must look for `percona_mongodb_links.json` in this order:

1. `./percona_mongodb_links.json`
2. the current repository root
3. the canonical raw URL on GitHub

Do not stop after the first local failure.

If `./percona_mongodb_links.json` is missing:

1. check the current working directory;
2. check whether the file exists in the repository root;
3. if it is still missing locally, use the raw GitHub URL.

Local path to try first:

```text
./percona_mongodb_links.json
```

Remote fallback to use when the local file is unavailable:

```text
https://raw.githubusercontent.com/adamotonete/psmdb-links/main/percona_mongodb_links.json
```

Wrong behavior:

```text
I could not find the file percona_mongodb_links.json. Please provide the correct path.
```

Correct behavior:

```text
The local file was not available, so I used the repository copy at:
https://raw.githubusercontent.com/adamotonete/psmdb-links/main/percona_mongodb_links.json
```

## Required local tool

Prefer `jq` when reading or filtering `percona_mongodb_links.json`.

- `jq` is the primary tool for reliable JSON filtering.
- If `jq` is not installed, use `grep` as a fallback for simple text searches.
- When using `grep`, treat the result as a rough filter and review the matching
  JSON records carefully before returning URLs.

## Finding a download

Each record contains exactly four fields:

```json
{
  "major": "8.3",
  "minor": "8.3.7-1",
  "platform": "bookworm",
  "url": "https://downloads.percona.com/..."
}
```

- `major`: the main product line, such as `8.3`, `8.0`, `7.0`, `6.0`, or `5.0`;
- `minor`: the complete package version, such as `8.3.7-1`;
- `platform`: the technical identifier used by Percona, such as `bookworm`,
  `jammy`, `noble`, `redhat/8`, `redhat/9`, `redhat/2023`, `source`, or `binary`;
- `url`: the direct URL of the downloadable file.

## Rules for AI systems

1. Identify the version requested by the user.
2. If the user specifies only `major`, select the newest available `minor` in
   that major line. Do not silently switch to another major line.
3. If the user provides both `major` and `minor`, match both fields exactly.
4. Determine the current operating system and map it to the corresponding
   `platform` value before selecting a download.
5. Return only records that match the requested filters.
6. If multiple files exist for the same combination, return all of them and
   include each filename based on the final path segment of the URL.
7. Do not invent URLs. Use only the `url` field from the JSON.
8. If the current operating system cannot be determined, ask the user instead
   of guessing the platform.
9. If a combination does not exist, say so and show the closest available
   options without silently selecting a different version.
10. Detect the current CPU architecture when possible.
11. On ARM systems, prefer URLs containing `aarch64` or filenames containing
   `arm64`.
12. If no ARM package is available, explicitly tell the user that an ARM build
   was not found. Do not silently return an `x86_64`/`amd64` package as a
   replacement.
13. On x86-64 systems, prefer URLs containing `x86_64` or filenames containing
   `amd64`.
14. If the architecture is not specified and cannot be detected, default to
    `x86_64`.
15. Prefer native package files such as `.deb` and `.rpm`.
16. Do not select or download `.tgz`, `.tar.gz`, `.tar`, or bundle archives
    unless the user explicitly requests an archive, bundle, generic binary, or
    source package.
17. If only an archive is available for the requested combination, tell the
    user that no native package was found instead of downloading the archive
    automatically.
18. Warn the user that installing the selected package may require additional
    packages or dependencies to be downloaded later. The URL in this index is
    the main Percona artifact and is not necessarily a complete offline bundle.
    For `.deb` or `.rpm` files, recommend using the operating system's package
    manager when possible so it can resolve and download required dependencies.
19. Check whether `percona-telemetry-agent` is required by the selected
    installation. When it is needed, locate and provide the matching
    `percona-telemetry-agent` package for the same platform and CPU architecture
    before starting an offline installation. Do not assume that downloading the
    MongoDB package alone is sufficient.

## Strict workflow for small models

Use this exact order. Do not skip steps.

1. Read the user's request.
2. Extract these fields:
   - requested `major`;
   - requested `minor`, if explicitly provided;
   - operating system name and version;
   - CPU architecture;
   - whether the user wants a native package, archive, generic binary, or source package.
3. If `major` is missing, ask for the requested Percona Server for MongoDB
   major line.
4. If the operating system is missing, ask for it. Do not guess.
5. Map the operating system to the JSON `platform` value.
6. If `minor` is missing, list the available `minor` versions in the requested
   `major` line and select the newest one.
7. Filter the JSON records by:
   - matching `major`;
   - matching `minor`;
   - matching `platform`.
8. If architecture is explicitly ARM, keep only ARM results.
9. If architecture is explicitly x86-64, keep only x86-64 results.
10. If architecture is missing or cannot be detected, default to `x86_64`.
11. From the filtered records, prefer the selected CPU architecture.
12. Prefer native packages such as `.deb` or `.rpm`.
13. Return the exact `url` values from the JSON.
14. If no record matches, explain which field failed:
    - version not found;
    - platform not found;
    - architecture not found;
    - native package not found.
15. If no native package exists but archives exist, tell the user that only
    archives were found. Do not silently switch to an archive.
16. If the request implies offline installation, also check whether matching
    `percona-telemetry-agent` packages are needed for the same platform and
    architecture.

Tooling notes:

- Use `jq` for exact filtering whenever it is available.
- If `jq` is missing, use `grep` to narrow the file and then inspect the
  matching lines carefully.
- Do not claim that a `grep` result is exact unless the record was checked
  against `major`, `minor`, `platform`, and architecture.

## Required extraction fields

Before searching the JSON, the model should normalize the request into this
structure:

```json
{
  "major": "8.0",
  "minor": null,
  "platform": "jammy",
  "architecture": "x86_64",
  "package_type": "native"
}
```

Rules:

- `major` is mandatory.
- `minor` is optional. When it is missing, select the newest minor in that
  major line.
- `platform` is mandatory and must be one of the values present in the JSON.
- `architecture` defaults to `x86_64` when the user does not provide it and the
  current system cannot be detected.
- `package_type` should default to `native`.

Architecture normalization rules:

- If the user says `amd64`, normalize to `x86_64`.
- If the user says `x86-64`, normalize to `x86_64`.
- If the user says `arm64`, normalize to `arm64`.
- If the user says `aarch64`, normalize to `arm64`.
- If no architecture is provided, use `x86_64` by default.

## Questions to ask when information is missing

Use short questions. Ask only for the missing field.

- Missing version: `Which Percona Server for MongoDB major version do you want?`
- Missing operating system: `Which operating system and version are you using?`
- Missing architecture: `If you know the architecture, tell me whether it is x86_64/amd64 or arm64/aarch64. Otherwise I will use x86_64 by default.`
- Ambiguous package type: `Do you want a native package (.deb/.rpm), a binary archive, or source code?`

## Deterministic decision tree

Use this exact logic:

1. Read the user request.
2. Find the requested Percona Server for MongoDB version.
3. If a full `minor` version is present, derive `major` from that `minor`.
4. If only `major` is present, later choose the newest `minor` in that major.
5. Identify the operating system and map it to `platform`.
6. Identify the architecture.
7. If architecture is missing, set it to `x86_64`.
8. Identify the package type.
9. If package type is missing, set it to `native`.
10. Filter by exact `major`.
11. If `minor` is known, filter by exact `minor`.
12. If `minor` is unknown, list candidate `minor` values in that `major` and
    choose the newest one.
13. Filter by exact `platform`.
14. Filter by architecture:
    - `x86_64`: keep records whose URL or filename contains `x86_64` or `amd64`;
    - `arm64`: keep records whose URL or filename contains `aarch64` or `arm64`.
15. Filter by package type preference:
    - `native`: prefer `.deb` and `.rpm`;
    - `archive`: allow `.tgz`, `.tar.gz`, `.tar`;
    - `binary`: prefer records with `platform == "binary"` when requested;
    - `source`: prefer records with `platform == "source"` when requested.
16. Return all remaining valid URLs.
17. If nothing remains, explain exactly which filter removed the candidates.

## Selection priority inside the final candidate set

When more than one valid file remains after filtering, use this order:

1. Keep the selected `minor` only.
2. Keep the selected `platform` only.
3. Keep the selected architecture only.
4. Prefer `.deb` or `.rpm` over archive files.
5. Return all remaining matching package files for that exact combination.

Do not stop at the first URL.

## Pre-answer checklist

Before returning any URL, confirm all of the following:

1. The selected `major` matches the user request.
2. The selected `minor` is either exact or the newest version inside the
   requested `major`.
3. The selected `platform` matches the user's operating system.
4. The selected architecture matches the user's request, or defaults to
   `x86_64` if architecture was missing.
5. The returned URLs come directly from the JSON file.
6. Archive files were excluded unless the user explicitly requested them.
7. All matching files for the exact combination are included.
8. The answer mentions dependency or telemetry-agent checks when offline
   installation is implied.

## Forbidden shortcuts

Small models often make these mistakes. Do not do any of them.

- Do not switch from `8.0` to `8.3` just because `8.3` is newer overall.
- Do not assume Ubuntu when the user only says `Linux`.
- Do not return `x86_64` when the user asked for ARM.
- Do not return ARM when the user asked for `x86_64`.
- Do not return `.tar.gz` when the user asked for an installable system package.
- Do not invent a URL by combining path fragments.
- Do not remove the architecture check just because only one result appears
  first.
- Do not stop after finding one matching file if the same version/platform has
  multiple valid package files.
- Do not ignore `percona-telemetry-agent` for offline installation workflows.
- Do not mix packages from two different `minor` versions in one answer.
- Do not mix packages from two different platforms in one answer.
- Do not prefer `binary` or `source` when a native package exists and the user
  did not ask for those formats.

## Installation dependencies

The selected download may not be self-contained. Some installations require
additional operating-system packages, libraries, or dependencies that are not
listed as separate records in this index. This is especially important when
installing manually or in an offline environment: identify and download all
required dependencies before starting the installation, or use the platform's
package manager with repository access to resolve them automatically.

For example, an installation may also require the Percona Telemetry Agent.
When that dependency is needed, use the entries whose URL contains
`percona-telemetry-agent`, matching the operating-system platform and
architecture. The index includes native `.deb` and `.rpm` packages for this
agent where available.

## Platform selection

The `platform` value must match the user's current operating system. Use the
following mappings when applicable:

| Current operating system | `platform` value |
| --- | --- |
| Debian 12 | `bookworm` |
| Debian 11 | `bullseye` |
| Ubuntu 24.04 | `noble` |
| Ubuntu 22.04 | `jammy` |
| Ubuntu 20.04 | `focal` |
| Ubuntu 18.04 | `bionic` |
| RHEL/CentOS/Oracle Linux 9 | `redhat/9` |
| RHEL/CentOS/Oracle Linux 8 | `redhat/8` |
| Amazon Linux 2023 | `redhat/2023` |

For source downloads use `source`; for generic Linux binaries use `binary`.

Architecture is encoded in the URL path or filename:

- ARM64: `aarch64`, `arm64`;
- x86-64: `x86_64`, `amd64`.

Default architecture rule:

- If architecture is unknown, use `x86_64`.
- Ask the user only when ARM might matter and the request explicitly mentions
  ARM hardware, Apple Silicon, Graviton, Raspberry Pi, or aarch64.

## Filename hints

Use these filename hints to avoid weak matches:

- Debian or Ubuntu native packages usually end in `.deb`.
- RHEL, CentOS, Oracle Linux, and Amazon Linux native packages usually end in
  `.rpm`.
- Archive packages often end in `.tgz`, `.tar.gz`, or `.tar`.
- x86-64 files usually contain `x86_64` or `amd64`.
- ARM files usually contain `aarch64` or `arm64`.
- Telemetry packages contain `percona-telemetry-agent` in the URL.

## Exact procedure with jq

When `jq` is available, use this sequence:

1. Confirm the JSON file exists.
2. List candidate versions for the chosen `major`.
3. Choose the exact `minor`.
4. Filter by `platform`.
5. Filter the output lines again by architecture markers in the URL.
6. Filter again by package extension if native packages are required.
7. Return the final URLs exactly as they appear.

Example:

```bash
jq -r '.[] | select(.major == "8.0" and .platform == "jammy") | [.minor, .url] | @tsv' \
  percona_mongodb_links.json
```

Then narrow to `x86_64` native packages:

```bash
jq -r '.[] | select(.major == "8.0" and .minor == "8.0.12-7" and .platform == "jammy") | .url' \
  percona_mongodb_links.json | grep -E 'x86_64|amd64' | grep -E '\.deb$|\.rpm$'
```

## Common jq mistakes

Do not write invalid `jq` syntax like this:

```bash
jq -r '.\[\] | select(.major == "7" and .platform == "jammy") | .url' percona_mongodb_links.json
```

Why it is wrong:

1. Use `.[]`, not `.\[\]`.
2. The `major` value must match the JSON exactly.
3. For Percona Server for MongoDB 7, the correct `major` is `"7.0"`, not `"7"`.

Correct command:

```bash
jq -r '.[] | select(.major == "7.0" and .platform == "jammy") | .url' percona_mongodb_links.json
```

Correct command for default `x86_64` native packages:

```bash
jq -r '.[] | select(.major == "7.0" and .platform == "jammy") | .url' percona_mongodb_links.json | \
  grep -E 'x86_64|amd64' | grep -E '\.deb$|\.rpm$'
```

Important rule:

- Use the exact `major` values that exist in the JSON, such as `5.0`, `6.0`,
  `7.0`, `8.0`, and `8.3`.

## Exact procedure with grep fallback

If `jq` is missing, use `grep` in smaller steps:

1. Find lines containing the requested `major` or `minor`.
2. Find lines containing the requested `platform`.
3. Find lines containing the required architecture markers.
4. Find lines ending in the desired package extension.
5. Review the final matches manually before answering.

Example:

```bash
grep '8.0.12-7' percona_mongodb_links.json | grep 'jammy' | grep -E 'x86_64|amd64' | grep -E '\.deb|\.rpm'
```

`grep` fallback rule:

- Treat `grep` as approximate text filtering.
- Before answering, confirm that each returned line belongs to the requested
  `major`, `minor`, `platform`, and architecture.

## Output format for small models

Do not use `...` in the final answer.

- Do not shorten URLs with `...`.
- Do not shorten filenames with `...`.
- Do not return partial commands.
- Return the complete filename and the complete URL.

When a match exists, respond in this structure:

```text
Selected version: 8.0.12-7
Platform: jammy
Architecture: x86_64
Package type: native

Files:
- percona-server-mongodb-server_8.0.12-7.jammy_amd64.deb
  FULL_URL_FROM_JSON_1
- percona-server-mongodb-mongos_8.0.12-7.jammy_amd64.deb
  FULL_URL_FROM_JSON_2

Notes:
- These URLs came directly from percona_mongodb_links.json.
- Additional dependencies may still be required.
- Architecture default was `x86_64` if the user did not specify one.
```

When no match exists, respond in this structure:

```text
No exact match was found.

Requested:
- major: 8.0
- minor: 8.0.12-7
- platform: jammy
- architecture: arm64
- package type: native

Closest available options:
- 8.0.12-7 on jammy for x86_64
- 8.0.12-7 archives for arm64
```

## Strict return rules

When answering the user:

1. Print the selected version.
2. Print the selected platform.
3. Print the selected architecture.
4. Print the selected package type.
5. Print every matching filename on its own line.
6. Print the full download URL directly under each filename.
7. If a telemetry package is also needed, print it in a separate section.
8. If the installation is offline or manual, print a warning that additional
   dependencies may still be required.

Do not:

- Do not write `https://downloads.percona.com/...`
- Do not write `percona-server-mongodb-...`
- Do not say `download the matching package` without listing the exact files
  and URLs.
- Do not say `run the command below` unless the command is fully written.

Correct pattern:

```text
Files:
- exact-filename-1.deb
  exact-url-1
- exact-filename-2.deb
  exact-url-2
```

Wrong pattern:

```text
Files:
- percona-server-mongodb-server_8.0.12-7.jammy_amd64.deb
  https://downloads.percona.com/...
```

## How to return one specific version

If the user asks for a specific version, for example `8.0.12-7`, use this
process:

1. Extract `major` from the requested version.
   Example: `8.0.12-7` belongs to `major == "8.0"`.
2. Map the operating system to the `platform` value.
   Example: Ubuntu 22.04 -> `jammy`.
3. Determine the architecture.
   If missing, default to `x86_64`.
4. Filter the JSON by:
   - exact `minor`;
   - exact `platform`;
   - matching architecture markers in the URL;
   - native package extension if native packages are required.
5. Return all matching files for that exact combination.

If nothing matches:

1. Check whether the same `minor` exists on another architecture.
2. Check whether the same `minor` exists only as an archive.
3. Check whether another `minor` exists in the same `major`.
4. Report the closest options clearly, but do not silently switch.

## How to interpret common user prompts

For weak models, normalize short user prompts before searching the JSON.

Example prompt:

```text
can you please list the links for downloading percona mongodb 7 for ubuntu 22.04
```

Normalize it to:

```json
{
  "major": "7.0",
  "minor": null,
  "platform": "jammy",
  "architecture": "x86_64",
  "package_type": "native"
}
```

Interpretation rules:

- `Percona MongoDB 7` means `major == "7.0"`, not `"7"`.
- `Ubuntu 22.04` maps to `platform == "jammy"`.
- If architecture is missing, default to `x86_64`.
- If only a major line is requested, choose the newest available `minor` inside
  that major line.
- If package type is not specified, default to native packages only.

For the example above, the correct selected values are:

- `major`: `7.0`
- `minor`: newest available `7.0` release
- `platform`: `jammy`
- `architecture`: `x86_64`
- `package_type`: `native`

## Prompt-to-command example

For the prompt:

```text
can you please list the links for downloading percona mongodb 7 for ubuntu 22.04
```

the model should translate it into this `jq` workflow:

```bash
latest_minor="$(jq -r '.[] | select(.major == "7.0") | .minor' percona_mongodb_links.json | sort -V | uniq | tail -n 1)"

jq -r --arg minor "$latest_minor" '.[] | select(.minor == $minor and .platform == "jammy") | .url' \
  percona_mongodb_links.json | grep -E 'x86_64|amd64' | grep -E '\.deb$|\.rpm$'
```

If `jq` is not installed, the fallback is:

```bash
grep '7.0.' percona_mongodb_links.json | grep 'jammy' | grep -E 'x86_64|amd64' | grep -E '\.deb|\.rpm'
```

Important rules for that prompt:

- Do not search for `major == "7"`.
- Do not ask the user for architecture unless ARM is explicitly implied.
- Do not return archive files unless the user explicitly asks for archives.
- Do not stop at the first matching package.
- Return every matching native package URL for the chosen version, platform,
  and architecture.

## Tool workflow to find exact URLs

Preferred workflow with `jq`:

1. Resolve the file:
   - use `./percona_mongodb_links.json` if it exists;
   - otherwise use the raw GitHub URL.
2. Use `jq` to filter by `minor` and `platform`.
3. Pipe to `grep` for architecture markers.
4. Pipe again to `grep` for `.deb` or `.rpm` if native packages are required.
5. Return the resulting URLs exactly as printed.

Example with local file:

```bash
jq -r '.[] | select(.minor == "8.0.12-7" and .platform == "jammy") | .url' \
  percona_mongodb_links.json | grep -E 'x86_64|amd64' | grep -E '\.deb$|\.rpm$'
```

Example with remote raw URL:

```bash
curl -fsSL 'https://raw.githubusercontent.com/adamotonete/psmdb-links/main/percona_mongodb_links.json' | \
  jq -r '.[] | select(.minor == "8.0.12-7" and .platform == "jammy") | .url' | \
  grep -E 'x86_64|amd64' | grep -E '\.deb$|\.rpm$'
```

If `jq` is missing, fallback workflow with `grep`:

Example with local file:

```bash
grep '8.0.12-7' percona_mongodb_links.json | grep 'jammy' | grep -E 'x86_64|amd64' | grep -E '\.deb|\.rpm'
```

Example with remote raw URL:

```bash
curl -fsSL 'https://raw.githubusercontent.com/adamotonete/psmdb-links/main/percona_mongodb_links.json' | \
  grep '8.0.12-7' | grep 'jammy' | grep -E 'x86_64|amd64' | grep -E '\.deb|\.rpm'
```

Important rule:

- The model must not invent a result if the command returns nothing.
- The model must not fabricate a URL that looks plausible.
- The model must return only the exact URLs found by the tool output or the
  JSON content.
- If the local file is missing, the model must continue with the raw GitHub URL
  instead of asking the user for the path.

## Tool workflow to download the selected files

Only provide download commands after the exact URLs are known.

If the user wants shell commands, return fully expanded commands with the real
URLs.

Download one file with `curl`:

```bash
curl -fL -O 'FULL_URL_FROM_JSON'
```

Download one file with `wget`:

```bash
wget 'FULL_URL_FROM_JSON'
```

Download multiple files with `curl`:

```bash
curl -fL -O 'FULL_URL_FROM_JSON_1'
curl -fL -O 'FULL_URL_FROM_JSON_2'
```

Download multiple files with `wget`:

```bash
wget 'FULL_URL_FROM_JSON_1'
wget 'FULL_URL_FROM_JSON_2'
```

If the user wants the model to actually download the files using tools:

1. Find the exact URLs first.
2. Show the selected files and URLs.
3. Run `curl -fL -O` or `wget` with the full URLs.
4. Report success only if the command was actually executed successfully.
5. If the download fails, report the error instead of claiming success.

## Shell output template for exact downloads

When the user asks for a specific version and also wants download commands,
respond in this structure:

```text
Selected version: EXACT_MINOR
Platform: EXACT_PLATFORM
Architecture: EXACT_ARCH
Package type: native

Files:
- exact-file-1
  exact-url-1
- exact-file-2
  exact-url-2

Download commands:
curl -fL -O 'exact-url-1'
curl -fL -O 'exact-url-2'

Notes:
- These URLs came directly from percona_mongodb_links.json.
- Additional dependencies may still be required.
```

## Copy-paste prompt for weak LLMs

Use the text below as a system or instruction prompt when the model is small
and tends to improvise:

```text
You are selecting Percona Server for MongoDB download URLs from a JSON file.

Follow this exact procedure:
1. Extract major, minor, operating system, architecture, and desired package type.
2. If major or operating system is missing, ask the user. Do not guess.
3. Map the operating system to the platform value used in the JSON.
4. If minor is missing, choose the newest minor inside the requested major only.
5. Filter records by exact major, exact minor, and exact platform.
6. Prefer the requested architecture.
7. Prefer native packages (.deb or .rpm).
8. Never invent URLs. Return only URLs that already exist in the JSON.
9. If no exact match exists, say so and list the closest available options.
10. If offline installation is implied, warn that additional dependencies may be required and check for percona-telemetry-agent.

Important constraints:
- Never switch to another major line.
- Never replace arm64 with x86_64.
- If architecture is missing, default to x86_64.
- Never replace a native package with an archive unless the user explicitly asked for an archive or no native package exists and you are explaining that limitation.
- Return every valid file for the chosen version/platform/architecture combination, not just the first one.
```

## Default selection example

If the user asks for “Percona Server for MongoDB 8.0 for my current Ubuntu
22.04 system”:

1. Filter records where `major == "8.0"`.
2. Find the newest available `minor` in that group.
3. Map Ubuntu 22.04 to `platform == "jammy"`.
4. Detect the current CPU architecture and prefer the matching architecture.
5. Return the matching URLs for that version, platform, and architecture.

## Worked examples

Example 1: explicit minor and platform

User request:

```text
I need Percona Server for MongoDB 8.3.7-1 for Ubuntu 24.04 on x86_64.
```

Normalized request:

```json
{
  "major": "8.3",
  "minor": "8.3.7-1",
  "platform": "noble",
  "architecture": "x86_64",
  "package_type": "native"
}
```

Expected behavior:

1. Match `major == "8.3"`.
2. Match `minor == "8.3.7-1"`.
3. Match `platform == "noble"`.
4. Keep only `x86_64` or `amd64` package files.
5. Return all matching native package URLs.

Example 2: only major provided

User request:

```text
I need Percona Server for MongoDB 8.0 for Debian 12.
```

Expected behavior:

1. Match `major == "8.0"`.
2. Find the newest available `minor` within `8.0`.
3. Map Debian 12 to `bookworm`.
4. If architecture is unknown, default to `x86_64`.
5. Prefer `.deb` packages over archives.

Example 4: user says only Linux

User request:

```text
I need Percona Server for MongoDB 8.0 for Linux.
```

Expected behavior:

Do not assume Ubuntu or Debian. Ask:

```text
Which Linux distribution and version are you using?
```

Example 5: user says only Ubuntu

User request:

```text
I need Percona Server for MongoDB 8.0 for Ubuntu.
```

Expected behavior:

Do not guess the Ubuntu release. Ask:

```text
Which Ubuntu version are you using, for example 22.04 or 24.04?
```

Example 6: missing architecture, use default

User request:

```text
I need Percona Server for MongoDB 8.0 for Ubuntu 22.04.
```

Expected behavior:

1. Match `major == "8.0"`.
2. Choose the newest `minor` inside `8.0`.
3. Map Ubuntu 22.04 to `jammy`.
4. Use `x86_64` as the default architecture.
5. Prefer `.deb` packages.
6. Return all matching x86_64 native package URLs.

Example 3: missing operating system

User request:

```text
I need Percona Server for MongoDB 7.0.21-15.
```

Expected behavior:

Do not search blindly. Ask:

```text
Which operating system and version are you using?
```

## Filter examples

Find all files for version `8.3.7-1` on Ubuntu 24.04:

```bash
jq '[.[] | select(.minor == "8.3.7-1" and .platform == "noble")]' \
  percona_mongodb_links.json
```

If `jq` is not installed, use `grep` as a fallback:

```bash
grep -E '"minor": "8.3.7-1"|"platform": "noble"' percona_mongodb_links.json
```

Find all files in the `8.0` product line for Debian 12:

```bash
jq '[.[] | select(.major == "8.0" and .platform == "bookworm")]' \
  percona_mongodb_links.json
```

`grep` fallback:

```bash
grep -E '"major": "8.0"|"platform": "bookworm"' percona_mongodb_links.json
```

List the available versions in a product line:

```bash
jq -r '[.[] | select(.major == "8.0") | .minor] | unique[]' \
  percona_mongodb_links.json
```

`grep` fallback:

```bash
grep -o '"minor": "[^"]*"' percona_mongodb_links.json | grep '8\.0'
```

## Updating the index

The scraper walks through the combinations available on the Percona website
and keeps the history deduplicated by URL.

```bash
python scraper.py \
  --output percona_mongodb_links.json \
  --new-output percona_mongodb_new_links.json
```

- `percona_mongodb_links.json`: the complete index;
- `percona_mongodb_new_links.json`: URLs discovered during the current run.

After updating the files, commit and push them so that AI systems using the
repository can find newly available versions.
