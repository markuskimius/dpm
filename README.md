# Distributed Package Manager (DPM)

## What does it do?

- Use git server(s) as your distributed package repository.
- Install a package by the project's URL, with an optional branch/tag.
- Auto-install dependencies if the package includes a DPM dependency file.
- Auto-remove package(s) installed to satisfy a dependency when the dependency is removed.


## Installation

- Update `PATH` to include DPM's `bin` directory.
- Source the output of `dpm sourceme`.  The output is in BASH format.

```bash
PATH+=:/path/to/dpm/bin
[[ -z "$DPM" ]] && source <(dpm sourceme)
```


## Example

TODO


## License

[GPLv2]


[GPLv2]: <https://github.com/markuskimius/dpm/blob/main/LICENSE>

