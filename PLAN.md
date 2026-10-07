# Plan: Fix opaque snap-area preview when "Blur footprint" is on

Upstream issue: [rxhanson/Rectangle#1897](https://github.com/rxhanson/Rectangle/issues/1897)
Reporter: Rectangle v2.0.3, macOS 27.0.1. The preview is solid white (light) or black (dark) even with blur enabled. v1.1 was translucent. Granting Screen Recording permission did not help.

Branch: `claude/gallant-mayer-mwqmnq`

## Engineering principles (apply to every item)

- [ ] Use best practices, the latest Swift version and the latest syntax and APIs. Gate newer APIs with `#available` and keep the deployment-target fallback.
- [ ] Code is clear, short and readable, written for humans. No clever tricks, no dead code, no speculative abstractions.
- [ ] Every behavior change is covered by tests, and the tests are actually run and pass.
- [ ] Match the existing coding style (`CONTRIBUTING.md`).
- [ ] Bug fix only. No new features, since upstream is not accepting them.
- [ ] Keep the diff minimal and limited to the fix.

## Suspected causes (`Rectangle/Snapping/FootprintWindow.swift`, `Rectangle/Defaults.swift`)

- With blur on, window alpha is forced to 1 and the configured `footprintAlpha` becomes the opacity of a solid white/black tint (`updateAppearance`). v1.1 applied alpha to the window. A stored `footprintAlpha` from an older version can therefore make the tint opaque. An unset key gives 0, so fresh installs and upgraded users behave differently.
- The `.fullScreenUI` material with `.behindWindow` blending, in a `.borderless` window, may render opaquely on macOS 26/27.
- "Reduce transparency" deliberately forces a solid fill. The reporter may have it enabled.

## Checklist

### 1. Investigate

- [ ] Ask the reporter for `defaults read com.knollsoft.Rectangle footprintAlpha`, the Reduce Transparency setting, and whether a fresh profile shows the blur.
- [ ] Confirm which cause applies (stored alpha, material rendering or accessibility).

### 2. Implement

- [ ] Keep the blur tint translucent for stored or legacy `footprintAlpha` values.
- [ ] Rendering fix if the cause is macOS 26/27: try a different material, or `NSGlassEffectView` on macOS 26+ with an `NSVisualEffectView` fallback.
- [ ] Keep Reduce Transparency behavior unchanged (solid fill).
- [ ] Keep non-blur footprint behavior unchanged.
- [ ] Keep config import/export of `footprintAlpha` unchanged.

### 3. Tests (`RectangleTests/RectangleTests.swift`, `FootprintAlphaDefaultsTests`)

- [ ] Blur tint stays translucent for a stored alpha of 1, 0.8 and 0.3.
- [ ] Unset alpha still gives 0 with blur and 0.3 without.
- [ ] Reduce Transparency still gives an opaque fill and `usesBlur == false`.
- [ ] Existing footprint tests still pass.
- [ ] Run the test suite (`xcodebuild test -scheme Rectangle`) and record the result.

### 4. Manual verification (needs a Mac)

- [ ] Light, dark and forced Light/Dark blur appearance.
- [ ] Reduce Transparency on and off.
- [ ] Fresh install and upgrade with a stored `footprintAlpha`.
- [ ] macOS 14, 15, 26 and 27 where available.
- [ ] Preview animation, fade and shadow still work.

### 5. Delivery

- [ ] Commit with a clear message.
- [ ] Push to `claude/gallant-mayer-mwqmnq`.
- [ ] Open a PR only when asked, with the Rectangle and macOS versions and a before/after screenshot.

## Status

| Area | State |
|---|---|
| Issue read, code analysed | done (nothing implemented) |
| Implementation | not started |
| Tests | not started |
| Manual testing | blocked: no macOS in this environment |
