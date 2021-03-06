# Localization and Internationalization
As you'd expect, given [Wikipedia's mission](https://wikimediafoundation.org/wiki/Mission_statement) and [core values](https://wikimediafoundation.org/wiki/Values), making the iOS app accessible & usable in as many languages as possible is very important to us.  This document is a quick overview of how we localize strings as well as the app itself.

## Localized strings
Localized strings which are used by the app are all kept in **Localizable.strings**.  We do not use localized string entries generated by Interface Builder because the keys aren't human readable.
### Strings displayed in the app
#### Adding a new localized string
Currently, we're not using `NSLocalizedString` directly.  This isn't ideal for a couple of reasons, but the current workflow for adding a new localized string is as follows:

1. Add an entry to **Localizations/en.lproj/Localizable.strings** with the English version of your string, using a hyphenated, lowercase key. For example, `"my-new-key" = "Hello world.";`.
2. Add another entry to **Localizations/qqq.lproj/Localizable.strings** with a description of what this localization is for. For example, `"my-new-key" = "Label for a button that does a thing.";`.
3. Retrieve your string by the chosen key:
  - ObjC: `MWKLocalizedString(@"my-new-key", nil);`
  - Swift: `localizedStringForKeyFallingBackOnEnglish(key)`

See `WMFLocalization.h` for more details.

#### Modifying a localized string
**Do not change the keys for localized strings.** Unless absolutely necessary, keys should remain the same to prevent complications when syncing with TWN.

### Importing translations
Translations are imported from [translatewiki.net](https://translatewiki.net) using the workflow documented on [mediawiki.org translation doc](https://www.mediawiki.org/wiki/Translation_of_app_string_resources#TWN_sync_for_iOS).

### Displaying text in the app
_TODO: elaborate on considerations when displaying text in the app, e.g. overflow/multiline for verbose languages, RTL handling, etc._

### Navigating the app
_TODO: elaborate on considerations when implementing navigation in the app, i.e. RTL handling_

### Testing the app
#### Indications for international testing
Some important things to test across different locales (and operating systems):

- View layout in LTR & RTL environments
- custom `NSDateFormatter`
- Data models for horizontal navigation which need to be reversed when app is RTL (e.g. image gallery data sources)

Text overflow is also an important consideration when designing and implementing views, but doesn't require exhaustive locale testing.  Typically, it's sufficient to pass short, medium, and long strings to the test subject and verify proper wrapping, truncating, and/or layout behavior. See [`WMFArticleListCellVisualTests`](../WikipediaUnitTests/Code/WMFArticleListCellVisualTests.m) for an example.

#### Internationalization testing strategies
We run a certain set of tests across multiple operating systems and locales to verify business logic, and especially views, exhibit proper conditional behavior & appearance.  From a project setup standpoint, this involves:

- Running LTR tests on the main scheme on iOS 8 & 9 simulators
- Running RTL tests in a separate, **Wikipedia RTL** scheme on iOS 8 & 9 simulators

> The RTL locale & writing direction are forced in the scheme using launch arguments as described in the [Testing Right-to-Left Layouts](https://developer.apple.com/library/ios/documentation/MacOSX/Conceptual/BPInternational/TestingYourInternationalApp/TestingYourInternationalApp.html) section of Apple's "Internationalization and Localization Guide."

##### Unit tests
Ideally, the code should be factored in such a way that the relevant inputs (i.e. OS version and/or layout direction) can be passed explicitly during tests.  For example, given a method that returns a different value based on a layout direction:
``` objc
// Method invoked in unit tests w/ different layout directions
- (BOOL)methodDoingSomethingForWritingDirection:(UIUserInterfaceLayoutDirection)layoutDirection;

// Method invoked in application code, which passes the `[[UIApplication sharedApplication] userInterfaceLayoutDirection]`
// to the first argument of the first method signature.
- (BOOL)methodDoingSomethingForApplicationWritingDirection;
```

In other cases where this isn't feasible, you'll need to add your test class to the **WikipediaRTL** scheme so that the application itself is in RTL.  Also, you'll need to write assertions based on the writing direction and/or OS at runtime (see [`WMFGalleryDataSourceTests`](../WikipediaUnitTests/Code/WMFGalleryDataSourceTests.m#L36) for an example). [`NSDate+WMFPOTDTitleTests`](../WikipediaUnitTests/Code/NSDate+WMFPOTDTitleTests.m) are another example that rely on the application state, and verify that the date is not affected by the current locale—which `NSDateFormatter` implicitly uses when computing strings from dates.

##### Visual tests
Visual tests can be incredibly useful when verifying LTR & RTL responsiveness across multiple OS versions.  Write you visual test as you normally would, ensure it's added to the **WikipediaRTL** scheme, and use the `WMFSnapshotVerifyViewForOSAndWritingDirection` convenience macro to record & compare your view with a reference image dedicated to a specific OS version and writing direction.  See [`WMFTextualSaveButtonLayoutVisualTests`](../WikipediaUnitTests/Code/WMFTextualSaveButtonLayoutVisualTests.m) for an example.

