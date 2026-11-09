# Sliding Window in Java --- Coding Templates

A practical GitHub reference for the most common sliding-window patterns
in Java.

> **Core idea:** Maintain a contiguous range `[left, right]` while
> moving the pointers so each element enters and leaves the window at
> most once (in standard two-pointer windows).

## Table of contents

1.  [How to recognize sliding
    window](#1-how-to-recognize-sliding-window)
2.  [Fixed-size window](#2-fixed-size-window)
3.  [Variable-size window --- longest valid
    window](#3-variable-size-window--longest-valid-window)
4.  [Variable-size window --- shortest valid
    window](#4-variable-size-window--shortest-valid-window)
5.  [At most K distinct elements](#5-at-most-k-distinct-elements)
6.  [Exactly K distinct elements](#6-exactly-k-distinct-elements)
7.  [Frequency-map window](#7-frequency-map-window)
8.  [Anagram / permutation matching](#8-anagram--permutation-matching)
9.  [Minimum window substring](#9-minimum-window-substring)
10. [Counting subarrays using at-most
    subtraction](#10-counting-subarrays-using-at-most-subtraction)
11. [Sliding window maximum/minimum using
    deque](#11-sliding-window-maximumminimum-using-deque)
12. [Binary search + sliding window](#12-binary-search--sliding-window)
13. [Circular sliding window](#13-circular-sliding-window)
14. [Two arrays / concatenated
    words](#14-two-arrays--concatenated-words)
15. [Common pitfalls](#15-common-pitfalls)
16. [Pattern selection cheat sheet](#16-pattern-selection-cheat-sheet)

------------------------------------------------------------------------

## 1. How to recognize sliding window

Sliding window is a strong candidate when the problem asks about a
**contiguous** subarray or substring and needs a maximum, minimum,
count, or validity check.

Ask these questions:

1.  Is the required range contiguous?
2.  Can I update the answer efficiently when one item enters and another
    leaves?
3.  Is the window size fixed, or does it expand and shrink?
4.  What exact condition makes the current window valid?

### Important limitation

The classic variable-size sum window generally requires **non-negative
array values** for the usual expand/shrink logic. With negative values,
use prefix sums, a monotonic deque, or another suitable technique.

------------------------------------------------------------------------

## 2. Fixed-size window

**Use when:** Every candidate window must contain exactly `k` elements.

Examples: - Maximum sum of a subarray of size `k` - Maximum average of a
subarray of size `k` - Count windows satisfying a condition -
Fixed-length character/frequency matching

### Template: running sum

``` java
static long maxSumSizeK(int[] nums, int k) {
    int n = nums.length;
    if (k <= 0 || k > n) {
        throw new IllegalArgumentException("Invalid window size");
    }

    long windowSum = 0;
    long answer = Long.MIN_VALUE;

    for (int right = 0; right < n; right++) {
        // 1. Add the new element.
        windowSum += nums[right];

        // 2. Remove the element that falls outside the size-k window.
        if (right >= k) {
            windowSum -= nums[right - k];
        }

        // 3. Process only complete windows.
        if (right >= k - 1) {
            answer = Math.max(answer, windowSum);
        }
    }

    return answer;
}
```

**Complexity:** `O(n)` time, `O(1)` extra space.

### Template: fixed-size window with `left` and `right`

``` java
static void fixedWindow(int[] nums, int k) {
    int left = 0;
    long sum = 0;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];

        if (right - left + 1 > k) {
            sum -= nums[left];
            left++;
        }

        if (right - left + 1 == k) {
            // Process nums[left ... right].
            // Example: update maximum, count, or answer.
        }
    }
}
```

### Template: count fixed windows meeting a condition

``` java
static int countWindows(int[] nums, int k, long threshold) {
    int left = 0;
    int count = 0;
    long sum = 0;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];

        if (right - left + 1 > k) {
            sum -= nums[left++];
        }

        if (right - left + 1 == k && sum >= threshold) {
            count++;
        }
    }

    return count;
}
```

------------------------------------------------------------------------

## 3. Variable-size window --- longest valid window

**Use when:** Find the longest contiguous range satisfying a condition.

Typical structure: 1. Expand by moving `right`. 2. Update window state.
3. While invalid, remove `nums[left]` and increment `left`. 4. Update
the answer after the window becomes valid.

``` java
static int longestValidWindow(int[] nums) {
    int left = 0;
    int answer = 0;

    for (int right = 0; right < nums.length; right++) {
        // Add nums[right] to the window state.

        while (/* window is invalid */) {
            // Remove nums[left] from the window state.
            left++;
        }

        // The current window is valid.
        answer = Math.max(answer, right - left + 1);
    }

    return answer;
}
```

### Example: longest subarray with sum at most `k` (non-negative values)

``` java
static int longestSubarraySumAtMostK(int[] nums, long k) {
    int left = 0;
    int answer = 0;
    long sum = 0;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];

        while (left <= right && sum > k) {
            sum -= nums[left++];
        }

        answer = Math.max(answer, right - left + 1);
    }

    return answer;
}
```

**Complexity:** `O(n)` time, `O(1)` extra space.\
**Assumption:** All array values are non-negative.

------------------------------------------------------------------------

## 4. Variable-size window --- shortest valid window

**Use when:** Find the minimum length of a contiguous range satisfying a
condition.

Typical structure: 1. Expand `right`. 2. Once the window is valid,
record its length. 3. Shrink from the left while it remains valid.

``` java
static int shortestValidWindow(int[] nums) {
    int left = 0;
    int answer = Integer.MAX_VALUE;

    for (int right = 0; right < nums.length; right++) {
        // Add nums[right] to the window state.

        while (/* window is valid */) {
            answer = Math.min(answer, right - left + 1);

            // Remove nums[left] from the window state.
            left++;
        }
    }

    return answer == Integer.MAX_VALUE ? 0 : answer;
}
```

### Example: minimum-size subarray sum at least `target`

``` java
static int minSubArrayLen(int target, int[] nums) {
    int left = 0;
    int answer = Integer.MAX_VALUE;
    long sum = 0;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];

        while (sum >= target) {
            answer = Math.min(answer, right - left + 1);
            sum -= nums[left++];
        }
    }

    return answer == Integer.MAX_VALUE ? 0 : answer;
}
```

**Complexity:** `O(n)` time, `O(1)` extra space.\
**Assumption:** The standard solution assumes positive array elements.

------------------------------------------------------------------------

## 5. At most K distinct elements

**Use when:** The window may contain no more than `k` distinct values.

Useful for: - Longest substring with at most `k` distinct characters -
Fruit Into Baskets - Longest subarray with at most `k` distinct numbers

### Generic array template

``` java
static int longestAtMostKDistinct(int[] nums, int k) {
    if (k < 0) return 0;

    Map<Integer, Integer> freq = new HashMap<>();
    int left = 0;
    int answer = 0;

    for (int right = 0; right < nums.length; right++) {
        freq.put(nums[right], freq.getOrDefault(nums[right], 0) + 1);

        while (freq.size() > k) {
            int value = nums[left];
            int newCount = freq.get(value) - 1;

            if (newCount == 0) {
                freq.remove(value);
            } else {
                freq.put(value, newCount);
            }

            left++;
        }

        answer = Math.max(answer, right - left + 1);
    }

    return answer;
}
```

### String version

``` java
static int longestSubstringAtMostKDistinct(String s, int k) {
    if (k < 0) return 0;

    Map<Character, Integer> freq = new HashMap<>();
    int left = 0;
    int answer = 0;

    for (int right = 0; right < s.length(); right++) {
        char ch = s.charAt(right);
        freq.put(ch, freq.getOrDefault(ch, 0) + 1);

        while (freq.size() > k) {
            char remove = s.charAt(left++);
            int count = freq.get(remove) - 1;

            if (count == 0) freq.remove(remove);
            else freq.put(remove, count);
        }

        answer = Math.max(answer, right - left + 1);
    }

    return answer;
}
```

**Complexity:** Expected `O(n)` time, `O(k)` space.

------------------------------------------------------------------------

## 6. Exactly K distinct elements

A common trick:

``` text
exactlyK(k) = atMostK(k) - atMostK(k - 1)
```

Why? The number of windows with at most `k` distinct values includes
those with at most `k - 1` distinct values. Subtracting leaves windows
with exactly `k`.

### Count subarrays with exactly K distinct integers

``` java
static long subarraysWithKDistinct(int[] nums, int k) {
    if (k <= 0) return 0;
    return atMostKDistinct(nums, k) - atMostKDistinct(nums, k - 1);
}

static long atMostKDistinct(int[] nums, int k) {
    if (k < 0) return 0;

    Map<Integer, Integer> freq = new HashMap<>();
    int left = 0;
    long count = 0;

    for (int right = 0; right < nums.length; right++) {
        freq.put(nums[right], freq.getOrDefault(nums[right], 0) + 1);

        while (freq.size() > k) {
            int value = nums[left++];
            int newCount = freq.get(value) - 1;

            if (newCount == 0) freq.remove(value);
            else freq.put(value, newCount);
        }

        // Every suffix ending at right and starting from left ... right is valid.
        count += right - left + 1L;
    }

    return count;
}
```

**Complexity:** Expected `O(n)` time, `O(k)` space.

### General rule

The `atMost(k) - atMost(k - 1)` technique works when the desired count
can be expressed as a difference between two nested at-most counts.

------------------------------------------------------------------------

## 7. Frequency-map window

**Use when:** The window needs to track frequencies of values or
characters.

``` java
static void frequencyWindow(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    int left = 0;

    for (int right = 0; right < s.length(); right++) {
        char added = s.charAt(right);
        freq.put(added, freq.getOrDefault(added, 0) + 1);

        while (/* frequency/window constraint is violated */) {
            char removed = s.charAt(left++);
            int count = freq.get(removed) - 1;

            if (count == 0) {
                freq.remove(removed);
            } else {
                freq.put(removed, count);
            }
        }

        // Process the valid window.
    }
}
```

### Frequency array for lowercase English letters

When the input is restricted to `'a'` through `'z'`, an array is usually
simpler and faster than a `HashMap`.

``` java
int[] freq = new int[26];

freq[s.charAt(right) - 'a']++;

freq[s.charAt(left) - 'a']--;
```

For uppercase English letters, use `ch - 'A'`. For arbitrary Unicode
text, use a suitable map rather than assuming 26 characters.

------------------------------------------------------------------------

## 8. Anagram / permutation matching

**Use when:** Find fixed-length substrings whose character frequencies
match a pattern.

### Find all anagrams of `p` in `s`

``` java
static List<Integer> findAnagrams(String s, String p) {
    List<Integer> answer = new ArrayList<>();

    int n = s.length();
    int m = p.length();

    if (m > n) return answer;

    int[] need = new int[26];
    int[] window = new int[26];

    for (int i = 0; i < m; i++) {
        need[p.charAt(i) - 'a']++;
    }

    for (int right = 0; right < n; right++) {
        window[s.charAt(right) - 'a']++;

        if (right >= m) {
            window[s.charAt(right - m) - 'a']--;
        }

        if (right >= m - 1 && Arrays.equals(need, window)) {
            answer.add(right - m + 1);
        }
    }

    return answer;
}
```

**Complexity:** `O(26 × n)` = `O(n)` for a fixed alphabet; `O(26)` extra
space.

### Permutation in string

``` java
static boolean checkInclusion(String p, String s) {
    int m = p.length();
    int n = s.length();

    if (m > n) return false;

    int[] need = new int[26];
    int[] window = new int[26];

    for (int i = 0; i < m; i++) {
        need[p.charAt(i) - 'a']++;
    }

    for (int right = 0; right < n; right++) {
        window[s.charAt(right) - 'a']++;

        if (right >= m) {
            window[s.charAt(right - m) - 'a']--;
        }

        if (right >= m - 1 && Arrays.equals(need, window)) {
            return true;
        }
    }

    return false;
}
```

------------------------------------------------------------------------

## 9. Minimum window substring

**Use when:** Find the shortest substring containing all required
characters, including required multiplicities.

``` java
static String minWindow(String s, String t) {
    if (t.length() == 0 || s.length() < t.length()) return "";

    Map<Character, Integer> need = new HashMap<>();

    for (char ch : t.toCharArray()) {
        need.put(ch, need.getOrDefault(ch, 0) + 1);
    }

    int required = need.size();
    int formed = 0;

    Map<Character, Integer> window = new HashMap<>();

    int left = 0;
    int bestStart = 0;
    int bestLength = Integer.MAX_VALUE;

    for (int right = 0; right < s.length(); right++) {
        char added = s.charAt(right);
        window.put(added, window.getOrDefault(added, 0) + 1);

        if (need.containsKey(added)
                && window.get(added).intValue() == need.get(added).intValue()) {
            formed++;
        }

        while (formed == required) {
            int length = right - left + 1;

            if (length < bestLength) {
                bestLength = length;
                bestStart = left;
            }

            char removed = s.charAt(left++);
            window.put(removed, window.get(removed) - 1);

            if (need.containsKey(removed)
                    && window.get(removed) < need.get(removed)) {
                formed--;
            }
        }
    }

    return bestLength == Integer.MAX_VALUE
            ? ""
            : s.substring(bestStart, bestStart + bestLength);
}
```

**Complexity:** Expected `O(n + m)` time and `O(n + m)` worst-case
auxiliary space with maps.

------------------------------------------------------------------------

## 10. Counting subarrays using at-most subtraction

For some problems, count subarrays with exactly `k` occurrences of a
property.

The general identity is:

``` text
exactlyK = atMostK - atMost(K - 1)
```

### Example: binary subarrays with sum exactly `goal`

This version assumes all values are `0` or `1`.

``` java
static long numSubarraysWithSum(int[] nums, int goal) {
    if (goal < 0) return 0;
    return atMostSum(nums, goal) - atMostSum(nums, goal - 1);
}

static long atMostSum(int[] nums, int goal) {
    if (goal < 0) return 0;

    int left = 0;
    long sum = 0;
    long count = 0;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];

        while (sum > goal) {
            sum -= nums[left++];
        }

        count += right - left + 1L;
    }

    return count;
}
```

### Example: exactly K odd numbers

``` java
static int numberOfSubarrays(int[] nums, int k) {
    return atMostKOdds(nums, k) - atMostKOdds(nums, k - 1);
}

static int atMostKOdds(int[] nums, int k) {
    if (k < 0) return 0;

    int left = 0;
    int oddCount = 0;
    int answer = 0;

    for (int right = 0; right < nums.length; right++) {
        if ((nums[right] & 1) == 1) oddCount++;

        while (oddCount > k) {
            if ((nums[left] & 1) == 1) oddCount--;
            left++;
        }

        answer += right - left + 1;
    }

    return answer;
}
```

**Important:** These standard at-most sliding-window counting templates
rely on a monotonic validity condition. Do not blindly use them for
arbitrary negative values.

------------------------------------------------------------------------

## 11. Sliding window maximum/minimum using deque

**Use when:** Find the maximum or minimum in every window of size `k`.

A monotonic deque stores **indices**, not just values. Remove indices
that have left the window, then remove dominated values from the back.

### Sliding window maximum

``` java
static int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;

    if (k <= 0 || k > n) return new int[0];

    int[] answer = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>();
    int index = 0;

    for (int right = 0; right < n; right++) {
        // Remove indices outside the current window.
        while (!deque.isEmpty() && deque.peekFirst() <= right - k) {
            deque.pollFirst();
        }

        // Remove smaller or equal values from the back.
        while (!deque.isEmpty()
                && nums[deque.peekLast()] <= nums[right]) {
            deque.pollLast();
        }

        deque.offerLast(right);

        if (right >= k - 1) {
            answer[index++] = nums[deque.peekFirst()];
        }
    }

    return answer;
}
```

### Sliding window minimum

Change the back-removal comparison:

``` java
while (!deque.isEmpty()
        && nums[deque.peekLast()] >= nums[right]) {
    deque.pollLast();
}
```

**Complexity:** `O(n)` time, `O(k)` space. Each index enters and leaves
the deque at most once.

------------------------------------------------------------------------

## 12. Binary search + sliding window

**Use when:** The answer is a numeric value, and you can check whether a
candidate answer is feasible.

General structure:

``` java
static int solve(int[] nums) {
    int low = /* minimum possible answer */;
    int high = /* maximum possible answer */;
    int answer = high;

    while (low <= high) {
        int mid = low + (high - low) / 2;

        if (isPossible(nums, mid)) {
            answer = mid;
            high = mid - 1; // Search for a smaller feasible answer.
        } else {
            low = mid + 1;
        }
    }

    return answer;
}

static boolean isPossible(int[] nums, int candidate) {
    // Use a sliding window to check feasibility.
    return false;
}
```

For a maximum feasible answer, reverse the binary-search direction:

``` java
if (isPossible(nums, mid)) {
    answer = mid;
    low = mid + 1;
} else {
    high = mid - 1;
}
```

**Key requirement:** Feasibility must be monotonic. For example, if a
candidate window limit works, larger limits may also work (or vice
versa).

------------------------------------------------------------------------

## 13. Circular sliding window

**Use when:** The array or string wraps around at the end.

One common approach is to iterate over a virtual doubled array using
modulo.

### Maximum circular sum of a window of size `k`

``` java
static long maxCircularWindowSum(int[] nums, int k) {
    int n = nums.length;

    if (n == 0 || k <= 0 || k > n) {
        throw new IllegalArgumentException("Invalid input");
    }

    long sum = 0;

    for (int i = 0; i < k; i++) {
        sum += nums[i];
    }

    long answer = sum;

    for (int start = 1; start < n; start++) {
        sum -= nums[start - 1];
        sum += nums[(start + k - 1) % n];

        answer = Math.max(answer, sum);
    }

    return answer;
}
```

**Complexity:** `O(n)` time, `O(1)` extra space.

------------------------------------------------------------------------

## 14. Two arrays / concatenated words

Some string-window problems use words of equal length rather than
individual characters. Maintain word frequencies and move in steps of
the word length.

### Substring with concatenation of all words

``` java
static List<Integer> findSubstring(String s, String[] words) {
    List<Integer> answer = new ArrayList<>();

    if (words.length == 0 || words[0].length() == 0) {
        return answer;
    }

    int wordLen = words[0].length();
    int wordCount = words.length;
    int totalLen = wordLen * wordCount;

    if (s.length() < totalLen) return answer;

    Map<String, Integer> need = new HashMap<>();

    for (String word : words) {
        need.put(word, need.getOrDefault(word, 0) + 1);
    }

    for (int offset = 0; offset < wordLen; offset++) {
        Map<String, Integer> window = new HashMap<>();
        int left = offset;
        int usedWords = 0;

        for (int right = offset; right + wordLen <= s.length(); right += wordLen) {
            String word = s.substring(right, right + wordLen);

            if (!need.containsKey(word)) {
                window.clear();
                usedWords = 0;
                left = right + wordLen;
                continue;
            }

            window.put(word, window.getOrDefault(word, 0) + 1);
            usedWords++;

            while (window.get(word) > need.get(word)) {
                String remove = s.substring(left, left + wordLen);
                window.put(remove, window.get(remove) - 1);
                usedWords--;
                left += wordLen;
            }

            if (usedWords == wordCount) {
                answer.add(left);

                String remove = s.substring(left, left + wordLen);
                window.put(remove, window.get(remove) - 1);
                usedWords--;
                left += wordLen;
            }
        }
    }

    return answer;
}
```

**Complexity:** Depends on substring and hash-map costs; the scan
considers each word-aligned position once, with word extraction and
hashing overhead.

------------------------------------------------------------------------

## 15. Common pitfalls

### 1. Using `if` instead of `while`

Incorrect for a variable-size window when multiple removals may be
needed:

``` java
if (sum > k) {
    sum -= nums[left++];
}
```

Usually correct:

``` java
while (sum > k) {
    sum -= nums[left++];
}
```

### 2. Updating the answer at the wrong time

-   **Longest valid window:** Update after shrinking until valid.
-   **Shortest valid window:** Update while the window is valid, before
    shrinking.
-   **Fixed window:** Update only after the window reaches size `k`.

### 3. Forgetting to remove zero-frequency keys

For distinct-element problems, `map.size()` is meaningful only if keys
with frequency zero are removed.

### 4. Using sliding window with negative numbers without proof

A negative number can reduce a sum when added, so the usual sum-based
expand/shrink logic may fail.

### 5. Off-by-one errors

Window length is:

``` java
right - left + 1
```

A window of size `k` is full when:

``` java
right - left + 1 == k
```

### 6. Integer overflow

Use `long` for sums and counts when constraints may exceed the `int`
range.

### 7. Forgetting edge cases

Test: - Empty input - `k == 0` - `k > n` - One element - All equal
values - Duplicates - All zeros - No valid window - Large values -
Negative values, if allowed by the problem

------------------------------------------------------------------------

## 16. Pattern selection cheat sheet

  -----------------------------------------------------------------------
  Problem asks for...                 Start with this pattern
  ----------------------------------- -----------------------------------
  Exactly `k` consecutive elements    Fixed-size window

  Maximum length satisfying a         Longest valid window
  condition                           

  Minimum length satisfying a         Shortest valid window
  condition                           

  At most `k` distinct values         HashMap + variable window

  Exactly `k` distinct values         `atMost(k) - atMost(k-1)`

  Anagrams / permutations             Fixed-size frequency window

  Minimum substring containing a      Frequency map + formed/required
  target                              

  Maximum in every window             Monotonic deque

  Exactly `k` occurrences of a        At-most subtraction, if applicable
  property                            

  Circular fixed-size range           Modulo-based fixed window

  Candidate answer with monotonic     Binary search + window check
  feasibility                         

  Equal-length word concatenation     Word-sized window
  -----------------------------------------------------------------------

## Recommended practice order

1.  Maximum Sum Subarray of Size K
2.  Maximum Average Subarray I
3.  Maximum Number of Vowels in a Substring of Given Length
4.  Minimum Size Subarray Sum
5.  Longest Substring Without Repeating Characters
6.  Max Consecutive Ones III
7.  Fruit Into Baskets
8.  Permutation in String
9.  Find All Anagrams in a String
10. Longest Repeating Character Replacement
11. Subarrays with K Different Integers
12. Minimum Window Substring
13. Sliding Window Maximum
14. Binary Subarrays With Sum
15. Shortest Subarray with Sum at Least K

## Java imports

Use these imports for most templates:

``` java
import java.util.*;
```

## Final checklist before coding

-   [ ] Is the range contiguous?
-   [ ] Is the window fixed-size or variable-size?
-   [ ] What state must I maintain: sum, count, frequency, distinct
    values, or deque?
-   [ ] What makes the window invalid?
-   [ ] Should I update the answer before or after shrinking?
-   [ ] Can the input contain negative numbers?
-   [ ] Can sums or counts overflow `int`?
-   [ ] Have I tested boundary cases?

------------------------------------------------------------------------

**GitHub tip:** Save this file as `sliding-window-templates-java.md` in
your DSA repository. You can link it from your main `README.md` under
your Arrays / Two Pointers / Sliding Window section.
