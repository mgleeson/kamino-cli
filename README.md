# kamino-cli

`kamino` is a small Bash utility for bulk cloning Git repositories from a line-delimited list.

It accepts repository URLs from either a text file or standard input, then clones each repository into a destination directory.

The script supports both HTTPS and SSH Git repository URLs. Authentication is intentionally left to the existing Git and SSH environment, so it works naturally with `ssh-agent`, `~/.ssh/config`, Git credential helpers, GitHub CLI authentication, SSO tooling, or CI secrets.

## Features

- Clone multiple Git repositories from a plain text list
- Optionally clone repositories into list-defined relative subdirectories
- Accept input from a file or from stdin
- Support HTTPS and SSH repository URLs
- Support a configurable destination directory
- Default destination is the current working directory
- Ignore blank lines and comment lines
- Skip repositories whose target directory already exists
- Clone into a temporary directory first, then move into place
- Prevent `git clone` from accidentally consuming the repo-list input stream
- Print a summary of cloned, skipped, and failed repositories
- Use labelled terminal output for status messages

## Requirements

- Bash
- Git
- A Unix-like shell environment

Tested target environments include Linux, macOS, WSL, and other systems with Bash and Git available.

## Installation

Clone or download this repository, then make the script executable:

```bash
chmod +x kamino
````

Optionally, place it somewhere on your `PATH`:

```bash
sudo cp kamino /usr/local/bin/kamino
```

Or for a user-local install:

```bash
mkdir -p "$HOME/.local/bin"
cp kamino "$HOME/.local/bin/kamino"
chmod +x "$HOME/.local/bin/kamino"
```

Make sure `$HOME/.local/bin` is in your `PATH` if using the user-local install method.

## Usage

```bash
kamino [-d DEST_DIR] [REPO_LIST_FILE]
kamino [REPO_LIST_FILE] [DEST_DIR]
kamino - [DEST_DIR]
```

## Examples

Clone all repositories listed in `repos.txt` into the current directory:

```bash
kamino repos.txt
```

Clone all repositories listed in `repos.txt` into `~/src/clones`:

```bash
kamino repos.txt ~/src/clones
```

Use the explicit destination option:

```bash
kamino -d ~/src/clones repos.txt
```

Pipe a repository list into `kamino`:

```bash
cat repos.txt | kamino
```

Pipe a repository list and specify a destination:

```bash
cat repos.txt | kamino -d ~/src/clones
```

Or:

```bash
cat repos.txt | kamino - ~/src/clones
```

Show help:

```bash
kamino --help
```

## Repository list format

The input file should contain one Git repository URL per line.

Example:

```text
https://github.com/example/project-one.git
git@github.com:example/project-two.git
ssh://git@example.com/example/project-three.git

# Blank lines are ignored.
# Lines beginning with # are ignored.
```

Both HTTPS and SSH-style repository URLs are supported.

Optionally, a line may use a simple two-field CSV-style format:

```text
Git repository URL, target/subdirectory
```

Example:

```text
https://github.com/example/moodle-mod_forum.git, mod/forum
https://github.com/example/moodle-block_demo.git, block/demo/
```

CSV-style target paths are relative to the destination directory. They must not be absolute paths, empty paths, `.` or `..`, or contain `..` path traversal segments.

This is a simple comma-separated format only. Lines with more than one comma are rejected.

## Destination directory behavior

If no destination directory is supplied, `kamino` clones repositories into the current working directory.

If a destination directory is supplied and does not already exist, `kamino` attempts to create it.

Each repository is cloned into a subdirectory named after the repository.

For example:

```text
https://github.com/example/project-one.git
```

will be cloned into:

```text
project-one
```

If a repository list line supplies a CSV-style target path, that repository is cloned into that relative path under the destination directory instead. Parent directories are created as needed.

For example:

```text
https://github.com/example/moodle-mod_forum.git, mod/forum
```

will be cloned into:

```text
mod/forum
```

If the target directory already exists, the repository is skipped rather than overwritten.

## Authentication

`kamino` does not manage authentication directly.

This is intentional. Git authentication is best handled by the surrounding environment.

For HTTPS repositories, configure Git credential helpers or another supported authentication mechanism.

For SSH repositories, ensure your SSH key and agent are ready before running `kamino`.

For example:

```bash
ssh-add -l
ssh -T git@github.com
```

For GitHub HTTPS authentication, you may also use the GitHub CLI:

```bash
gh auth login
```

## Output

`kamino` prints labelled status messages while processing repositories.

At the end of a run, it prints a summary showing:

* input source
* destination directory
* number of repositories processed
* number cloned
* number skipped
* number failed

Example summary:

```text
[ DONE ]          Summary
  source:      repos.txt
  destination: /home/user/src/clones
  processed:   5
  cloned:      4
  skipped:     1
  failed:      0
```

## Exit codes

`kamino` uses the following exit codes:

| Exit code | Meaning                                                                                            |
| --------: | -------------------------------------------------------------------------------------------------- |
|       `0` | All repositories were cloned successfully, or skipped because the target directory already existed |
|       `1` | One or more repositories failed to clone                                                           |
|       `2` | Usage, input, dependency, or environment error                                                     |
|     `130` | Interrupted by `SIGINT` or `SIGTERM`                                                               |

## Safety notes

`kamino` clones each repository into a temporary directory inside the destination directory first.

Only after a successful clone does it move the temporary directory into its final target location.

This avoids leaving partially cloned repositories in the expected final location.

The script also runs `git clone` with stdin redirected from `/dev/null`, which prevents Git, SSH, or credential prompts from accidentally consuming the repository list stream when input is piped.

## Limitations

`kamino` is intentionally simple and conservative.

It does not currently:

* update existing repositories
* overwrite existing target directories
* retry failed clone operations
* clone in parallel
* validate repository URLs before cloning
* manage SSH keys or HTTPS credentials
* support quoted CSV fields or commas inside repository-list fields

Existing directories are skipped rather than modified.

## Troubleshooting

### `git is required but was not found in PATH`

Install Git and ensure it is available in your shell:

```bash
git --version
```

### SSH repository clone fails

Check that your SSH key is loaded:

```bash
ssh-add -l
```

Test your Git host connection:

```bash
ssh -T git@github.com
```

For non-GitHub hosts, replace `git@github.com` with the appropriate host.

### HTTPS repository clone asks for credentials or fails

Configure a Git credential helper or authenticate using your provider’s tooling.

For GitHub, one common option is:

```bash
gh auth login
```

### A repository was skipped

A repository is skipped when the target directory already exists.

For example, this repository:

```text
https://github.com/example/project-one.git
```

will be skipped if this directory already exists inside the destination:

```text
project-one
```

Remove, rename, or move the existing directory before running `kamino` again if you want to clone it fresh.


## License

This script is licenced under the GNU Affero General Public License v3.0
