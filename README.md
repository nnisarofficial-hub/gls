# Week 1 Project — `gls`
### A reimplementation of the `ls` command in Go

---

## What You're Building

`ls` is one of the most-used commands in any terminal. You've typed it hundreds of times. This week you're building your own version of it in Go — called `gls` — from scratch.

By the end of this project you'll have a working binary that lists files and directories, supports multiple flags, outputs colored text, and is published as a downloadable release on GitHub.

This is not a tutorial. You will figure things out. The instructions tell you **what** to build and **where to look** — not what to type. That's intentional.

---

## What You'll Learn

- How Go projects are structured (`go.mod`, `cmd/`, package layout)
- Reading the filesystem with the `os` and `path/filepath` packages
- Parsing command-line flags with the `flag` package
- Formatting and printing tabular output with `fmt`
- Coloring terminal output with ANSI escape codes
- Writing table-driven tests
- Setting up a GitHub repository with CI and goreleaser releases

---

## Prerequisites

Make sure these are installed before you start:

```bash
go version        # should print go1.21 or higher
git --version     # any recent version
gh --version      # GitHub CLI — install from https://cli.github.com
```

You'll also need:
- A GitHub account
- VS Code with the [Go extension](https://marketplace.visualstudio.com/items?itemName=golang.go) installed
- `gopls` installed — VS Code will prompt you on first open, say yes

---

## Project Requirements

Here is the full spec for `gls`. Build it feature by feature — don't try to write everything at once.

### Basic behavior

```bash
gls               # list files in the current directory
gls /tmp          # list files in /tmp
gls ~/Documents   # list files in ~/Documents
```

### Flags

| Flag | Behavior |
|---|---|
| `-l` | Long format: show permissions, size, and last modified time for each entry |
| `-a` | Show all files including hidden ones (names starting with `.`) |
| `-r` | Recursive: list all subdirectories too, not just the top level |
| `-h` | Show a help message and exit |

Flags can be combined: `gls -la /tmp` should show all files including hidden ones in long format.

### Output format

**Default (no flags):**
```
Documents/    Downloads/    go/    notes.txt    .zshrc
```
Files and directories listed on one line, separated by spaces. Directories have a trailing `/`.

**Long format (`-l`):**
```
drwxr-xr-x    4096    Jan 15 09:32    Documents/
drwxr-xr-x    4096    Jan 15 09:32    Downloads/
-rw-r--r--    1823    Jan 14 22:11    notes.txt
```
Columns: permissions, size in bytes, last modified, name. Align columns neatly.

**Color output (always on):**
- Directories → Blue
- Executable files → Green
- Regular files → White (default terminal color)
- Hidden files → Dimmed/grey

### Error handling

```bash
gls /does/not/exist   # print: gls: /does/not/exist: no such file or directory
gls /root             # print: gls: /root: permission denied
```
Print errors to `stderr`. Exit with code 1 on error.

---

## Step-by-Step Build Guide

Work through these steps in order. Each step produces something that runs.

---

### Step 1 — Set up the project

Create a new directory called `gls`. Initialize it as a Go module:

```bash
mkdir gls
cd gls
go mod init github.com/<your-username>/gls
```

Create the entry point:
```
gls/
├── cmd/
│   └── gls/
│       └── main.go    ← your entry point
├── go.mod
```

In `main.go`, start with a program that just prints `"hello"` and makes sure it compiles:

```bash
go run ./cmd/gls
```

> **Why `cmd/gls/main.go` and not just `main.go` in the root?**
> This is the standard Go project layout. When a project grows, you might have multiple binaries (`cmd/server`, `cmd/worker`, etc.). Starting with this structure costs nothing now and saves a painful refactor later.

---

### Step 2 — List files in the current directory

Read the Go documentation for [`os.ReadDir`](https://pkg.go.dev/os#ReadDir). This is the function you'll use to get a list of files.

Make `gls` (with no arguments) print the names of all files in the current directory, one per line.

```bash
go run ./cmd/gls
# should print the files in your gls project directory
```

Then make it accept an optional path argument:

```bash
go run ./cmd/gls /tmp
```

If no path is given, use `.` (current directory). Look at `os.Args` to read command-line arguments.

> **Hint:** `os.ReadDir` returns `[]os.DirEntry`. Each `DirEntry` has a `.Name()` method and an `.IsDir()` method.

---

### Step 3 — Add flag parsing

Read the Go documentation for the [`flag`](https://pkg.go.dev/flag) package.

Add these flags:
- `-l` (bool, default false)
- `-a` (bool, default false)
- `-r` (bool, default false)

After calling `flag.Parse()`, the remaining arguments (the path) are in `flag.Args()`.

Test that flags work:

```bash
go run ./cmd/gls -l
go run ./cmd/gls -a /tmp
go run ./cmd/gls -l -a -r
```

At this point the flags don't do anything yet — just make sure they parse without errors.

> **Watch out:** `flag.Parse()` must be called before you read any flag values. Call it early in `main`.

---

### Step 4 — Implement the `-a` flag (hidden files)

By default, `gls` should skip files whose names start with `.` (dotfiles like `.zshrc`, `.gitignore`, the special `.` and `..` entries).

When `-a` is passed, include them.

```bash
go run ./cmd/gls -a
# should now show .gitignore, go.sum, etc.
```

---

### Step 5 — Add directory trailing slash and color

When printing names:
- If an entry is a directory, append `/` to its name
- Apply ANSI color codes based on file type

ANSI color codes work by printing a special escape sequence before the text and a reset code after:

```go
// These are the escape sequences you need
const (
    colorReset  = "\033[0m"
    colorBlue   = "\033[34m"  // directories
    colorGreen  = "\033[32m"  // executables
    colorDim    = "\033[2m"   // hidden files
    // regular files use no color (colorReset is enough)
)

// Usage:
fmt.Printf("%s%s%s  ", colorBlue, name+"/", colorReset)
```

To check if a file is executable, you need its permission bits. Use `entry.Info()` to get `fs.FileInfo`, then check `info.Mode()&0111 != 0`.

After this step, running `gls` in your project directory should show colorized output.

---

### Step 6 — Implement the `-l` flag (long format)

Long format shows: permissions, size, last modified, name — in aligned columns.

You need `entry.Info()` (returns `fs.FileInfo`) for:
- `info.Mode()` → permissions
- `info.Size()` → size in bytes
- `info.ModTime()` → last modified time

Format the permission bits with `info.Mode().String()` — Go does this for you.

Format the time: `info.ModTime().Format("Jan 02 15:04")` gives you `Jan 15 09:32`.

For aligned columns, use `fmt.Fprintf` with width specifiers. For example:
```go
fmt.Fprintf(w, "%-10s  %8d  %s  %s\n", permissions, size, modTime, name)
```

> **Hint:** Use a `tabwriter` from the [`text/tabwriter`](https://pkg.go.dev/text/tabwriter) package to align columns automatically without manually calculating widths. Read the docs — it's worth knowing.

---

### Step 7 — Implement the `-r` flag (recursive)

When `-r` is given, after listing the contents of the current directory, print a blank line then recurse into each subdirectory:

```
Documents/
  report.pdf
  notes.txt

Downloads/
  archive.zip
```

Write a recursive function. It should accept a path and an indent level (so nested directories get indented).

> **Think about:** what happens when a directory is very deeply nested, or when there's a symlink that creates a cycle? You don't need to handle infinite cycles for this project, but think about why it's a concern.

---

### Step 8 — Clean up and write tests

**Clean up:** Go through your code and make sure:
- No `fmt.Println` used for errors — use `fmt.Fprintln(os.Stderr, ...)` instead
- Exit with code `1` on errors: `os.Exit(1)`
- All error returns from function calls are checked — never write `result, _ := someFunc()`

**Write tests:** Create `cmd/gls/main_test.go`.

Table-driven test pattern:

```go
func TestFormatPermissions(t *testing.T) {
    tests := []struct {
        name     string
        mode     fs.FileMode
        expected string
    }{
        {"directory", fs.ModeDir | 0755, "drwxr-xr-x"},
        {"regular file", 0644, "-rw-r--r--"},
        {"executable", 0755, "-rwxr-xr-x"},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := formatPermissions(tt.mode)
            if result != tt.expected {
                t.Errorf("got %q, want %q", result, tt.expected)
            }
        })
    }
}
```

Write tests for at least:
- `formatPermissions` — given a `fs.FileMode`, returns the correct string
- `formatSize` — if you wrote a helper to format sizes
- The hidden file filter — given a list of entries with some dotfiles, verify the correct ones are excluded when `-a` is false

Run your tests:
```bash
go test ./...
go test -v ./...         # verbose output
go test -race ./...      # race detector — run this always
```

---

### Step 9 — Set up the GitHub repository

Create the repo on GitHub and push your code:

```bash
git init
git add .
git commit -m "feat: initial gls implementation"
gh repo create gls --public --source=. --push
```

Add a `.gitignore` at the root:

```
gls           # the compiled binary
dist/         # goreleaser output
*.out         # test coverage output
```

Add a `README.md`:

```markdown
# gls

A reimplementation of the `ls` command in Go.

## Install

Download the latest binary for your OS from the [Releases](../../releases) page.

## Usage

    gls [flags] [path]

    Flags:
      -l    Long format (permissions, size, modified time)
      -a    Show hidden files
      -r    Recursive listing
      -h    Help

## Examples

    gls                  # list current directory
    gls -la /tmp         # long format, all files
    gls -r ~/projects    # recursive listing

## Build from source

    go build ./cmd/gls
```

---

### Step 10 — Set up CI with GitHub Actions

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: "1.22"

      - name: Vet
        run: go vet ./...

      - name: Test
        run: go test -race ./...

      - name: Build
        run: go build ./cmd/gls
```

Push this file. Go to your GitHub repo → Actions tab and watch the CI run. It must be green before you move on.

```bash
git add .github/
git commit -m "chore: add CI workflow"
git push
```

---

### Step 11 — Set up goreleaser

goreleaser builds your binary for Linux, macOS, and Windows in one shot and publishes it as a GitHub Release.

**Install goreleaser:**
```bash
# macOS
brew install goreleaser

# Linux
go install github.com/goreleaser/goreleaser/v2@latest
```

**Create `.goreleaser.yaml` at the project root:**

```yaml
version: 2

before:
  hooks:
    - go mod tidy
    - go test ./...

builds:
  - id: gls
    main: ./cmd/gls
    binary: gls
    env:
      - CGO_ENABLED=0
    goos:
      - linux
      - windows
      - darwin
    goarch:
      - amd64
      - arm64

archives:
  - formats:
      - tar.gz
    name_template: "{{ .ProjectName }}_{{ .Os }}_{{ .Arch }}"
    format_overrides:
      - goos: windows
        formats:
          - zip

checksum:
  name_template: "checksums.txt"

changelog:
  sort: asc
  filters:
    exclude:
      - "^docs:"
      - "^chore:"
      - "^test:"
```

**Test the build locally (no release, just checks it works):**

```bash
goreleaser release --snapshot --clean
```

This creates a `dist/` folder with binaries for every OS. Open `dist/` and look at the files — these are what get uploaded to GitHub Releases.

**Add a GitHub Actions release workflow:** Create `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-go@v5
        with:
          go-version: "1.22"

      - name: Run goreleaser
        uses: goreleaser/goreleaser-action@v6
        with:
          version: latest
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Publish your first release:**

```bash
git tag v1.0.0
git push origin v1.0.0
```

Go to your GitHub repo → Actions. Watch the release workflow run. When it finishes, go to the Releases tab — you should see `v1.0.0` with binary attachments for Linux, macOS, and Windows.

Download the binary for your OS and run it. It should work exactly like running `go run ./cmd/gls`.

---

## Done — Submission Checklist

Go through this before saying you're finished:

- [ ] `gls` with no arguments lists files in the current directory
- [ ] `gls /some/path` lists files in that path
- [ ] `-l` shows permissions, size, and modified time in aligned columns
- [ ] `-a` shows hidden files (dotfiles)
- [ ] `-r` recurses into subdirectories
- [ ] Flags can be combined: `gls -la`, `gls -lar /tmp`
- [ ] Directories have a trailing `/` in output
- [ ] Directories are blue, executables are green, hidden files are dim
- [ ] Errors print to stderr, not stdout
- [ ] Non-existent path exits with code 1 and a clear message
- [ ] `go test -race ./...` passes with no failures
- [ ] Tests exist for at least 3 functions with multiple cases each
- [ ] `.gitignore` excludes the compiled binary and `dist/`
- [ ] CI workflow is green on the `main` branch
- [ ] `v1.0.0` is tagged and published on GitHub Releases
- [ ] Binaries for Linux, macOS, and Windows are attached to the release
- [ ] README explains installation and usage

---

## Things to Look Up (Don't Skip These)

These are Go standard library packages and concepts used in this project. Read the official docs for each one — not a tutorial, the actual docs at [pkg.go.dev](https://pkg.go.dev):

- [`os`](https://pkg.go.dev/os) — `ReadDir`, `Stat`, `Args`, `Stderr`, `Exit`
- [`path/filepath`](https://pkg.go.dev/path/filepath) — `Join`, `Abs`
- [`flag`](https://pkg.go.dev/flag) — `Bool`, `Parse`, `Args`, `Usage`
- [`fmt`](https://pkg.go.dev/fmt) — `Fprintf`, `Fprintln`, width specifiers (`%-10s`, `%8d`)
- [`text/tabwriter`](https://pkg.go.dev/text/tabwriter) — aligning columns
- [`io/fs`](https://pkg.go.dev/io/fs) — `FileInfo`, `FileMode`, `DirEntry`
- [`testing`](https://pkg.go.dev/testing) — `T`, `Run`, `Errorf`, `Fatal`

---

## A Note on Getting Stuck

Getting stuck is part of the process — not a sign that something is wrong.

When you're stuck:
1. Re-read the relevant docs on [pkg.go.dev](https://pkg.go.dev)
2. Write a tiny standalone program that just tests the one thing you don't understand
3. Search the exact error message
4. If still stuck after 30 minutes, reach out — but come with the specific error and what you've already tried
