# Custom Changes

Changes made to AndroidAPS on top of the upstream
[nightscout/AndroidAPS](https://github.com/nightscout/AndroidAPS) source.
Base: upstream `master` at commit `05a3433598` (3.4.2.3).

---

## 1. Disable redundant version check in applyMaxIOBConstraints

**File:** `plugins/constraints/src/main/kotlin/app/aaps/plugins/constraints/versionChecker/VersionCheckerPlugin.kt`

`triggerCheckVersion()` was being called from inside `applyMaxIOBConstraints`,
which fires every loop cycle (~every 5 minutes). A single intentional check
already runs 30 seconds after launch in `MainApp.doInit()`.

**Fix:** Commented out the call in `applyMaxIOBConstraints`.

```kotlin
//versionCheckerUtils.triggerCheckVersion()
```

---

## 2. Wear OS — rotary bezel input for all numeric entry screens

**Files changed:**
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/ViewSelectorActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/WizardActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/CarbActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/ECarbActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/TreatmentActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/BolusActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/FillActivity.kt`
- `wear/src/main/kotlin/app/aaps/wear/interaction/actions/TempTargetActivity.kt`

`PlusMinusEditText` already had an `onGenericMotion` handler for rotary encoder
events, but it was registered on the `editText` view which has
`isFocusable = false`. On Wear OS, rotary events are dispatched to the focused
view — since `editText` can never be focused, the events were consumed by the
`WearableRecyclerView` (the page scroller) instead.

**Fix:** Added `getCurrentPage()` to `ViewSelectorActivity`, then overrode
`dispatchGenericMotionEvent` in each input activity to intercept rotary events
at the activity level and forward them to the correct `PlusMinusEditText` for
the current page.

Bezel behaviour per screen:

| Screen | Page 0 | Page 1 | Page 2 |
|---|---|---|---|
| Wizard | Carbs | Percentage (if enabled) | — |
| Carbs | Carbs | — | — |
| Extended Carbs | Carbs | Start time | Duration |
| Treatment | Insulin | Carbs | — |
| Bolus | Insulin | — | — |
| Fill / Prime | Insulin | — | — |
| Temp Target | Duration | Low / Target | High (if dual target) |

Spinning clockwise increases the value; counter-clockwise decreases it.
The existing dynamic acceleration (1× → 2× → 4×) applies as you spin faster.
