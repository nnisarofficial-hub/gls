# Week 1 Project — `gls`
### A reimplementation of the `ls` command in Go

---

## What You're Building

`ls` is one of the most-used commands in any terminal. You've typed it hundreds of times. This week you'll build your own version called `gls` in Go — from scratch.

By the end you'll have a working binary that lists files and directories, supports multiple flags, outputs colored text, and is published as a real downloadable release on GitHub that anyone can install.

**This is not a tutorial where you copy-paste and move on.** A starter scaffold is provided — the structure is done for you. Your job is to fill in the logic. If something doesn't work, read the error, think about it, look it up. That process is the actual learning.

---

## What You'll Learn

- How Go projects are structured (`go.mod`, `cmd/`, packages)
- Reading the filesystem with `os` and `path/filepath`
- Parsing command-line flags with the `flag` package
- Formatting output with `fmt` and `text/tabwriter`
- Coloring terminal output with ANSI escape codes
- Writing table-driven tests with the `testing` package
- Setting up GitHub CI and publishing releases with goreleaser

---

## Prerequisites

Run all three commands and make sure they work before you start:

```bash
go version        # must show go1.21 or higher
git --version     # any recent version
gh --version      # GitHub CLI — get it from https://cli.github.com
```

Also needed:
- A GitHub account
- VS Code with the [Go extension](https://marketplace.visualstudio.com/items?itemName=golang.go) installed
- When VS Code asks to install `gopls`, say yes

---

## Final Output

When you're done, `gls` will behave like this:

```bash
$ gls
cmd/   go.mod   go.sum   README.md   .gitignore

$ gls -l
drwxr-xr-x      0   May 29 10:11   cmd/
-rw-r--r--    421   May 29 09:45   go.mod
-rw-r--r--   2103   May 29 09:45   go.sum
-rw-r--r--    874   May 29 10:02   README.md

$ gls -a
cmd/   go.mod   go.sum   README.md   .gitignore   .github/

$ gls -r
cmd/
  gls/
    main.go
    main_test.go
go.mod
go.sum
```

---

## Flags Reference

| Flag | What it does |
|---|---|
| `-l` | Long format: permissions, size, modified time |
| `-a` | Show hidden files (names starting with `.`) |
| `-r` | Recursive: list subdirectories too |

Flags can be combined: `gls -la`, `gls -lar /tmp`

---

## Part 1 — Project Setup

### Step 1 — Create the project

> **Concept: Go modules**
>
> Every Go project starts with `go mod init`. This creates a `go.mod` file that declares your module name and Go version — like `package.json` in Node. The module name is usually your GitHub path so others can import your code.
>
> ```bash
> go mod init github.com/your-username/gls
> ```
> This creates:
> ```
> go.mod   ← tracks your module name and dependencies
> ```

Create the project:

```bash
mkdir gls
cd gls
go mod init github.com/<your-username>/gls
```

Now create the folder structure:

```bash
mkdir -p cmd/gls
touch cmd/gls/main.go
touch cmd/gls/main_test.go
```

Your project should look like this:

```
gls/
├── cmd/
│   └── gls/
│       ├── main.go
│       └── main_test.go
└── go.mod
```

> **Why `cmd/gls/main.go` and not just `main.go`?**
> This is Go's standard project layout. When a project grows, it might have multiple binaries (`cmd/server`, `cmd/worker`, etc.). Starting with this structure costs nothing now and avoids a painful refactor later.

---

### Step 2 — Copy the starter scaffold

Copy this entire block into `cmd/gls/main.go`. Do not change anything yet — just read through it and make sure you understand what each function is supposed to do.

```go
package main

import (
	"flag"
	"fmt"
	"os"
	"text/tabwriter"
)

// flags holds the parsed command-line options.
type flags struct {
	long      bool // -l: show permissions, size, modified time
	all       bool // -a: show hidden files (dotfiles)
	recursive bool // -r: recurse into subdirectories
}

func main() {
	f := parseFlags()

	// After flag.Parse(), non-flag arguments are in flag.Args().
	// If no path was given, default to "." (current directory).
	path := "."
	if len(flag.Args()) > 0 {
		path = flag.Args()[0]
	}

	if err := run(path, f, 0); err != nil {
		fmt.Fprintln(os.Stderr, "gls:", err)
		os.Exit(1)
	}
}

// parseFlags registers and parses all command-line flags.
// Returns a flags struct with the parsed values.
func parseFlags() flags {
	// TODO Step 3: define -l, -a, -r boolean flags using flag.Bool
	// TODO Step 3: call flag.Parse()
	// TODO Step 3: return the flags struct with the parsed values
	return flags{}
}

// run lists the contents of path according to the given flags.
// indent tracks the nesting level for recursive listing — always call with 0 initially.
func run(path string, f flags, indent int) error {
	// TODO Step 4: read directory entries using os.ReadDir(path)
	// TODO Step 4: if there's an error (path doesn't exist, no permission), return it
	// TODO Step 5: loop over entries; skip hidden files unless f.all is true
	// TODO Step 5: call printEntry for each entry you're keeping
	// TODO Step 8: if f.recursive is true and the entry is a directory, call run recursively
	return nil
}

// printEntry prints one file or directory entry.
// indent is used for recursive listing — multiply by 2 spaces for indentation.
func printEntry(entry os.DirEntry, f flags, indent int) error {
	info, err := entry.Info()
	if err != nil {
		return err
	}

	name := entry.Name()
	if entry.IsDir() {
		name = name + "/"
	}

	// TODO Step 7: wrap name with colorize(name, entry)

	indentStr := ""
	for i := 0; i < indent; i++ {
		indentStr += "  " // 2 spaces per level
	}

	if f.long {
		// TODO Step 6: use a tabwriter to print long format
		// For now, just print the name so it compiles
		w := tabwriter.NewWriter(os.Stdout, 0, 0, 3, ' ', 0)
		fmt.Fprintf(w, "%s%s\n", indentStr, name)
		w.Flush()
	} else {
		fmt.Printf("%s%s   ", indentStr, name)
	}

	_ = info // remove this line once you use info in Step 6
	return nil
}

// formatLong returns the long-format string for one entry.
// Output example: "-rw-r--r--\t1823\tJan 14 22:11\tnotes.txt"
// Note: columns are separated by \t so tabwriter can align them.
func formatLong(info os.FileInfo, name string) string {
	// TODO Step 6: format permissions using info.Mode().String()
	// TODO Step 6: format size using info.Size()
	// TODO Step 6: format time using info.ModTime().Format("Jan 02 15:04")
	// TODO Step 6: join with \t between each column, return the result
	return name
}

// isHidden returns true if the filename starts with "."
func isHidden(name string) bool {
	// TODO Step 5: implement this — it's one line using strings.HasPrefix
	return false
}

// colorize wraps a name in ANSI color codes based on the file type.
// It returns the name unchanged if the entry type doesn't match any color rule.
func colorize(name string, entry os.DirEntry) string {
	// TODO Step 7: return colorBlue + name + colorReset for directories
	// TODO Step 7: return colorGreen + name + colorReset for executables
	//              (hint: use entry.Info() and check info.Mode()&0111 != 0)
	// TODO Step 7: return colorDim + name + colorReset for hidden files (isHidden)
	// TODO Step 7: return name unchanged for everything else
	return name
}

// ANSI escape codes for terminal colors.
// Usage: fmt.Printf("%s%s%s", colorBlue, text, colorReset)
const (
	colorReset = "\033[0m"
	colorBlue  = "\033[34m" // directories
	colorGreen = "\033[32m" // executables
	colorDim   = "\033[2m"  // hidden files
)
```

Now verify it compiles with no errors:

```bash
go build ./cmd/gls
```

You should see no output — that means it compiled. Running it will do nothing yet (all functions return early), but it must compile cleanly before you proceed.

> **Common mistake:** `imported and not used` error. Go does not allow unused imports. If you add an import you're not using yet, Go will refuse to compile. Remove it or comment it out until you need it.

---

### ✅ Checkpoint 1

Run this and make sure it works:

```bash
go build ./cmd/gls && echo "✓ compiles"
```

Expected output:
```
✓ compiles
```

If you see errors, fix them before moving on. Do not proceed with a broken build.

---

## Part 2 — Core Features

### Step 3 — Parse flags

> **Concept: the `flag` package**
>
> Go's built-in `flag` package handles command-line arguments. Here's the pattern:
>
> ```go
> verbose := flag.Bool("verbose", false, "enable verbose output")
> count   := flag.Int("count", 10, "number of results")
> flag.Parse() // must call this before reading any flag values
>
> if *verbose {  // flag.Bool returns a *bool, so you dereference it
>     fmt.Println("verbose mode on")
> }
> ```
>
> After `flag.Parse()`, any remaining non-flag arguments (like a file path) are in `flag.Args()`.
>
> ```bash
> ./myprogram -verbose -count 5 /tmp
> # flag.Args() == ["/tmp"]
> ```

**Your task:** Fill in `parseFlags()`.

1. Declare three boolean flags: `long` (`-l`), `all` (`-a`), `recursive` (`-r`). Give each a sensible description string.
2. Call `flag.Parse()`.
3. Return a `flags` struct with the values.

Run it and verify the flags are being read:

```bash
go run ./cmd/gls -l
go run ./cmd/gls -a /tmp
go run ./cmd/gls -l -a -r
go run ./cmd/gls -h   # should print usage automatically
```

The program still does nothing visible — that's fine. The important thing is it doesn't crash and `-h` prints your flag descriptions.

> **Common mistake:** Returning the struct with hardcoded `false` values and forgetting to use the `*bool` dereference. `flag.Bool` returns a pointer. You must dereference it with `*` when putting it in the struct: `flags{ long: *long }`.

---

### Step 4 — List files

> **Concept: `os.ReadDir`**
>
> `os.ReadDir` reads a directory and returns its contents as `[]os.DirEntry`.
>
> ```go
> entries, err := os.ReadDir("/tmp")
> if err != nil {
>     // handle error — maybe path doesn't exist, or no permission
> }
> for _, entry := range entries {
>     fmt.Println(entry.Name())  // just the filename, not full path
>     fmt.Println(entry.IsDir()) // true if it's a directory
> }
> ```
>
> `os.ReadDir` returns entries sorted alphabetically. You don't need to sort them yourself.

**Your task:** Fill in the `run()` function.

1. Call `os.ReadDir(path)`. If it returns an error, return the error immediately.
2. Loop over the entries.
3. For each entry, call `printEntry(entry, f, indent)`.

Don't handle the `-a` or `-r` flags yet. Just get all files printing.

```bash
go run ./cmd/gls
```

Expected output (filenames on one line, separated by spaces):

```
cmd/   go.mod
```

> **Common mistake:** Not returning the error from `os.ReadDir`. Never write `entries, _ := os.ReadDir(path)` — the `_` silently swallows errors. Always handle them.

---

### ✅ Checkpoint 2

```bash
go run ./cmd/gls
```

You should see the files in your project directory printed on one line. Then:

```bash
go run ./cmd/gls /tmp
```

You should see the contents of `/tmp`. If you get `gls: /tmp: no such file or directory`, your `/tmp` path might differ — try `/var/folders` or any path you know exists.

```bash
go run ./cmd/gls /this/does/not/exist
```

Expected:
```
gls: /this/does/not/exist: no such file or directory
```

---

### Step 5 — Implement `-a` (hidden files)

> **Concept: `strings.HasPrefix`**
>
> ```go
> import "strings"
>
> strings.HasPrefix(".gitignore", ".")  // true
> strings.HasPrefix("main.go", ".")     // false
> ```
>
> Hidden files in Unix are simply files whose names start with `.`.
> The special entries `.` (current dir) and `..` (parent dir) also start with `.` and should always be excluded.

**Your task:**

1. Fill in `isHidden(name string) bool`. One line: return true if the name starts with `"."`.
2. In `run()`, before calling `printEntry`, check: if `!f.all` and `isHidden(entry.Name())`, skip the entry with `continue`.

```bash
go run ./cmd/gls      # should NOT show .gitignore
go run ./cmd/gls -a   # SHOULD show .gitignore
```

Add `"strings"` to your import block at the top.

---

### Step 6 — Implement `-l` (long format)

> **Concept: `os.FileInfo` and `text/tabwriter`**
>
> `entry.Info()` returns `os.FileInfo` which has:
> ```go
> info.Mode().String()    // "-rw-r--r--" or "drwxr-xr-x"
> info.Size()             // int64, size in bytes
> info.ModTime()          // time.Time
> info.ModTime().Format("Jan 02 15:04")  // "May 29 10:11"
> ```
>
> `tabwriter` automatically aligns columns. Separate columns with `\t` and it handles the spacing:
>
> ```go
> w := tabwriter.NewWriter(os.Stdout, 0, 0, 3, ' ', 0)
> fmt.Fprintln(w, "Alice\t30\tEngineer")
> fmt.Fprintln(w, "Bob\t25\tDesigner")
> w.Flush() // must call Flush or nothing prints
> ```
> Output:
> ```
> Alice   30   Engineer
> Bob     25   Designer
> ```

**Your task:**

1. Fill in `formatLong(info os.FileInfo, name string) string`.
   - Get permissions: `info.Mode().String()`
   - Get size: `info.Size()` (it's an `int64`)
   - Get time: `info.ModTime().Format("Jan 02 15:04")`
   - Join them with `\t` between each: `permissions + "\t" + size + "\t" + time + "\t" + name`
   - Use `fmt.Sprintf` to format the size as a number: `fmt.Sprintf("%d", info.Size())`

2. In `printEntry()`, when `f.long` is true:
   - Call `formatLong(info, name)` to get the formatted string
   - Print it using `tabwriter` (a new writer per entry is fine for now)

> **Common mistake:** Forgetting `w.Flush()`. If you don't call it, `tabwriter` buffers everything and nothing prints.

```bash
go run ./cmd/gls -l
```

Expected output:
```
drwxr-xr-x   0     May 29 10:11   cmd/
-rw-r--r--   421   May 29 09:45   go.mod
```

---

### ✅ Checkpoint 3

Run these one by one and verify each:

```bash
# Default listing — files on one line
go run ./cmd/gls

# Long format
go run ./cmd/gls -l

# Show hidden files
go run ./cmd/gls -a

# Both together
go run ./cmd/gls -la
```

For `-la`, you should see hidden files in long format. If any combination crashes or shows wrong output, fix it before Step 7.

---

### Step 7 — Add color

> **Concept: ANSI escape codes**
>
> ANSI codes are special sequences your terminal interprets as formatting commands. They look like `\033[34m` where `34` is the color number.
>
> The pattern is always: **color code → text → reset code**
>
> ```go
> // This prints "hello" in blue, then resets
> fmt.Printf("\033[34m%s\033[0m\n", "hello")
>
> // Using the constants already in your scaffold:
> fmt.Printf("%shello%s\n", colorBlue, colorReset)
> ```
>
> If you forget the reset code, everything after it will also be colored.

The scaffold already has the color constants. **Your task:** fill in `colorize()`.

Rules:
- Directory → `colorBlue`
- Executable file → `colorGreen` — check with `entry.Info()` then `info.Mode()&0111 != 0`
- Hidden file → `colorDim` — use your `isHidden()` function
- Everything else → return `name` unchanged (no color)

Then in `printEntry()`, wrap the name with `colorize(name, entry)` before printing it.

```bash
go run ./cmd/gls
```

Directories should appear in blue. If you have any executable files, they should appear in green.

> **Common mistake:** Applying color but forgetting `colorReset` at the end. Your prompt will turn blue after gls exits. Always end with `colorReset`.

> **Note:** The `-l` flag long format should also use color on the filename column. Make sure `formatLong` receives the already-colorized name.

---

### Step 8 — Implement `-r` (recursive)

> **Concept: recursive functions**
>
> A recursive function calls itself. The key is having a clear stopping condition (the base case) so it doesn't loop forever.
>
> ```go
> func countDown(n int) {
>     if n <= 0 {
>         return // base case — stop here
>     }
>     fmt.Println(n)
>     countDown(n - 1) // recursive call
> }
> ```
>
> For directory listing, the base case is "this entry is not a directory — don't recurse."

**Your task:** In `run()`, after printing each entry, add:

```go
if f.recursive && entry.IsDir() {
    subPath := // TODO: build the full path to the subdirectory
               // hint: use path/filepath.Join(path, entry.Name())
    fmt.Println() // blank line before each subdirectory
    if err := run(subPath, f, indent+1); err != nil {
        fmt.Fprintln(os.Stderr, "gls:", err)
        // don't return — continue with other entries
    }
}
```

Add `"path/filepath"` to your imports.

```bash
go run ./cmd/gls -r
```

Expected output:
```
cmd/   go.mod   go.sum

  main.go   main_test.go
```

> **Common mistake:** Using `entry.Name()` alone as the path for the recursive call. `entry.Name()` is just the filename — not the full path. Use `filepath.Join(path, entry.Name())` to build the correct full path.

---

### ✅ Checkpoint 4

```bash
# All flags working
go run ./cmd/gls -l
go run ./cmd/gls -a
go run ./cmd/gls -la
go run ./cmd/gls -r
go run ./cmd/gls -lar

# Error handling
go run ./cmd/gls /does/not/exist
# Expected: gls: /does/not/exist: no such file or directory

# Specific path
go run ./cmd/gls /tmp
```

Everything above should work without crashing. Color should be visible. Long format should have aligned columns.

---

## Part 3 — Tests

### Step 9 — Write tests

> **Concept: table-driven tests**
>
> Instead of writing a separate test function for each case, Go encourages putting all cases for one function in a table (a slice of structs):
>
> ```go
> func TestDouble(t *testing.T) {
>     tests := []struct {
>         name  string
>         input int
>         want  int
>     }{
>         {"positive", 5, 10},
>         {"zero", 0, 0},
>         {"negative", -3, -6},
>     }
>
>     for _, tt := range tests {
>         t.Run(tt.name, func(t *testing.T) {
>             got := double(tt.input)
>             if got != tt.want {
>                 t.Errorf("double(%d) = %d, want %d", tt.input, got, tt.want)
>             }
>         })
>     }
> }
> ```
>
> Run a single case: `go test -run TestDouble/zero`

Open `cmd/gls/main_test.go` and write tests for these three functions:

**Test 1: `isHidden`**

```go
func TestIsHidden(t *testing.T) {
    tests := []struct {
        name   string
        input  string
        want   bool
    }{
        // TODO: add at least 5 cases
        // Hint: ".gitignore" → true, "main.go" → false,
        //       ".git" → true, "README.md" → false, "." → true
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := isHidden(tt.input)
            if got != tt.want {
                t.Errorf("isHidden(%q) = %v, want %v", tt.input, got, tt.want)
            }
        })
    }
}
```

**Test 2: `formatLong`**

You need a fake `os.FileInfo` to test this. Go's `os` package doesn't expose a struct you can create directly, but `os.Stat` gives you one:

```go
func TestFormatLong(t *testing.T) {
    // Create a real temp file so we can get a real FileInfo
    tmpFile, err := os.CreateTemp("", "gls-test-*.txt")
    if err != nil {
        t.Fatal("could not create temp file:", err)
    }
    defer os.Remove(tmpFile.Name())
    tmpFile.Close()

    info, err := os.Stat(tmpFile.Name())
    if err != nil {
        t.Fatal("could not stat temp file:", err)
    }

    result := formatLong(info, "testfile.txt")

    // Check that the result contains the pieces we expect
    if !strings.Contains(result, "testfile.txt") {
        t.Errorf("formatLong result %q does not contain filename", result)
    }
    if !strings.Contains(result, info.ModTime().Format("Jan 02 15:04")) {
        t.Errorf("formatLong result %q does not contain modified time", result)
    }
}
```

**Test 3: filtering hidden files**

This one tests the filtering logic in `run()` indirectly by checking `isHidden` against a list of names:

```go
func TestHiddenFilesFiltering(t *testing.T) {
    names := []string{"main.go", ".gitignore", "README.md", ".env", "go.mod"}

    var visible []string
    for _, name := range names {
        if !isHidden(name) {
            visible = append(visible, name)
        }
    }

    expected := []string{"main.go", "README.md", "go.mod"}
    if len(visible) != len(expected) {
        t.Errorf("got %d visible files, want %d: %v", len(visible), len(expected), visible)
    }
}
```

Add `"strings"` to the test file's imports.

Run your tests:

```bash
go test ./...            # run all tests
go test -v ./...         # verbose — shows each test case
go test -race ./...      # race detector — always run this
```

All tests must pass before moving on.

---

### ✅ Checkpoint 5

```bash
go test -race -v ./...
```

Expected output (approximately):

```
=== RUN   TestIsHidden
=== RUN   TestIsHidden/.gitignore
=== RUN   TestIsHidden/main.go
...
--- PASS: TestIsHidden (0.00s)
=== RUN   TestFormatLong
--- PASS: TestFormatLong (0.00s)
=== RUN   TestHiddenFilesFiltering
--- PASS: TestHiddenFilesFiltering (0.00s)
PASS
ok  	github.com/<your-username>/gls/cmd/gls	0.123s
```

If you see `FAIL`, read the error message carefully — it tells you exactly which case failed and what the expected vs actual values were.

---

## Part 4 — GitHub, CI, and Releases

### Step 10 — Create the GitHub repository

First add a `.gitignore` in the project root:

```
gls
dist/
*.out
```

Then push to GitHub:

```bash
git init
git add .
git commit -m "feat: initial gls implementation"
gh repo create gls --public --source=. --push
```

Go to `github.com/<your-username>/gls` in your browser and verify the code is there.

---

### Step 11 — Add a README

Create `README.md` in the project root:

```markdown
# gls

A reimplementation of the `ls` command, built in Go.

## Install

Download the latest binary for your OS from the [Releases](../../releases) page.

## Usage

    gls [flags] [path]

**Flags:**

    -l    Long format (permissions, size, modified time)
    -a    Show hidden files
    -r    Recursive listing

## Examples

    gls                  # list current directory
    gls -la /tmp         # long format, show hidden files
    gls -r ~/projects    # recursive listing

## Build from source

    go build ./cmd/gls
```

Commit it:

```bash
git add README.md .gitignore
git commit -m "docs: add README and gitignore"
git push
```

---

### Step 12 — Add CI with GitHub Actions

> **Concept: GitHub Actions**
>
> GitHub Actions runs automated jobs whenever you push code or open a pull request. A job is defined in a YAML file under `.github/workflows/`. Each job has steps — commands that run in order.
>
> The CI job below runs on every push and PR to `main`. It installs Go, runs `go vet` (a built-in linter), runs your tests with the race detector, and builds the binary. If any step fails, the push is marked red on GitHub.

Create the directory and file:

```bash
mkdir -p .github/workflows
touch .github/workflows/ci.yml
```

Paste this into `ci.yml`:

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

Commit and push:

```bash
git add .github/
git commit -m "chore: add CI workflow"
git push
```

Go to your GitHub repo → **Actions** tab. You'll see a workflow running. Wait for it to finish — it should be green. If it's red, click into it to see which step failed and why.

---

### Step 13 — Set up goreleaser

> **Concept: goreleaser**
>
> goreleaser is a tool that cross-compiles your Go binary for multiple operating systems and architectures in one shot, then uploads all the binaries to a GitHub Release. Without it you'd have to run `go build` six times with different `GOOS`/`GOARCH` environment variables and upload each file manually.
>
> goreleaser is triggered by a git tag. When you push `v1.0.0`, a GitHub Actions workflow fires, goreleaser builds everything, and a release appears on GitHub with downloadable binaries.

**Install goreleaser** (run this once):

```bash
# macOS
brew install goreleaser

# Linux / WSL
go install github.com/goreleaser/goreleaser/v2@latest
```

**Create `.goreleaser.yaml`** in the project root:

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

**Test it locally first** (no release, just verifies it builds):

```bash
goreleaser release --snapshot --clean
```

This creates a `dist/` folder. Open it — you should see binary files for Linux, macOS, and Windows. That's what will be uploaded to GitHub.

**Add the release workflow:** Create `.github/workflows/release.yml`:

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

Commit and push everything:

```bash
git add .goreleaser.yaml .github/workflows/release.yml
git commit -m "chore: add goreleaser and release workflow"
git push
```

**Publish your first release:**

```bash
git tag v1.0.0
git push origin v1.0.0
```

Go to your GitHub repo → **Actions** tab. Watch the release workflow run. When it finishes (about 2–3 minutes), go to the **Releases** tab. You should see `v1.0.0` with binary files attached.

Download the binary for your OS, give it execute permission, and run it:

```bash
# macOS/Linux
chmod +x gls_darwin_arm64/gls
./gls_darwin_arm64/gls

# It should work exactly like go run ./cmd/gls
```

---

### ✅ Final Checkpoint

Go through every item. Don't mark it done unless you've actually tested it:

- [ ] `gls` with no arguments lists files in the current directory
- [ ] `gls /some/path` lists files at that path
- [ ] `-l` shows permissions, size, and modified time in aligned columns
- [ ] `-a` shows hidden files (dotfiles)
- [ ] `-r` recurses into subdirectories with indentation
- [ ] Flags combine correctly: `gls -la`, `gls -lar /tmp`
- [ ] Directories have a trailing `/` in output
- [ ] Directories are blue, executables are green, hidden files are dim
- [ ] Error for non-existent path: `gls: /x: no such file or directory`
- [ ] Errors print to stderr (test: `gls /bad/path 2>/dev/null` — should print nothing)
- [ ] `go test -race ./...` passes with no failures
- [ ] Tests cover `isHidden`, `formatLong`, and hidden file filtering
- [ ] `.gitignore` excludes the binary and `dist/`
- [ ] CI is green on the GitHub Actions tab
- [ ] `v1.0.0` release exists on GitHub Releases page
- [ ] Binaries for Linux, macOS, and Windows are attached to the release
- [ ] README explains installation and usage

---

## When You're Stuck

1. **Read the error message fully.** Go's errors are very specific. `undefined: strings` means you forgot to import `"strings"`. `cannot use X as type Y` tells you exactly what type mismatch you have.

2. **Write a tiny test program.** If you don't understand how `tabwriter` works, create a scratch file, import it, and experiment with 5 lines of code. Delete the file when you understand it.

3. **Check [pkg.go.dev](https://pkg.go.dev).** The official Go docs are excellent. Search the package name (e.g., `os`, `flag`, `text/tabwriter`) and read the function signatures and examples.

4. **After 30 minutes on the same problem, ask.** Come with: the specific error message, what you've already tried, and what you think might be wrong.

---

## Packages Used in This Project

| Package | What you use it for |
|---|---|
| [`os`](https://pkg.go.dev/os) | `ReadDir`, `Stat`, `Stderr`, `Exit`, `CreateTemp` |
| [`path/filepath`](https://pkg.go.dev/path/filepath) | `Join` |
| [`flag`](https://pkg.go.dev/flag) | `Bool`, `Parse`, `Args` |
| [`fmt`](https://pkg.go.dev/fmt) | `Printf`, `Fprintf`, `Fprintln`, `Sprintf` |
| [`strings`](https://pkg.go.dev/strings) | `HasPrefix` |
| [`text/tabwriter`](https://pkg.go.dev/text/tabwriter) | Aligned column output |
| [`testing`](https://pkg.go.dev/testing) | `T`, `Run`, `Errorf`, `Fatal` |
