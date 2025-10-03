---
title: "My journey to translating FrogPilot Qt UI to Ukrainian with some UI tuning"
date: 2025-10-02
tags:
  ["openpilot", "frogpilot", "qt", "translations", "localization", "ukrainian"]
---

Over the past weeks, I’ve been working on localizing and extending the
translation system for the [FrogPilot](https://github.com/FrogAi/FrogPilot) -
fork of [OpenPilot](https://github.com/commaai/openpilot) open source car
driving assist projects.  
This post summarizes the process, challenges, and solutions implemented —
especially around **Qt translations** and handling **dynamic alerts**.

![translation demo](/blog/translation-FP/resources/IMG_4254.jpeg) ![translation demo](/blog/translation-FP/resources/IMG_4255.jpeg) 

## Background

Qt provides a built-in translation system using:

- `tr("...")` for translatable strings inside QObject-based classes,
- `QT_TRANSLATE_NOOP("Context", "Text")` for registering strings outside class contexts,
- `.ts` files for storing translations (editable via [Qt Linguist](https://doc.qt.io/qt-5/linguist-translators.html)),
- `.qm` files as compiled translation catalogs.

To enable language support and/or change the translation text, you need to
recompile the UI.
By default, most but not all UI strings in Frog\OpenPilot wrapped in translated
format with `tr()`.
Some dynamic alerts and Python-sourced messages weren’t translatable at all.

## Goals

- Enable a full **Ukrainian translation** of the UI.
- Fix missing strings in `.ts` catalogs.
- Support **dynamic alerts with parameters** (e.g., “Calibration 12%”).
- Localize date/time formats and file size units.
- Contribute changes back upstream to FrogPilot.

## Building and Deployment on Device and Mac

Working with OpenPilot and FrogPilot involves different environments: the
target **device** (usually a comma 3x), local **Mac development**, and
sometimes **Docker containers** for testing.

- **On-device build:**  
  Building directly on the device ensures that all architecture-specific
  dependencies and Qt libraries match the runtime environment. This is the
  most reliable method to test translations and alerts in real conditions.

- **Mac builds and containers:**  
  Compiling on Mac or in containers can sometimes fail due to missing ARM
  cross-compilers, incompatible Qt versions, or file path issues.
  Often, `.ts` updates work locally, but runtime tests may fail if the UI
  binary was not rebuilt for the target architecture. At least I was unable
  to assemble the UI natively on Mac or in a devcontainer.

- **Branch strategy:**  
  FrogPilot maintaines have two types of branches: **built branches** (compiled
  for the device) and **dev branches** (contains all files for
  building/development/translation).
  It is impossible to compile UI elements on the "compiled" branch.
  The **dev** branches are compiled automatically with `scons` during booting device.

- **Repo renaming trick:**  
  Comma.ai's auto-installer expects specific repo names (`openpilot`).
  To make your own fork with name different than `openpilot` you shoud
  **temporary rename** you repo to `openpilot` and back, this creates http
  redirect from old repo name.

## Compiling and Updating Translations

To regenerate translation catalogs, Qt provides `lupdate`:

```bash
lupdate -locations none \
  -recursive selfdrive/ui frogpilot/ui \
  -ts selfdrive/ui/translations/main_uk.ts \
  -I . -no-obsolete
```

- `-locations none` avoids embedding file paths,
- `-no-obsolete` removes vanished strings,
- .ts files are edited in Qt Linguist,
- .qm files are built and loaded at runtime.

## `update_translations.py` script

The standard OpenPilot package includes helpr script, `update_translations.py`,
which helps add and update translation languages.

- Regenerating .ts translation files from Qt UI source code.
- Calls lupdate recursively on selfdrive/ui and frogpilot/ui.
- Handles obsolete (vanished) strings and plural forms.
- Generates alerts_generated.h for offroad alerts JSON.
- Ensures that manual translations in .ts files are preserved while updating
  automatically discovered strings.

However, in real testing, you don't need to update `.ts` manually every time -
that is performed on the device during compilation, and there is no need to do
it manually. To add new language it is sufficient to add defenition to
`/openpilot/selfdrive/ui/translations/languages.json` and run
the `update_translations.py` script once.

## The Challenge: Dynamic Alerts

OpenPilot displays alerts such as:

- Calibration in Progress: 12%
- Drive Above 15 mph
- Take Control, Turn Exceeds Steering Limit

These were hardcoded English strings assembled at runtime, and not present
in `.ts` files.
That meant they could never be translated with the standard Qt pipeline.

### My Solution: Alert Translation Map

I introduce a static translation map (in new `alert_tr.h` alert translation file) containing known alert templates:

```cpp
static const std::vector<AlertTranslation> alertTranslations = {
  {"Calibration in Progress: %1%", QT_TRANSLATE_NOOP("Alerts", "Калібрування триває: %1%")},
  {"Drive Above %1", QT_TRANSLATE_NOOP("Alerts", "Рухайтесь швидше ніж %1")},
};
```

Then vibe-coded a `translateAlert()` function that:

1. Matches the incoming alert text against known patterns (using `QRegularExpression`).
2. Applies `QObject::tr(...)` to load the proper translation.
3. Substitutes parameters (`%1`, `%2`, …) with runtime values.

This way, even dynamic messages are properly localized.

> Example: Integrating Dynamic Alert Translation

Dynamic alerts are generated at runtime, often containing parameters such as
percentages or speed values.
my `translateAlert()` function handles this:

```cpp
// alert_tr.h
inline QString translateAlert(const QString &text, const QStringList &params = {}) {
for (const auto &alert : alertTranslations) {
QRegularExpression rx(alert.raw_text1); // can include (%1) placeholders
QRegularExpressionMatch match = rx.match(text);
if (match.hasMatch()) {
QString translated = QObject::tr(alert.tr_text1);
// automatically substitute %1, %2 ... if parameters are present
for (int i = 1; i < match.lastCapturedIndex() + 1; ++i) {
translated = translated.arg(match.captured(i));
}
return translated;
}
}
return text; // fallback to original text if no match
}
```

Usage in alerts.cc when drawing alerts:

```cpp
if (alert.size == cereal::ControlsState::AlertSize::MID) {
  p.setFont(InterFont(sidebarsOpen ? 78 : 88, QFont::Bold));
  p.drawText(QRect(0, c.y() - 125, width(), 150),
             Qt::AlignHCenter | Qt::AlignTop,
             translateAlert(alert.text1));
  p.setFont(InterFont(sidebarsOpen ? 56 : 66));
  p.drawText(QRect(0, c.y() + 21, width(), 90),
             Qt::AlignHCenter,
             translateAlert(alert.text2));
}
```

With this setup:

- Any alert text defined in `alertTranslations` will be translated.
- Placeholders (`%1`, `%2`, …) are replaced at runtime with actual values.
- Both text1 and text2 of the alert are translated independently,
  supporting single- or multi-line alerts.

## My helper Script

Made helper python script that helps to fill translation dictionary array.

`update_alerts.py`:

- Parses events.py for dynamic alerts strings.
- Extracts `StartupMessage Top/Bottom`.
- Converts Python alert definitions into a C++ alertTranslations array with QT_TRANSLATE_NOOP.
- Handles parameters in alerts (%1, %2), `<br>` for multi-line messages.
- Automatically rewrites alert_tr.h to include all known alerts for the UI.

The script requires refinement, as it does not count for complex alerts with
high variability, so I had to manually refine the dictionary array a bit.

## Localized Formatting

Me also updated date/time and file size displays to respect the selected locale:

```cpp
QLocale locale(uiState()->language.mid(5));
QString date = locale.toString(QDate::currentDate(), "d MMMM yyyy");
```

and

```cpp
if (totalSize >= GB)
  return QString::number(totalSize / GB, 'f', 2) + QObject::tr(" GB");
else
  return QString::number(totalSize / MB, 'f', 2) + QObject::tr(" MB");
```

So users now see ГБ / МБ and 24-hour time where appropriate.

## Lessons Learned

- `tr()` only works inside QObject contexts. For global maps, `QT_TRANSLATE_NOOP`
  is required.
- `lupdate` is strict: any string not wrapped in `tr()` or `QT_TRANSLATE_NOOP` is
  invisible to the translation system.
- Dynamic strings need patterns and a translation layer above the raw alerts.

## Contribution

Me upstreamed the changes in this pull request:
👉 FrogPilot [PR](https://github.com/FrogAi/FrogPilot/pull/282) #282

If you’re interested in contributing translations:

- FrogPilot: [Contributing Guidelines](https://github.com/FrogAi/FrogPilot#contributing)
- Qt Translations: [Qt Linguist Manual](https://doc.qt.io/qt-5/linguist-translators.html)

⸻

Final Thoughts

Translating a large C++/Qt project like OpenPilot is not only about .ts files —
it also requires _rethinking how runtime messages are generated and displayed_.
The result is a much more user-friendly interface, now available in Ukrainian,
with groundwork laid for other languages.

---
