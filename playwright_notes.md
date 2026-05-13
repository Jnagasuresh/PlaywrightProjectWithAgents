
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
---
# Waits in Playwright

# Waits in Playwright

## 1. Auto-Waiting (Built-in — No Code Needed)

Playwright **automatically waits** before most actions. When you call `.click()`, `.fill()`, etc., it waits for the element to be:

- Attached to DOM
- Visible
- Stable (not animating)
- Enabled
- Editable (for input actions)

```javascript
// Playwright waits automatically — no explicit wait needed
await page.locator('#submit').click();
await page.locator('#name').fill('John');
```

---

## 2. Locator Waits

### `locator.waitFor()`

Waits for an element to reach a specific state.

```javascript
await page.locator('.spinner').waitFor({ state: 'hidden' });
await page.locator('.results').waitFor({ state: 'visible' });
await page.locator('#item').waitFor({ state: 'detached' });
await page.locator('#item').waitFor({ state: 'attached' });
```

---

## 3. Expect / Assertion Waits

`expect()` has **built-in retry logic** — it keeps checking until the assertion passes or times out. Preferred in tests.

```javascript
await expect(page.locator('.toast')).toBeVisible();
await expect(page.locator('.toast')).toBeHidden();
await expect(page.locator('#count')).toHaveText('42');
await expect(page.locator('input')).toHaveValue('hello');
await expect(page.locator('.btn')).toBeEnabled();
await expect(page.locator('.btn')).toBeDisabled();
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveTitle('Home');
```

---

## 4. Page-Level Waits

### `page.waitForURL()`

Waits for the URL to match a string, regex, or predicate.

```javascript
await page.waitForURL('/dashboard');
await page.waitForURL(/\/order\/\d+/);
await page.waitForURL(url => url.includes('success'));
```

### `page.waitForSelector()` *(legacy)*

Older API, still works but locators are preferred.

```javascript
await page.waitForSelector('.modal', { state: 'visible' });
```

### `page.waitForTimeout()` ⚠️

Hard sleep — **avoid in tests**, only for debugging.

```javascript
await page.waitForTimeout(2000); // waits 2 seconds unconditionally
```

---

## 5. Network Waits

### `page.waitForResponse()`

Waits for a specific HTTP response.

```javascript
// By URL string
await page.waitForResponse('**/api/users');

// By predicate
await page.waitForResponse(res =>
  res.url().includes('/api/search') && res.status() === 200
);

// Paired with an action
const [response] = await Promise.all([
  page.waitForResponse('**/api/submit'),
  page.locator('#submit').click()
]);
const data = await response.json();
```

### `page.waitForRequest()`

Waits for a specific outgoing request.

```javascript
const [request] = await Promise.all([
  page.waitForRequest('**/api/login'),
  page.locator('#login').click()
]);
console.log(request.postData());
```

### `page.waitForLoadState()`

Waits for the page to reach a network load state.

```javascript
await page.waitForLoadState('load');             // HTML loaded
await page.waitForLoadState('domcontentloaded'); // DOM ready
await page.waitForLoadState('networkidle');      // no network for 500ms ⚠️ flaky
```

---

## 6. `page.waitForFunction()`

Waits until a custom JavaScript expression returns truthy. Most flexible, but use sparingly.

```javascript
// Wait for a global JS variable to be set
await page.waitForFunction(() => window.__appReady === true);

// Wait for element count
await page.waitForFunction(
  count => document.querySelectorAll('.item').length >= count,
  5 // argument passed in
);

// Wait for localStorage value
await page.waitForFunction(
  () => localStorage.getItem('token') !== null
);
```

---

## 7. Event Waits

### `page.waitForEvent()`

Waits for a browser/page event to fire.

```javascript
// Wait for a new tab/popup
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.locator('#open-tab').click()
]);

// Wait for a file download
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.locator('#download-btn').click()
]);
const path = await download.path();

// Wait for a dialog
page.on('dialog', dialog => dialog.accept());
await page.locator('#alert-btn').click();
```

---

## Quick Reference

| Wait Type | Best For |
|---|---|
| Auto-waiting | Most click/fill/type actions |
| `locator.waitFor()` | Gating logic on element state |
| `expect()` assertions | Test assertions with retries |
| `waitForURL()` | Navigation / redirects |
| `waitForResponse()` | API calls triggered by UI |
| `waitForRequest()` | Inspecting outgoing requests |
| `waitForLoadState()` | Page-level load completion |
| `waitForFunction()` | Custom JS conditions |
| `waitForEvent()` | Popups, downloads, dialogs |
| `waitForTimeout()` | ❌ Debugging only |

---

## Golden Rules

1. **Prefer auto-waiting** — don't add waits if Playwright already handles it
2. **Use `expect()` in assertions** — clearer errors, built-in retry
3. **Use `waitFor()` for control flow** — gating actions, not assertions
4. **Avoid `networkidle` and `waitForTimeout()`** — both are flaky in CI
5. **Pair network waits with `Promise.all()`** — avoids race conditions
