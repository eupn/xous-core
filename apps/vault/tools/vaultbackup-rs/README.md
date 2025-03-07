# `vaultbackup-rs`

A tool to backup/restore `vault` app storage to/from your computer.

## Supported data

 - passwords
 - TOTP tokens

## Dependencies

[`hidapi`](https://github.com/libusb/hidapi) and `protoc` is required for this tool to work.

On Debian-based systems, it can be installed with `apt-get install libhidapi-dev protobuf-compiler`.

On top of that, Linux systems need the following `udev` rules:

```udev
SUBSYSTEM=="usb", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="5bf0", GROUP="plugdev", TAG+="uaccess"
SUBSYSTEM=="usb", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="3613", GROUP="plugdev", TAG+="uaccess"
```

Place them in `/etc/udev/rules.d/99-precursor-usb.rules`, and either reboot your system or reload `udev` rules with:

```
# udevadm control --reload-rules
```

`udev` rules aren't strictly required, they're there to allow you to communicate with Precursor without root privileges.

If you prefer prefixing `sudo` to every `vaultbackup-rs` invocation, you're free to do so.

## Usage

The Precursor device must have "host readout" enabled, by selecting the `vault` context menu at F4
and selecting "Enable host readout".

```
vaultbackup
A backup/restore tool for the Precursor vault app.

USAGE:
    vaultbackup-rs <SUBCOMMAND>

OPTIONS:
    -h, --help    Print help information

SUBCOMMANDS:
    backup     Backup data from device
    format     Format a known password manager export for Vault
    help       Print this message or the help of the given subcommand(s)
    restore    Restore data to device
```

Using the tool is quite simple:

```
vaultbackup-rs <ACTION> <TARGET> <PATH>
```

`ACTION` can either be `backup` or `restore`, while target specifies the data you want to act upon:

| Target  | Meaning  |
|---|---|
|`password`|Act on passwords storage|
|`totp`|Act on TOTP storage|

`PATH` specifies the path where `vaultbackup-rs` reads/writes data.

The output format is JSON.

## Importing other password manager's exports

`vaultbackup-rs` supports importing other password manager's export data in Vault, but to do so, you have to format it to Vault's format first.

The `subcommand` does this for you.

Supported password managers:
 - Bitwarden: TOTP, logins
 - Google Authenticator: TOTP

Example:

```bash
$ vaultbackup-rs format bitwarden your-bitwarden-export.json
```

For Google Authenticator, the input file is expected to contain
`otpauth://` or `otpauth-migration://` URIs, one per line; for example,
to import QR codes from the Google Authenticator app:

```bash
$ zbarcam --raw | tee authenticator-export.txt
$ vaultbackup-rs format authenticator authenticator-export.txt
$ vaultbackup-rs restore totp authenticator_to_vault_totps.json
```

If `zbarcam` can't decode a dense QR code with multiple TOTP exports, try selecting just one TOTP code at a time, and then exporting them in series. The commands will automatically collate all the individually read QR codes.
