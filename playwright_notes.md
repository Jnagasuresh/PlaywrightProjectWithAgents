
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
