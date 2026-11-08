# Improvement suggestions

A small PHP/Apache application for collecting improvement suggestions.
Entries are stored in `data/suggestions.json`; new entries start with the
status `erfasst` (recorded).

## Apache setup

The application requires Apache with PHP 8.1 or later, the PHP `mbstring` and
JSON extensions, and sessions enabled. The Excel export additionally requires
`ZipArchive` and `XMLWriter`. cURL is used for translation; if cURL is
unavailable, the application falls back to PHP HTTP streams. Apache must
process `.htaccess` files with `AllowOverride AuthConfig` (or
`AllowOverride All`) and load `mod_authz_core`.

1. Copy the contents of `sfi_LeapOS` into an Apache web directory. PHP needs
   write permission in `data`, where `data/suggestions.json` is created or
   updated.
2. Create a `.htpasswd` file outside the public web directory. The paths
   configured for local MAMP and Alfahosting are
   `/Applications/MAMP/access/sfi_LeapOS/.htpasswd` and
   `/var/www/vhosts/h331132.web114.alfahosting-server.de/access/sfi_LeapOS/.htpasswd`.
   For other hosts, update the paths in `admin/index.php` and `export/index.php`.
   For example:

   ```text
   htpasswd -c /Applications/MAMP/access/sfi_LeapOS/.htpasswd admin
   ```

3. The admin page selects the password file based on the host: `localhost`
   and `127.0.0.1` use the MAMP path; other hosts use the Alfahosting path.
   The login form asks only for the password, not the username stored in
   `.htpasswd`.
4. Enable `AllowOverride AuthConfig` (or `AllowOverride All`) for this
   directory and load `mod_authz_core`. The `.htaccess` files in `admin` and
   `export` allow access to the PHP pages, which then use the application's
   password check; they do not enable HTTP Basic Authentication.
   `data/.htaccess` blocks direct HTTP access to the JSON file.

The public page is `index.php`; the administration area is at `…/admin`, and
the export page is at `…/export`. The public list can be searched and filtered
by date, status, category, classification, model and LeapOS version. Admins
can edit and delete entries and change their status. Rejecting a suggestion
requires a reason. Editing a newly recorded entry changes its status to
`geprüft` (reviewed). Only reviewed entries appear in the export page, and
successfully exported entries are then marked `versendet` (sent).

## Languages and versions

The interface is available in German (`de`) and English (`en`). The selector
in the upper-right corner changes the language for the current session.
Strings are stored in `languages/de.php` and `languages/en.php`, which use the
same keys. To add a language, add a matching catalogue and include its
ISO-639 code in `SUPPORTED_LANGUAGES` in `lib.php`. The option label is
generated from the language code.

Available LeapOS versions combine `LEAPOS_VERSIONS` in `lib.php` with versions
in existing suggestions. Users can add a numeric, dot-separated version. If
an administrator edits or deletes the last suggestion using a custom version,
that version is no longer offered.

URL parameters such as `username`, `userID` and `userid` are not processed.
Suggestions do not store an author name or author number.

## Export

The password-protected page at `…/export` lists only suggestions with status
`geprüft`. It includes the number, topic, model, LeapOS version, category,
classification and suggestion. Category and classification are shown as
readable text; unknown values are shown as `Not specified`.

The export is available as CSV or an Excel-compatible `.xlsx` file. Exported
suggestions are marked `versendet` after the file has been generated
successfully and no longer appear in the reviewed export list.

## Translation

Free-text translation of topics and suggestions uses LibreTranslate. A
self-hosted instance has no fixed provider character quota; practical limits
depend on server capacity and configuration. A local example is:

```text
pip install libretranslate
libretranslate --host 127.0.0.1 --port 5000 --load-only de,en
```

The application defaults to `https://killerminion.ddns.berlin:443/translate`.
Set `LIBRETRANSLATE_URL` in the Apache environment to use a different complete
translation endpoint, for example `http://127.0.0.1:5000/translate`. For a
protected or managed service, `LIBRETRANSLATE_API_KEY` can also be set. Keep
API keys in server configuration, never in the repository. The configured
endpoint is accessed over HTTPS through an nginx reverse proxy; its
certificate is verified and must be valid.

If the translation service is unavailable, free text is shown unchanged and
the export page displays a notice. See the [LibreTranslate API
guide](https://docs.libretranslate.com/guides/api_usage/) and
[installation guide](https://docs.libretranslate.com/guides/installation/).

## Deployment and data

Include hidden files such as `.htaccess` in deployment packages. The VS Code
deployment configuration explicitly includes `admin/.htaccess` and
`data/.htaccess`. `data/suggestions.json` is intentionally excluded from the
deployment package because it contains live suggestions and must not be
overwritten by code deployments.

For a fresh installation, the JSON file may be absent and the application
shows an empty list. The `data` directory must be writable for the first save.
When deploying an existing installation, preserve its `data/suggestions.json`
separately and do not replace it with an empty file.
