---
title: Android  Html.from() 末尾大量空白去除
date: 2022-04-10 11:34:41
tags: [android, html]
---

### Android  Html.from() 末尾大量空白去除

### 解决方法

```kotlin
    /**
     * Trims trailing whitespace.
     *
     * Removes any of these characters:
     * - 0009, HORIZONTAL TABULATION
     * - 000A, LINE FEED
     * - 000B, VERTICAL TABULATION
     * - 000C, FORM FEED
     * - 000D, CARRIAGE RETURN
     * - 001C, FILE SEPARATOR
     * - 001D, GROUP SEPARATOR
     * - 001E, RECORD SEPARATOR
     * - 001F, UNIT SEPARATOR
     *
     * @return "" if source is null, otherwise string with all trailing whitespace removed
     */
    private fun CharSequence.trimTrailingWhitespace(): CharSequence {
        var i = length

        // loop back to the first non-whitespace character
        while (--i >= 0 && Character.isWhitespace(this[i])) {
            continue
        }
        return this.subSequence(0, i + 1)
    }
```

