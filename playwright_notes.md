
## Some Important Notes

## .pressSequentially:
types text character by character, simulating actual keyboard input (keydown → keypress → keyup for each character).

```
await page.locator('selector').pressSequentially(text, options?)
```

Options:
delay – milliseconds between keystrokes (default: 0)
timeout – max wait time
noWaitAfter – skip waiting for navigation/network
---
## waitfor
waitFor() is a method on a Locator that waits until an element reaches a specified state before proceeding.

Syntax:
```
await page.locator('selector').waitFor(options?)
```
Options:

- state – what to wait for: 'attached' | 'detached' | 'visible' | 'hidden' (default: 'visible')
- timeout – max wait in ms (default: 30000)

**4 States**:
```
// Element exists in DOM (may be invisible)
await locator.waitFor({ state: 'attached' });

// Element is gone from DOM entirely
await locator.waitFor({ state: 'detached' });

// Element is visible on screen (default)
await locator.waitFor({ state: 'visible' });

// Element is hidden or removed
await locator.waitFor({ state: 'hidden' });
```
### Common Use cases

1. Wait for a loader/spinner to disappear
   ```
   await page.locator('.loading-spinner').waitFor({ state: 'hidden' });
await page.locator('.data-table').waitFor({ state: 'visible' });
```
3. Wait for dynamic content to appear
```
await page.locator('.search-results').waitFor();
// now safe to interact
await page.locator('.search-results li').first().click();
```
4. Wait for a modal to open
```
await page.locator('#submit-btn').click();
await page.locator('.modal').waitFor({ state: 'visible' });
await page.locator('.modal .confirm').click();
```
5. Wait for element to detach after deletion
6. Wait for toast/notification to disappear before next step
   ```
   await page.locator('.toast-success').waitFor({ state: 'hidden' });
await page.locator('#next-action').click();
```
7. Wait for async content after navigation
   ```
   await page.goto('/dashboard');
await page.locator('.user-greeting').waitFor(); // waits for dynamic render
```
