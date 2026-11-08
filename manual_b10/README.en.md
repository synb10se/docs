# Leapmotor B10 quick guide

This static web application is an unofficial quick guide to the Leapmotor B10
with the LeapOS 3.41.30 infotainment system. It helps explain key vehicle and
system settings and does not replace the original owner's manual.

## Contents

The articles are grouped in seven menu sections:

- **Getting started:** overview and safety notices
- **Controls:** main screen, status bar and user profiles
- **Vehicle:** charging and discharging, doors and windows, lighting, keys and
  access, and safety and child protection
- **Driving:** driving settings, driver assistance (ADAS) and climate control
- **Infotainment:** sound, voice assistant, display, dashcam, and system,
  security and updates
- **Service:** maintenance, emergency information and technical data
- **Legal:** legal notice

Menu names and features may vary by equipment, market and software version.
Screenshots in the guide show LeapOS 3.41.30.

## Using the guide

The site is static and needs no server-side logic or build step. Select
articles from the menu. Press `/` to focus the search field; navigate results
with the arrow keys and Enter. Click images to open an enlarged view. A
light/dark theme is available and saved in the browser.

## Files

- `index.html` loads the content and interface.
- `assets/content/articles/` contains the articles; `menu.js` defines menu
  groups and order.
- `assets/app.js` implements navigation, search, theme switching and the image
  viewer.
- `assets/style.css` defines the layout and appearance.
- `img/` contains the images used by the articles.

To publish the guide, place the complete contents of this project directory on
a web server. Image paths are relative to `index.html`, so preserve the
directory structure.

## Notice

This guide is unofficial. The original owner's manual is authoritative for
vehicle operation and safety. The web application's start page identifies the
software version and provides further notices.
