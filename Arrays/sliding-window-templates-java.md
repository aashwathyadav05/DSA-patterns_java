
# Sliding Window Templates

## 1. Fixed Size Sliding Window

**Use when:** The window size `k` is fixed. Initialize the first window, then shift it by removing the outgoing element and adding the incoming element.

```java
static void fixedWindow(int[] arr, int k) {
    int n = arr.length;

    if (k <= 0 || k > n) return;

    int windowSum = 0;

    // Step 1: Initialize the first window of size k
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }

    // Process the first window
    int ans = windowSum;

    // Step 2: Slide the window
    for (int i = k; i < n; i++) {

        // Remove outgoing element
        windowSum -= arr[i - k];

        // Add incoming element
        windowSum += arr[i];

        // Process current window
        ans = Math.max(ans, windowSum);
    }

    System.out.println(ans);
}
```

**Window movement:**

```text
[1, 2, 3] 4  5
 1 [2, 3, 4] 5
 1  2 [3, 4, 5]
```

**Time:** O(n)  
**Space:** O(1)

---

## 2. Variable Size Sliding Window – Maximum Length

**Use when:** Find the longest subarray/substring satisfying a given condition.

**Logic:**
1. Expand the window using `right`.
2. Add the new element to the window.
3. If the window becomes invalid, shrink using `left`.
4. Update the maximum length after the window becomes valid.

```java
static int maxLength(int[] arr, int k) {
    int left = 0;
    int ans = 0;

    for (int right = 0; right < arr.length; right++) {

        // Step 1: Expand window
        // Add arr[right] to window state

        // Step 2: Shrink while invalid
        while (/* window is invalid */) {

            // Remove arr[left] from window state
            left++;
        }

        // Step 3: Update maximum length
        ans = Math.max(ans, right - left + 1);
    }

    return ans;
}
```

**Example: Longest subarray with sum <= k**

(Assumes non-negative array elements.)

```java
static int maxLength(int[] arr, int k) {
    int left = 0;
    int ans = 0;
    long sum = 0;

    for (int right = 0; right < arr.length; right++) {

        sum += arr[right];

        while (sum > k) {
            sum -= arr[left];
            left++;
        }

        ans = Math.max(ans, right - left + 1);
    }

    return ans;
}
```

**Time:** O(n)  
**Space:** O(1)

---

## 3. Variable Size Sliding Window – Minimum Length

**Use when:** Find the shortest subarray/substring satisfying a given condition.

**Logic:**
1. Expand the window using `right`.
2. Add the new element.
3. While the window is valid, update the minimum length.
4. Shrink the window using `left` to find a smaller valid window.

```java
static int minLength(int[] arr, int k) {
    int left = 0;
    int ans = Integer.MAX_VALUE;

    for (int right = 0; right < arr.length; right++) {

        // Step 1: Expand window
        // Add arr[right] to window state

        // Step 2: Shrink while valid
        while (/* window is valid */) {

            // Update minimum length
            ans = Math.min(ans, right - left + 1);

            // Remove arr[left] from window state
            left++;
        }
    }

    return ans == Integer.MAX_VALUE ? 0 : ans;
}
```

**Example: Minimum subarray length with sum >= k**

(Assumes positive array elements.)

```java
static int minLength(int[] arr, int k) {
    int left = 0;
    int ans = Integer.MAX_VALUE;
    long sum = 0;

    for (int right = 0; right < arr.length; right++) {

        sum += arr[right];

        while (sum >= k) {

            ans = Math.min(ans, right - left + 1);

            sum -= arr[left];
            left++;
        }
    }

    return ans == Integer.MAX_VALUE ? 0 : ans;
}
```

**Time:** O(n)  
**Space:** O(1)

---

## Quick Revision

| Pattern | Window behavior | Answer update |
|---|---|---|
| Fixed Size | Initialize k, then shift | Every complete window |
| Variable – Maximum | Expand, shrink if invalid | After shrinking |
| Variable – Minimum | Expand, shrink while valid | Before shrinking |

### Remember

- Fixed window → `right - left + 1 == k`
- Maximum length → `while (invalid)` then update answer.
- Minimum length → `while (valid)` then update answer.
- Always use `while` when multiple shrinking steps may be required.
- Standard sum-based variable windows need non-negative/positive values for the usual monotonic shrinking logic.
