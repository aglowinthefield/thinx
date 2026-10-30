# Fork development

- Fork: https://github.com/aglowinthefield/thinx
- Upstream: https://github.com/pkwagner/thinx
- Branch: `master`
- Local checkout: `C:\Users\tom\code\thinx`
- Windows executable: `C:\Users\tom\go\bin\thinx.exe`
- Go module: `github.com/aglowinthefield/thinx`

## Build, test, and install

Close any running Thinx instance before replacing its Windows executable.
From the checkout:

```sh
go test ./...
go vet ./...
go install ./cmd/thinx
```

Run `thinx` in Windows Terminal. The Go bin directory must be on PATH.
`go install ./cmd/thinx` installs your current local changes; the README's
`go install github.com/aglowinthefield/thinx/cmd/thinx@latest` installs published
source from the fork instead. No fork release binaries have been published yet.

The config round-trip test checks Unix file permissions only on Unix; Windows
uses ACLs, which that test does not validate.

## Git workflow

`origin` is the fork; `upstream` is the original project. GitHub CLI's default
repository is the fork. Use feature branches for roadmap work:

```sh
git switch -c feature/search-filter
# implement and test
git push -u origin HEAD
```

To review upstream updates without changing the working branch:

```sh
git fetch upstream
git log --oneline HEAD..upstream/master
```

Merge or rebase upstream deliberately after reviewing those changes.

## Safety and credentials

Thinx uses unofficial Things Cloud access. Back up your Things database from
Things on your Mac before testing cloud-writing changes. Unit tests do not
replace a manual sync verification on disposable tasks.

Upstream stores the Things Cloud username and password in **plaintext JSON** at
`os.UserConfigDir()/thinx/config.json` (normally `%APPDATA%\thinx\config.json`
on Windows). The synced database is separate, normally at
`%LOCALAPPDATA%\thinx\things.db`. Neither should be committed or shared. Windows
file privacy depends on filesystem ACLs; Unix permission bits do not secure it.
Moving credentials to an OS credential store is recommended future work.

First-run onboarding authenticates and syncs with the real account. There is no
implemented offline provider or logout command yet. Removing the config does
not remove the separate synced database.

## Roadmap starting points

No roadmap feature is implemented by the fork setup commit. Suggested sequence:

1. **Search/filter:** `internal/tui/tui.go` disables the list's built-in filtering
   and handles keys without forwarding them to the list update method.
   `internal/domain/todo.go` already exposes a title-based `FilterValue`.
   Start with filtering the current list, with input-routing tests.
2. **Evening scheduling:** confirm the pinned Things Cloud SDK's representation,
   then update the domain, adapter read/write mapping, equality checks, and modal.
3. **Project/checklist editing:** preserve stable IDs in the domain before
   adding editing; the current display-oriented model does not retain them.
4. **Reordering:** confirm persistent SDK support and add a repository operation
   before implementing row-move bindings. Local-only movement will not survive sync.
5. **Trash/restore:** deletion already calls `TrashTask`; add a trash query and
   separate restore operation. Logbook/completion is not trash/deletion.

The actual SDK is `github.com/pkwagner/things-cloud-sdk`, pinned in `go.mod`.
Inspect that version's capabilities before designing cloud mutations.
