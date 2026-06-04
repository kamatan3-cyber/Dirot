# Dirot — Apartment Rating App — Project Instructions

## Language and Direction
- All UI text must be in Hebrew
- All screens must use right-to-left (RTL) text direction
- Do not use emojis anywhere in the app

## Design
- Minimalist and modern design
- Clean layout with clear visual hierarchy
- Color coding for final scores: green (≥4), amber (≥3), red (<3)
- Home screen displays top 3 apartments in a podium layout ranked by Value for Money
- Home screen heading is a dynamic time-based greeting: "בוקר טוב," / "צהריים טובים," / "ערב טוב,"
- Plus (+) button in header opens the add apartment form
- Hamburger menu contains: כל הדירות, השוואת דירות, הגדרות
- Back buttons are displayed as a left-pointing arrow icon (←), no text

## Branding
- Browser tab title: "Dirot"
- Logo file: "Dirot2.png" (project root)
- Logo appears in the top-left of the header on every screen
- Logo is also used as the browser tab favicon

## Apartment Fields
Each apartment contains the following fields:
- Apartment name (mandatory)
- City (עיר) — mandatory
- Street and number (רחוב ומספר) — mandatory
- Price in ILS (₪), displayed with ₪ symbol throughout
- Room score (1–5)
- Living room and balcony score (1–5)
- Kitchen score (1–5)
- Location score (1–5)
- Resident quality score (1–5) — "איכות דיירים"
- Number of residents (1, 2, 3, or 4) — "מספר דיירים"
- Photos — each photo can be tagged to a room: חדר, סלון ומרפסת, מטבח, or כללי (untagged)
- One photo selected as profile image
- Notes (free text)
- Visit date

## Photo Tagging
- When uploading photos in the add/edit form, each image shows tag buttons below it: כללי, חדר, סלון ומרפסת, מטבח
- Tags are stored in a parallel array `imageTags` alongside the `images` array
- In the apartment profile page, the assigned label appears below each photo
- Tags are used for photo comparison in the compare screen

## Price Scoring — Fixed Scale (ILS / monthly rent)
Price scores are fixed (not dynamic). Prices represent monthly rent in Israeli Shekels:
- ₪900–₪1,100  → score 5
- ₪1,101–₪1,300 → score 4
- ₪1,301–₪1,600 → score 3
- ₪1,601–₪1,800 → score 2
- ₪1,801–₪2,100 → score 1
- Outside ₪900–₪2,100 range: display a warning; price score is not calculated (null)
- Price 0 / not entered: price score is null (no price score calculated)

## Apartment Score Formula
The final score is calculated on a scale of 1 to 5:
- 35% — resident quality (איכות דיירים)
- 35% — average of room, living room + balcony, and kitchen scores
- 20% — location score
- 10% — number of residents: weighted by configurable per-count scores (see Settings)

## Resident Count Scoring — Default Values
The per-count score for number of residents is fully configurable in the Settings screen.
Default values:
- 1 דייר  → 5 pts
- 2 דיירים → 4 pts
- 3 דיירים → 5 pts
- 4 דיירים → 3 pts

These are stored in `apt-settings` under the key `cntScores` and editable in the "ציון לפי מספר דיירים" settings section.

## Value for Money Score
- Calculated on a scale of 1 to 5
- Uses fixed price score (see above); if null, VFM equals the apartment score
- Default weight mix: 50% final score, 50% price score
- Weight mix is editable in the settings screen

## Address Fields
- The address is split into two separate fields: **עיר** (city) and **רחוב ומספר** (street and number)
- Both are stored separately in the apartment object as `city` and `street`
- Backward compatibility: old data with a single `address` field is migrated to `street` on load
- `fullAddress(apt)` helper returns `"street, city"` for display and geocoding; falls back to legacy `address` field
- City and street are displayed separately throughout the app (profile page details, ranking cards, compare columns)
- For geocoding (map features), the combined full address is used

## Navigation Structure
- Home → podium of top 3 by VFM; clicking any podium card opens apartment profile
- "כל הדירות" button on home / menu item → כל הדירות screen (internal screen name: 'ranking')
- Clicking any apartment card in כל הדירות → apartment profile page
- Pencil icon on each card in כל הדירות → edit form for that apartment
- Back from edit-form → כל הדירות
- Back from profile → כל הדירות
- Back from all other inner screens → home
- "עריכת דירות" is NOT in the hamburger menu

## כל הדירות Screen
- Title: "כל הדירות"
- Sort options: ציון סופי, ערך לכסף, מחיר (ascending)
- Favorites filter toggle
- "הצג על המפה" button — opens all apartments on a single Leaflet map (geocodes each address via Nominatim, 700ms between requests)
- No PDF or Excel export buttons
- Each card has: מפה, שתף, star fav toggle (☆/★), pencil edit icon, מחק
- Clicking a card (not an action button) opens the apartment profile page
- No history button (history feature removed)

## Favorites
- Favorite toggle button uses star icons: ☆ (not favorited) and ★ (favorited)
- The amber "מועדף" badge still appears in card titles as a visual indicator
- `fav-on` CSS class applies amber styling to the active star button

## Apartment Profile Page
- Top section: profile image displayed as a tall portrait rectangle (300px height, object-fit: cover, anchored to bottom of image)
- Score section: final score, VFM score, price score (if calculable)
- Details section: price (₪), visit date, number of residents (מספר דיירים), notes
- Ratings section: all individual scores (room, living, kitchen, location, resident quality)
- Bottom: all apartment photos in a horizontal scrollable row; each photo shows its room tag label below it

## Apartment Comparison Screen
- Always compares exactly two apartments (selected by checkbox)
- Desktop: side-by-side column cards with image, scores, and parameter rows
- Mobile (≤560px): compact table layout — header row with apartment names, then one row per parameter showing the label on the right and both scores side by side
- Photo comparison section below with buttons: מטבח, סלון ומרפסת, חדר
- Clicking a photo-compare button opens a modal with side-by-side photos tagged to that room parameter for each apartment, plus the score for that parameter

## Settings Screen
- Editable weight percentages for apartment score parameters (must sum to 100%)
- Editable VFM weight mix between final score and price score (must sum to 100%)
- Editable per-count resident scores (0–5) for 1, 2, 3, and 4 דיירים
- Reset to defaults button

## History Feature
- **Removed.** The edit history modal, history button, and `trackHistory` function are no longer part of the UI.
- Existing `history` arrays in stored apartment data are preserved (not deleted), but no longer displayed or updated.

## Data Storage
- All data stored in localStorage only
- Key `apts`: JSON array of apartment objects
- Key `apt-settings`: JSON object with `weights`, `vfmWeights`, and `cntScores`
- Apartment object includes `imageTags: string[]` parallel to `images: string[]`
- Apartment object stores `city` and `street` separately; legacy `address` field preserved for backward compat
- Never delete or reset existing localStorage data

## General Rules
- Always preserve existing functionality when adding new features
- Keep all forms and screens consistent in style
- Never remove existing fields or features unless explicitly asked
- All prices displayed with ₪ symbol
- Use "דיירים" (not "שותפים") throughout the entire app for resident count and quality fields
