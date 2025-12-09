# EVColumnizer : Everything-powered values as custom columns

> :warning: **EXPERIMENTAL**
> This script is provided for testing and feedback only. It's not a final release and may contain (serious/funny) bugs.


## What it does

`EVColumnizer` is a Directory Opus add-in that deepens Opus's integration with Everything. It lets you surface any Everything column value inside Opus as if it were a native Opus column.


## Features
* Built with speed in mind.
* Read any value from Everything inside Directory Opus. Performance is best when those values are indexed.
* Configure everything from an easy UI: pick which Everything columns you want to expose in Opus. Enable only the columns you need.
* Fine-grained column customization, including which file types each column applies to. (see note 1)
* Preview data using a sample file/folder.


## Requirements

* Everything **1.5** or newer, with Opus integration already enabled.
* [Everything CLI](https://github.com/voidtools/es) (`es.exe`) **1.1.0.34** or newer.
* Directory Opus 13.19.6 or newer.

Recommended to set `everything_autolaunch` in Opus settings.

`es.exe` can be placed in **that** same path or configured from the script's Settings dialog.

## How it works

The script uses Everything's CLI and tries to minimize the number of calls required to fetch values (see **Current limitations** for details).
To start using it, after install, just click in `Configure` in the Script Management window.

## Usage
Usage is similar (if not equal) to my other script add-in : MediaInfo++.

## Notes

* Columns are divided into three categories: **indexed**, **fast non-indexed**, and **regular**.

  * **Indexed:** columns Everything always keeps indexed (for example: folder file counts, custom columns, native Everything properties).
  * **Fast non-indexed:** columns not indexed by default but that return values very quickly, the difference versus indexed is tiny.
* For speed, indexed and fast columns are fetched in groups: the script requests values for the whole folder in as few calls as possible and asks for all enabled columns of the same category at once. This drastically speeds up retrieval.
* Regular columns are fetched by running `es.exe` per item per column (see **Current limitations** for why).
* An in-memory cache is used to avoid re-requesting values that were just calculated. The cache is volatile and tied to each `Tab`.

> **Note 1:** Indexed columns cannot be restricted by file type because they're retrieved for everything at once rather than filtered per type.
> **Note 2:** To reduce query length, the script requests the data list using `parent:` with the parent folder (valid only for filesystem files). Ideally it would use `filelist:` to target only related files, but `parent:` is used as a workaround because of Everything's query-size limits.



## Current limitations

* ES currently has a bug where long queries (>255 characters, per void's explanation) fail and return no data. This release adds workarounds. void is aware of the issue and we expect a fix in a future release; the script will be updated accordingly.
* For now the script only works on the regular filesystem (no Collections, etc.) because workarounds would otherwise require running `es.exe` per item, which defeats the script's speed goals.
* Some columns visible in Everything's GUI may not return values via `es.exe`. Please report any such cases.

## Technical notes / feedback

* There isn't a reliable programmatic way to tell whether a column is indexed (beyond common ones like size, etc.). Some properties can be indexed only for specific file types or paths. Here, "indexed" means properties Everything maintains without user intervention. If a column is indexed in Everything, its values should be faster to retrieve.
* "Fast" means the value can be retrieved quicker and indicates those columns are fetched in batches rather than individually.
* "Indexed" columns are hardcoded, so you can't choose specific filetypes.
* Folders-only columns can't choose specific filetypes.
* The in-memory cache is a workaround to allow batch retrievals and avoid recalculating values (for example, after expanding a column). A disk cache was avoided to keep things simpler. The cache is cleared when the tab is refreshed (unless you hold <kbd>Ctrl</kbd>+<kbd>Shift</kbd> during refresh), when you change folder, or when the tab is closed. Opinions on whether the memory trade-off is worth the speed are welcome.
* Everything returns raw values, so some column types may not make sense here (I'm considering dropping date/time columns). Some values come back in hexadecimal; a few require hardcoded conversions to be human-friendly (attributes, etc.). Ultimately the user chooses the data type for each column.
* You can preserve the cache across a tab refresh by holding <kbd>Ctrl</kbd>+<kbd>Shift</kbd> while refreshing. The cache is always cleared when changing folders or closing the tab.



## About handling "regular" columns

I haven't settled on the best approach for regular columns. Options I'm weighing:

1. **One call for multiple files per column (current approach).**

   * Faster overall: the initial call does most of the work and each subsequent item takes ~2–3 ms. Files are pre-filtered and non-related files are skipped.
   * Drawback: data becomes available only after the batch completes, which can be bad for very large sets.

2. **One call per file for multiple columns.**

   * Useful if the user enables many regular columns, but still suffers from the ES call delay (albeit reduced). Slower than the first option.

3. **Multiple files with an item limit (e.g. 50).**

   * This was the plan for ALL the columns, but ES 1.1.0.34's query-size limit breaks it. I'll revisit this in future releases.



## If you want to help

If you're interested, try the script and report edge cases (columns that return nothing when it should, weird raw values, or performance issues) or any suggestion you might have. Feedback will guide the next improvements.



*Any suggestions or test results are appreciated.* :wink:

(*Main icon from [All SVG Icons](https://allsvgicons.com)*)
