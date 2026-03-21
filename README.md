# copy

Single standalone Python CLI for local filesystem transfers using `rsync` (copy by default, move with `-m/--move`).

## Requirements

- Linux with:
  - `python3`
  - `rsync`
  - coreutils (`cp`, `mv`, `rm`, `find`)
  - `sudo` (only when using `-s/--sudo`)

## Project Structure

```text
copy/
├── .gitignore
├── copy
├── README.md
└── tests/
    ├── test_cli_matrix.py
    └── test_copy_cli.py
```

## Command

```bash
./copy [OPTIONS] SOURCE DESTINATION
```

- Default mode: copy
- Move mode: `-m`, `--move`

## Flags

- `-m`, `--move`
  - Move mode (transfer then remove source data).
- `-s`, `--sudo`
  - Run transfer/removal commands with sudo.
- `-o`, `--overwrite`
  - Replace conflicting destination target instead of merge behavior.
- `-c`, `--contents-only`
  - Merge source contents directly into destination path (no source-basename nesting).
- `-b`, `--backup`
  - Create timestamped backup when destination data would be merged/replaced.
- `-v`, `--verbose`, `--showall`
  - Show hierarchical preview: up to 5 changed entries per level (modified first), expand only modified folders, and abbreviate remaining non-shown counts.

## Current Behavior

- Always runs an `rsync` dry-run preflight first (`--itemize-changes --stats`).
- Uses `--size-only` for transfer decisions.
- For simple operations (no merge/overwrite/backup/contents-only), uses native backend:
  - `cp -a` in copy mode
  - `mv` in move mode
- `SOURCE/*` is treated as contents-only mode (equivalent to using `-c` on `SOURCE/`).
- Parent/self-overlap safety is enforced; no-op cases print `No changes detected`.
- Move mode removes transferred source files and cleans empty source directories.

## Preview Output

- Mode line (`Overwrite Move Copy Merge Rename Backup File Dir Contents`).
- Tree preview rooted at destination context.
- In verbose mode, unchanged overflow is abbreviated (plus new/modified/removed overflow when present).
- Regular file summary:
  - `new`, `modified`, `unchanged`, `removed`, `removed_from_source`
- Planned transfer bytes + confirmation prompt.

## Usage Examples

```bash
# Copy directory into destination root (may nest source basename)
./copy /data/src/project /data/dst/

# Move directory
./copy -m /data/src/project /data/archive/

# Overwrite conflicting target
./copy -m -o /data/src/project /data/archive/

# Contents-only merge (no nesting)
./copy -c /data/src/project/ /data/archive/project_renamed

# Overwrite explicit destination path in contents-only mode
./copy -m -o -c /data/src/project/ /data/archive/project_renamed

# Literal SOURCE/* behavior (same as contents-only)
./copy "/data/src/project/*" /data/archive/

# Sudo mode
./copy -s /root/input /mnt/shared/output/
```

## Test

```bash
python3 -m unittest discover -s tests -v
```
