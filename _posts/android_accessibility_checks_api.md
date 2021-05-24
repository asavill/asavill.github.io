At the core of the solution is the AccessibilityChecks class. This class ships with the androidx.test.espresso.accessibility library and allows automated accessibility checks in Espresso tests that cover a range of common [accessibility issues]([Accessibility-Test-Framework-for-Android/AccessibilityCheckPreset.java at master · google/Accessibility-Test-Framework-for-Android · GitHub](https://github.com/google/Accessibility-Test-Framework-for-Android/blob/master/src/main/java/com/google/android/apps/common/testing/accessibility/framework/AccessibilityCheckPreset.java)). 

Enabling the AccessibilityChecks is as simple as calling enable as part of your Espresso test setup.

```kotlin
 AccessibilityChecks.enable()
```

Once enabled, the new checks will run when any view action is performed. The default behaviour is for the check to be made on any view that the action is performed on, as well as any descendant view(s). 

To take the hierarchy approach one setup further you can also request that the AccessibilityCheck class runs all the checks for the entire view hierarchy of a screen.

```kotlin
AccessibilityChecks.enable().setRunChecksFromRootView(true)
```

It's recommended to do this instead of focusing solely on intractable views such as buttons and lists because it  may highlight other common problems such a duplicate speakable text in a subtitle/title, poor contrast or inefficient view traversal orders.

### Adding a rule

A simple way of applying these checks to your espresso tests would be to introduce a @Rule. This ensures that the test remains free of most of the boilerplate for enabling and disabling the accessibility checks and keeps any amendments to the AcessibilityChecks setup in one place.

```kotlin
class AccessibilityChecksTestRule : ExternalResource() {
    override fun before() {
        AccessibilityChecks.enable().setRunChecksFromRootView(true)
    }

    override fun after() {
        AccessibilityChecks.disable()
    }
}
```

You can then simply include the rule in any espresso test and the checks will run when the view action is performed.

```kotlin
@RunWith(AndroidJUnit4::class)
class AccessibilityTest {
    @get:Rule
    val accessibilityChecks = AccessibilityChecksTestRule()

  @Test
    fun clickAButton() {

       Espresso.onView(ViewMatchers.withId(R.id.button))
            .perform(ViewActions.click())

    }
}
```

### HELP! My view is breaking all of the rules!

Once the checks are running you may find more of them are failing than you anticipated. This can be a problem as it creates a failing test in your suite, and you may need to make some new design choices that you can't address immediately.  To good news is that the AccessibilityChecks API allows you to provide a SupressingResultMatcher to temporarily ignore certain checks. 

Under these circumstances, it's best to use a ViewMatcher to ensure only specific checks about specific views are suppressed.

```kotlin
AccessibilityChecks.enable().apply {
        setSuppressingResultMatcher(
                allOf(
                    matchesCheckNames(`is`("TouchTargetSizeCheck")),
                    matchesViews(withId(R.id.button))
                )
        )
}
```
