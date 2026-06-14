# Virtual CUPS printer for manual duplexing on Linux.
Backend and PPD for a virtual printer that helps with duplexing.

## Disclaimer
This has only been tested on the Canon LBP 6030, though it should work on any **face-down output printers** (common in laser printers).

Due to the use of PPDs, this method will NOT work on `cups-filters` 3.0 and above.

## What does it do?
- Present itself as simplex/duplex capable for the clients.
- Automatically deal with odd/even pages.
- Pause printing for reloading the stack.
- Flexible resume printing trigger (through file deletion).
- Can be opened to LAN for other devices to use.

## Prerequisites 
- Have `cups pdftk gs` installed.
- Have a real printer set up in CUPS.

## How to install?
1. Download `manualduplex`.
2. Edit the line with `REAL_PRINTER=` and set it to your real printer's CUPS name.
3. Move the file to `/usr/lib/cups/backend/`
4. Grant the required permission and restart CUPS.
```
> sudo chown root:root /usr/lib/cups/backend/manualduplex
> sudo chmod 700 /usr/lib/cups/backend/manualduplex
> sudo systemctl restart cups
```
5. Download the `Printer_Server.ppd` and put it wherever you want (E.g. `~/Printer_Server.ppd`)
6. Install the virtual printer:
```
> sudo lpadmin -p Printer_Server -E -v manualduplex:// -P ~/Printer_Server.ppd
```
⚠️ **WARNING**:
- Change the `~/Printer_Server.ppd` to where you downloaded the file.
- Exclude `-E` if you don't want the virtual printer to be opened to the LAN.

## How to use?
If you're installing this on your local device, the printer should appear as `Printer_Server` with both simplex and duplex capability. When doing simplex, it would print as normal. 

![Fig 1. How to flip the stack correctly](/resources/paper-rearrangement.png)

When doing duplex (long edge), the printer would first the odd pages first before pausing. The user would then flip the stack (Fig 1) and delete the `/tmp/duplex_resume_XXX.lock` file (`XXX` is a placeholder for the job ID) to continue the print job.

## Tips and tricks
Because triggering the second print job is as easy as deleting a tempfile, you can setup buttons, scripts, Discord webhooks, QR codes,... to run `sudo rm -f /tmp/duplex_resume_*.lock` and resume printing.

For instance, in my setup, my old laptop act as a print server sitting right next to the printer itself so I bind `LEFTCTRL` to it through `triggerhappy`.

## Debug
By default, the logs will be stored at `/tmp/printer_server.logs`. If any issue occured, submit an Issue with the relevant log.
