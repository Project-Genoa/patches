# Patches for Minecraft Earth

This repository contains patches for Minecraft Earth to change various API/service addresses/URLs and to allow the game to continue functioning.

## Format notes

These patches are intended to be human-readable for documentation purposes and will not work directly in a patcher script without modification or adaptation.

In some cases, "variables" are used to show where configurable values should be substituted into a patch (e.g. when patching a URL where the replacement URL shall be decided based on the needs of the patcher). Variable names are included in the patch file as `${variablename}`. The variables used in each patch are detailed in the patch listing. The "default" values listed are ones that will give the same or most similar behavior to the app without the patch applied.

Due to the lack of a standardised human-readable binary patch format, binary patches are expressed in hexdump format (as produced by the command `hexdump -C -v` on Linux) against the original filename with `.hexdump` appended. For example, a patch against `lib/arm64-v8a/libgenoa.so` is instead expressed as a patch against `lib/arm64-v8a/libgenoa.so.hexdump`. This is done to produce a human-readable patch file that documents the actual changes to be made, however any patcher script will need to reimplement the application of the changes in an appropriate way (including the substitution of variables into embedded strings - see below).

For text files, the patch files may be applied using `patch -p1` as standard after performing variable substitution (if applicable).

### Regarding binary patches, strings, and variables

Some binary patches against `libgenoa.so` add new strings into an area at the end of the file (this is explained in more detail in the patch listing for `extend-libgenoa`). All of these strings follow a format where the first 4 bytes at the address of the string form a little-endian value giving the length of the string, followed by the character data itself. The character data shall be null-terminated. The length value does not include either the null terminator or the length field itself, only the actual characters.

When substituting variables into such strings, it is necessary to update both the character data and the length field. The data included in the human-readable hexdump patch shows the "template" string to be used for variable substitution (including the length field/value appropriate for said string).

The patch listing includes a list of all addresses where template strings that require variable substitution appear in a given patch.

## Patch listing

* `allow-http`

  Allows HTTP connections/URLs to be used in some parts of the app that don't allow them otherwise. This is a prerequisite for most patches that change an API/service address in the event that the replacement address uses HTTP in place of HTTPS.

* `change-locator-address`

  Changes the Minecraft Earth API locator address.

  Variables:
  - `locatorprotocol` The protocol to use for the locator server (`http` or `https`). Default value: `https`
  - `locatorhostname` The domain or IP address to use for the locator server. This may include a port number if necessary. Default value: `locator.mceserv.net`

  Template strings:
  - `lib/arm64-v8a/libgenoa.so` - `0x07000300`

  Prerequisites:
  - `extend-libgenoa`

* `change-msa-login-address`

  Changes the MSA login service address.

  Variables:
  - `liveprotocol` The protocol to use for the MSA login service (`http` or `https`). Default value: `https`
  - `livehostname` The base domain to use for the MSA login service. This may include a port number if necessary. Requests will be made to multiple subdomains of the given domain. Default value: `live.com`

  Prerequisites:
  - `disable-msa-login-signature-validation`
  - `allow-http` if `liveprotocol` is `http`

* `change-playfab-address`

  Changes the PlayFab API address.

  Variables:
  - `playfabprotocol` The protocol to use for the PlayFab API (`http` or `https`). Default value: `https`
  - `playfabhostname` The base domain to use for the PlayFab API. This may include a port number if necessary. Requests will be made to multiple subdomains of the given domain. Default value: `playfabapi.com`

  Template strings:
  - `lib/arm64-v8a/libgenoa.so` - `0x07000100`
  - `lib/arm64-v8a/libgenoa.so` - `0x07000380`

  Prerequisites:
  - `extend-libgenoa`
  - `allow-http` if `playfabprotocol` is `http`

* `change-xboxab-address`

  Changes the address for `xboxab.com`.

  Variables:
  - `xboxabprotocol` The protocol to use for `xboxab.com` (`http` or `https`). Default value: `https`
  - `xboxabhostname` The domain to use in place of `xboxab.com`. This may include a port number if necessary. Default value: `xboxab.com`

  Template strings:
  - `lib/arm64-v8a/libgenoa.so` - `0x07000280`

  Prerequisites:
  - `extend-libgenoa`
  - `allow-http` if `xboxabprotocol` is `http` (unverified)

* `change-xboxlive-address-base`

  Changes the address for most Xbox Live API requests.

  Variables:
  - `xboxliveprotocol` The protocol to use for the Xbox Live API (`http` or `https`). Default value: `https`
  - `xboxlivehostname` The base domain to use for the Xbox Live API. This may include a port number if necessary. Requests will be made to multiple subdomains of the given domain. Default value: `xboxlive.com`

  Template strings:
  - `lib/arm64-v8a/libgenoa.so` - `0x07000000`
  - `lib/arm64-v8a/libgenoa.so` - `0x07000080`

  Prerequisites:
  - `extend-libgenoa`
  - `allow-http` if `xboxliveprotocol` is `http` (unverified)

* `change-xboxlive-address-extra`

  Changes the address for some additional Xbox Live API requests that directly use a full hardcoded request URL rather than being built from the base domain at runtime. This is included as a separate patch for clarity.

  Variables:
  - `xboxliveprotocol` The protocol to use for the Xbox Live API (`http` or `https`). Default value: `https`
  - `xboxlivehostname` The base domain to use for the Xbox Live API. This may include a port number if necessary. Requests will be made to multiple subdomains of the given domain. Default value: `xboxlive.com`

  Template strings:
  - `lib/arm64-v8a/libgenoa.so` - `0x07000180`
  - `lib/arm64-v8a/libgenoa.so` - `0x07000200`

  Prerequisites:
  - `extend-libgenoa`
  - `allow-http` if `xboxliveprotocol` is `http`

* `disable-license-check`

  Disables the Google Play license check, allowing the app to function after being removed from the Google Play store. Note: This is not the same patch as the one produced by the Project Earth team as that patch does not work when offline.

* `disable-msa-login-signature-validation`

  Disables signature validation of MSA login API responses. This is required when using a replacement MSA login service.

* `disable-sunset-time-check`

  Disables the Minecraft Earth sunset date/time check, which prevents the game from being playable if the device time is set later than June 30th 2021. Note: This is not exactly the same patch as the one produced by the Project Earth team as that patch inverts the date check condition whereas this one removes it entirely, although in most cases this produces no observable difference.

* `disable-telemetry`

  Disables multiple telemetry/analytics reporters. The following telemetry components are disabled: AppsFlyer, Braze, App Center, Microsoft Vortex. Note that some analytics data is still reported directly to Xbox Live and PlayFab unless these are replaced as well, otherwise this list appears to be exhausitive.

* `extend-libgenoa`

  Extends the `libgenoa.so` shared library file to allow new data to be added at the end of the file, avoiding the length limitations of in-place patching of the original data. This is a prerequisite for most of the `libgenoa.so` patches. Note: The large amount of zeros is necessary as this memory area is used at runtime for static/global variables. The new data area starts at `0x07000000` (beyond the end of the runtime static data area) and is marked with `0xFF`.

* `fix-official-msa-login-after-signature-change`

  Allows MSA login against the official/default MSA login service after changing the app name or signature (which happens during the process of recompiling a patched version of the app). This patch hardcodes the original signature to use when signing requests to the MSA login service and is required unless you are using a replacement MSA login service that does not validate the client signature. Note: This is not exactly the same patch as the one produced by the Project Earth team as that patch contains smali code which is technically invalid, although there is no functional difference.
