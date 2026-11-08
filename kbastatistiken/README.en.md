# KBA statistics

This project fetches new vehicle-registration data by model series directly
from the German Federal Motor Transport Authority (KBA) statistics portal and
turns it into an HTML report. The report can be previewed in a browser and
optionally saved as the published version.

The PHP interface is in `index.php`. It retrieves data through
`kba-arcgis.php`, runs `csv2html.py` with temporary files and controls when the
generated report is saved. Both the interface and generated report link to the
[legal notice](https://synvoll.de/kontakt/impressum/).

## Overview

The usual workflow is:

1. Click **Generate statistics** in `index.php`.
2. `kba-arcgis.php` fetches the latest available data from the KBA ArcGIS
   service and writes a temporary CSV file.
3. `csv2html.py` creates the HTML report.
4. The report is shown as a preview in an `iframe`.
5. Only after reviewing the preview and confirming with the password is
   `kba-statistiken.html` updated.

![GitHub workflow](https://github.com/synb10se/lpmff/blob/main/kbastatistiken/doc/GitHub-Worflow.jpg)

## Files

### `index.php`

`index.php` is the interactive PHP interface. It can:

- retrieve current data through `kba-arcgis.php`;
- execute the Python script with `exec()`;
- render the generated HTML as a preview;
- offer a download of the generated, not-yet-saved report;
- display a preview only after successful processing;
- replace the saved report only after the correct password is entered; and
- reset the current session state.

Important details:

- A temporary input and output file are used for each Python run.
- After generation, the interface displays the number of records and the
  latest reporting month.
- Generating a report does not require a password. Anyone can retrieve the
  current available data and download the result using **Download HTML file**.
  Only replacing `kba-statistiken.html` is password-protected.
- The generated HTML is held in the session as `pending_html`.
- The saved file is replaced only after an `action=save` request with a valid
  password. Generating a preview or a failed fetch leaves the saved file
  unchanged.
- `?action=download` is available only while an unsaved result exists in the
  current session.

### `kba-arcgis.php`

`kba-arcgis.php` provides data-retrieval functions used by `index.php`. Instead
of relying on the KBA portal's frequently changing download link, it queries
the underlying ArcGIS FeatureServer at this fixed endpoint:

```text
https://services-eu1.arcgis.com/U09msXRZoxesNntH/arcgis/rest/services/SP_Modellreihen/FeatureServer/0
```

`kba_fetch_csv()`:

- retrieves field names and column labels from the FeatureServer;
- determines the record count and latest reporting month;
- retrieves all records in pages of up to 1,000, with up to six requests in
  parallel;
- writes a CSV with the same headings as the manual export, encoded as UTF-8
  with a byte-order mark; and
- verifies that all records were received.

If the service is unavailable, the data is incomplete or its structure has
changed, an error is shown in `index.php` and the saved
`kba-statistiken.html` is left untouched. A fetch typically takes 10–15
seconds.

### `csv2html.py`

The Python script converts the CSV to the HTML report. It can also be run from
the command line:

```bash
python3 csv2html.py --input data.csv --output kba-statistiken.html --quiet
```

It uses only the Python standard library. Without options, it reads
`SP_Modellreihen.csv`, writes `lm-kba-statistik.html` and includes model series
`B05,B10,C10,T03`. Use `--models B05,B10` to select a comma-separated model
list, `--verbose` for additional processing details, `--quiet` to suppress
status messages or `--dry-run` to validate the input without writing HTML. The
CSV must have the column headings expected from a KBA export.

The report contains the project's relevant tables, including year/month
filters and a browser-friendly layout. In the Leapmotor new-registration
table, `total` column headings also show each model's sum across all reporting
months included in the report. These totals do not change when the year or
month filters are changed.

### `kba-statistiken.html`

`kba-statistiken.html` is the persistently saved report. It is not replaced
every time a preview is generated; it changes only after saving is explicitly
confirmed in `index.php`.

The file can be opened in a browser, published on a web server or linked from a
wiki or forum. It does not require PHP or Python to be viewed.

## Requirements

The web interface requires a PHP web server with the `exec()` function enabled
and the PHP cURL extension. The server must be able to make outbound HTTPS
connections to `services-eu1.arcgis.com`. Python 3 must also be available. The
standalone Python script requires no third-party packages.

The `PYTHON_BIN` constant in `index.php` is set to `python3`:

```php
const PYTHON_BIN = 'python3';
```

If Python is installed elsewhere, change it to the correct path, for example:

```php
const PYTHON_BIN = '/usr/bin/python3';
```

## Password protection for saving

Saving is enabled only when the environment variable
`KBA_SAVE_PASSWORD_HASH` is set. It must contain a PHP password hash, not the
plain-text password.

For example, generate a hash with:

```bash
php -r "echo password_hash('SECRET_PASSWORD', PASSWORD_DEFAULT), PHP_EOL;"
```

Set the resulting hash in the PHP server's environment as
`KBA_SAVE_PASSWORD_HASH`.

Without a valid hash, or when the password is incorrect, the saved report is
not changed. The new preview remains in the session until it is successfully
saved.

## Using the web interface

1. Open `index.php` in a browser.
2. Click **Generate statistics** and wait for the data retrieval to finish.
3. Review the report in the `iframe` preview and check the displayed reporting
   date.
4. Optionally click **Download HTML file** to export the preview.
5. If the preview is correct, enter the password and click **Confirm local
   save**.
6. `kba-statistiken.html` is then updated.

The KBA does not always publish new monthly data at the beginning of the
month, so check the reporting date before saving.

Use **Reset** to discard the current session state and return to the saved
report.

## Report contents

The generated page typically contains several tabs with KBA analyses, such as:

- Leapmotor new registrations;
- comparisons of electric and plug-in hybrid vehicles; and
- comparisons of combustion-engine vehicles and electric mobility.

Year and month filters apply across all tabs. **Show all** resets the filters.
The layout is optimized for browsers: headings and column labels remain visible
in long tables, and tabs are arranged vertically on smaller screens.

## Important notes

- `index.php` is the report-generation interface, not the finished report to
  link to directly.
- The visitor-facing output is `kba-statistiken.html`.
- A preview generated by `index.php` is held only in the session; it is not a
  persistent file.
- The workflow intentionally requires manual review and confirmation before
  the saved report is updated.
