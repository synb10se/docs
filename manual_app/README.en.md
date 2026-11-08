# Leapmotor app quick guide

This static web application is an unofficial quick guide to the Leapmotor app
for Android and iOS. It documents app features and supplements the
manufacturer's information; it is not an official manual. Screenshots are
from iOS, so the Android interface may differ.

## Contents

The articles are grouped in four menu sections:

- **Getting started:** overview and vehicle setup
- **My vehicle:** vehicle overview, charging, climate and preconditioning,
  Bluetooth key and sharing, and energy consumption
- **Personal centre:** vehicle management and app settings
- **Legal:** legal notice

The guide identifies Leapmotor app version 1.18.2 (October 2026) as its
reference version. Menus, features and availability may change with app
updates.

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

This guide is unofficial. Leapmotor's information is authoritative. The
web application's start article and legal notice also describe the app version
and other manufacturer-related notices.
