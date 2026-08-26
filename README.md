# 30DayApp

Name:Bhume Limpaisansakun

ID:2320210160

An Android app that recommends one Bangkok restaurant for every day of the month —
from street-side crab omelettes to three-star tasting menus — as tap-to-expand
cards in a scrolling list.

Built with Jetpack Compose and Material 3 as the final project for
[Android Basics with Compose](https://developer.android.com/courses/android-basics-compose/course),
Unit 3.

## What it does

Thirty restaurants, ordered so that reading day by day walks you outward across the
city rather than criss-crossing it: the Old Town first, then Chinatown, Charoen
Krung, Silom and Sathorn, Pratunam and Lumphini, and finally Sukhumvit. Street
food, noodle shops and fine dining are interleaved throughout rather than sorted by
price, so no stretch of the month is all cheap or all expensive.

A collapsed card shows the day, the restaurant, and its cuisine and neighbourhood.
Tapping anywhere on it expands the card to reveal the photo, the dish to order, and
why the place is worth the trip.

## Features

- **30 restaurants** in a scrolling `LazyColumn`, one per day
- **Tap to expand** — the card grows with `animateContentSize` and the chevron
  rotates 180° via `animateFloatAsState`
- **Custom Material 3 brand** — a hand-written charcoal-and-saffron palette with
  full light and dark schemes
- **Survives rotation** — expansion state is held in `rememberSaveable`
- **Fully offline** and **zero third-party dependencies** — no networking, no image
  loading library, nothing beyond AndroidX and Compose
- **Localisable** — every user-facing string is a resource, referenced by id

## Built with

| | |
|---|---|
| Language | Kotlin 2.2.10 |
| UI | Jetpack Compose (BOM 2026.02.01), Material 3 |
| Min / target SDK | 24 / 37 |
| Build | AGP 9.3.2, Gradle 9.5 |
| Testing | JUnit 4, Compose UI Test |

## Getting started

Clone the repository, open it in Android Studio, and press **Run**.

To build from the command line you need a **JDK 25** on `JAVA_HOME` — the runtime
bundled with Android Studio is one:

```bash
export JAVA_HOME="/c/Program Files/Android/Android Studio/jbr"
./gradlew :app:assembleDebug
```

## Architecture

Deliberately simple, and scoped to what Unit 3 covers — there is no ViewModel,
because a card's expanded state is local to that card and `rememberSaveable`
handles it without one.

```
app/src/main/java/com/example/a30dayapp/
├── MainActivity.kt          entry point, plus a @Preview per colour scheme
├── model/Restaurant.kt      the data class
├── data/Datasource.kt       the 30 entries — the only place content lives
└── ui/
    ├── GourmetApp.kt        top bar and the scrolling list
    ├── RestaurantCard.kt    one card, its expand state and its animations
    └── theme/               Color.kt, Theme.kt, Type.kt
```

`Restaurant` holds resource ids rather than literal strings:

```kotlin
data class Restaurant(
    val day: Int,
    @param:StringRes val nameResourceId: Int,
    @param:StringRes val cuisineAreaResourceId: Int,
    @param:StringRes val signatureDishResourceId: Int,
    @param:StringRes val descriptionResourceId: Int,
    @param:DrawableRes val imageResourceId: Int,
)
```

Two details worth calling out, because both were bugs waiting to happen:

- **Photos live in `res/drawable-nodpi/`, not `res/drawable/`.** A bitmap in plain
  `drawable/` is assumed to be mdpi and gets upscaled by the device's density
  factor as it decodes. On an xxhdpi phone that is 3×, so a 2976×1984 photo would
  become an 8928×5952 bitmap — roughly 212 MB — and the app would be killed.
  `nodpi` means "use this at its native size".
- **The chevron is a vector drawable, not a Material icon.**
  `material-icons-core` is not a transitive dependency of material3 in this Compose
  BOM, and `material-icons-extended` is deprecated. Drawing the glyph avoids
  pulling in an artifact for a single icon.

## Theme

A hand-written charcoal-and-saffron palette. Dynamic colour is deliberately
switched off — on Android 12+ it would repaint the app with the device wallpaper's
colours and lose the brand entirely. The app follows the device's light/dark
setting.

To force the charcoal look regardless of the device, change one default in
`ui/theme/Theme.kt`:

```kotlin
fun GourmetTheme(
    darkTheme: Boolean = true,
```

Typography names its `FontFamily` explicitly in every style, so dropping a `.ttf`
into `res/font/` and pointing `DisplayFamily` / `BodyFamily` at it is a two-line
change.

## Testing

```bash
./gradlew :app:testDebugUnitTest        # unit tests
./gradlew :app:connectedDebugAndroidTest # UI tests — needs a device or emulator
```

`DatasourceTest` is the one that earns its keep. Thirty near-identical entries
referencing 150 resource ids make a mistyped prefix close to inevitable, and that
kind of bug stays invisible until you scroll to the affected day. It asserts there
are exactly 30 entries, that days run 1–30 with no gaps or repeats, and that no
drawable or string resource is used twice.

`GourmetAppTest` covers the screen: the title renders, day 1's card shows its
restaurant, and tapping it reveals the description.

## Customising

**Photos.** Each day's image is `res/drawable-nodpi/rNN_<name>.jpg`. Drop a
replacement over the existing file, keeping the name, and no Kotlin changes.
`scripts/import_photos.sh` bulk-imports a folder of `Day1.jpg … Day30.jpg`, and
`scripts/make_placeholders.sh` regenerates the generated placeholder art for any
day with no photo.

**Text.** Everything is in `res/values/strings.xml`, four strings per restaurant:
`rNN_name`, `rNN_area`, `rNN_dish`, `rNN_description`. Two encoding traps to know
about — `&` must be written `&amp;`, and an apostrophe must be escaped as `\'`.
Both are silent build failures otherwise.

## The 30 days

| Days | Area | Restaurants |
|---|---|---|
| 1–6 | Old Town / Phra Nakhon | Jay Fai · Thipsamai · Krua Apsorn · Err · Nusara · Rongros |
| 7–10 | Yaowarat (Chinatown) | Nai Ek Roll Noodles · Hua Seng Hong · T&K Seafood · Potong |
| 11–12 | Charoen Krung | 80/20 · Charmgang |
| 13–18 | Silom / Sathorn / Bang Rak | Prachak Pet Yang · Le Du · Sorn · Sühring · Somtum Der · Blue by Alain Ducasse |
| 19–23 | Pratunam / Siam / Lumphini | Go-Ang Chicken Rice · Jeh O Chula · Polo Fried Chicken · Gaggan Anand · Paste |
| 24–30 | Sukhumvit / Thonglor / Ekkamai | Rung Rueang · Wattana Panich · Supanniga · Appia · Peppina · Sushi Masato · Baan Tepa |
