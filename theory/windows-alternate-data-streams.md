# Windows Alternate Data Streams (ADS)

NTFS lets any file carry extra, hidden "streams" attached to it in addition to
its normal (default) data stream. Neither `dir` nor File Explorer show them by
default, which makes ADS a simple way to hide a payload inside an otherwise
ordinary-looking file.

All paths/filenames are placeholders.

## Concept

```
file.txt              → default data stream (what you normally see/edit)
file.txt:hidden.txt    → an additional named stream on the same file
```

Writing to `file.txt:hidden.txt` does not change the visible size or content
of `file.txt`, and it does not show up in a normal `dir` listing.

## Hiding and running a payload

```cmd
:: write into a hidden stream
notepad file.txt:secret.txt

:: hide an executable inside a normal-looking log file
type payload.exe > windowslog.txt:hidden.exe

:: cmd/PowerShell won't "run" a stream by its stream-qualified name directly,
:: so create a symlink with an .exe extension pointing at the stream, then run *that*
mklink update.exe C:\Temp\windowslog.txt:hidden.exe
update.exe
```

## Detecting ADS (defensive side)

```cmd
dir /r file.txt
:: → file.txt:secret.txt:$DATA   (reveals the hidden stream)
```

`Get-Item -Path file.txt -Stream *` in PowerShell does the same thing.

## Notes

- ADS only exists on **NTFS** — not FAT32/exFAT.
- The `mklink` step exists because you can't directly execute
  `file.txt:stream` from cmd — you point a normal-looking `.exe` symlink at it
  instead.
- Some older AV/EDR engines don't scan alternate streams, which is why this
  shows up as an evasion/persistence trick — modern EDR generally does inspect
  them, so treat this as a technique to recognize, not a reliable bypass.
- `dir /r` (or the PowerShell `-Stream` cmdlet) is the standard way to spot
  this defensively.
