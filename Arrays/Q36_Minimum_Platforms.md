# Minimum Platforms for Train Schedule

## 1. Problem Statement

Given arrival and departure times of all trains that reach a railway station, find the minimum number of platforms required for the railway station so that no train waits.

We are given two arrays `arr[]` (arrival times) and `dep[]` (departure times) of size `n`.

* Time values are represented in 24-hour format as integers (e.g., `900` for 9:00 AM, `1230` for 12:30 PM).
* If a train arrives and another departs at the same time, the platform cannot be shared immediately; the departing train must leave before the arriving train can use the platform.

### Input
* `arr[]`: An array of arrival times of size `n`.
* `dep[]`: An array of departure times of size `n`.
* `n`: The number of trains ($1 \le n \le 50000$).

### Output
* Returns the minimum number of platforms required.

### Constraints
* `0 <= arr[i] <= dep[i] <= 2359`

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Are arrival and departure times sorted?**
   * *Answer*: No, they are not guaranteed to be sorted.
2. **Can departure time be less than arrival time (e.g., train arrives at 11:00 PM and leaves at 1:00 AM)?**
   * *Answer*: In standard formulations of this problem, we assume all trains arrive and depart within the same day, so `arr[i] <= dep[i]`. If overnight trains exist, we must handle date transitions.
3. **What happens if a train departs at 10:00 and another arrives at 10:00?**
   * *Answer*: As per standard rules, if arrival and departure times are identical, we need a platform. We assume departure must be completed first, or we count them as overlapping. (Here we assume arrival at time $X$ and departure at time $X$ requires a platform. If they don't overlap, check the problem description: "departure train leaves first" vs "arrival waits"). Let's assume they overlap to be safe, or clarify this with the interviewer.

---

## 3. Thinking Process

### First Instinct (Brute Force)
* For each train, count how many other trains overlap with its schedule.
* Two trains $i$ and $j$ overlap if:
  `arr[i] <= arr[j] <= dep[i]` OR `arr[j] <= arr[i] <= dep[j]`.
* The maximum number of overlaps found for any train is the minimum number of platforms needed.
* This takes $O(n^2)$ time since we check every pair of trains.

### Better Instinct (Sweep Line Simulation / Chronological Events)
* We can think of the train arrivals and departures as events on a timeline.
* An arrival at time $T$ is an event that increases the platform count by 1.
* A departure at time $T$ is an event that decreases the platform count by 1.
* We can collect all events, sort them chronologically, and process them:
  * Sort events. If two events have the same time:
    * If one is arrival and one is departure, we must process arrival first (or departure first, depending on the rule).
  * Maintain `current_platforms = 0` and `max_platforms = 0`.
  * For each event:
    * If event is arrival: `current_platforms++`.
    * If event is departure: `current_platforms--`.
    * Update `max_platforms = max(max_platforms, current_platforms)`.
* This takes $O(n \log n)$ time due to sorting and $O(n)$ space to store the events.

### Optimal Insight (Two-Pointer Sweeping on Sorted Arrays)
* Can we do the sweep-line simulation in $O(1)$ extra space?
* Yes, by sorting the `arr` and `dep` arrays **independently**.
* **Crucial Realization**: We don't need to know which specific train is arriving or departing. We only care about the sequence of arrivals and departures as values.
* If we sort `arr` and `dep` independently:
  * Let `i = 0` (pointer for `arr`) and `j = 0` (pointer for `dep`).
  * While `i < n` and `j < n`:
    * If `arr[i] <= dep[j]`: This means a train has arrived before the earliest departing train has left. We need a platform.
      * `platform_needed++`
      * `i++` (Move to next arrival)
    * If `arr[i] > dep[j]`: A train has departed before the next train arrives. A platform becomes free.
      * `platform_needed--`
      * `j++` (Move to next departure)
    * Keep track of the maximum platform count seen at any point.
* This takes $O(n \log n)$ time for sorting and $O(1)$ auxiliary space (if sorting in-place).

---

## 4. Pattern Recognition

* **Topic**: Greedy / Interval Scheduling / Sweep Line.
* **Pattern**: Two Pointers on Sorted Events.
* **Why this works**: Sorting the events chronologically allows us to simulate the timeline of the station. By tracking the count of active intervals (trains in the station), we find the peak concurrency.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `max_platforms = 1`.
2. For each train `i` from `0` to `n-1`:
   * Set `overlap = 1`.
   * For each train `j` from `0` to `n-1`:
     * If `i != j`:
       * If `arr[i] >= arr[j]` and `dep[j] >= arr[i]` (train `j` is at the station when train `i` arrives), increment `overlap`.
   * Update `max_platforms = max(max_platforms, overlap)`.
3. Return `max_platforms`.

### Complexity Analysis
* **Time Complexity**: $O(n^2)$ due to nested comparisons.
* **Space Complexity**: $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    int findPlatform(const std::vector<int>& arr, const std::vector<int>& dep) {
        int n = arr.size();
        int max_platforms = 0;
        
        for (int i = 0; i < n; ++i) {
            int overlap = 0;
            for (int j = 0; j < n; ++j) {
                // Check if train j overlaps with arrival of train i
                if (arr[j] <= arr[i] && dep[j] >= arr[i]) {
                    overlap++;
                }
            }
            max_platforms = std::max(max_platforms, overlap);
        }
        
        return max_platforms;
    }
};
```

---

## 6. Better Solution

*Note: Since the optimal solution is the standard two-pointer sweep on sorted arrays, we can consider the "Better Solution" to be the Event-based sorting approach using a custom struct. It is easier to conceptualize but uses $O(n)$ extra space to store the events.*

### Algorithm (Event Sorting)
1. Represent each train's arrival and departure as an event:
   * Arrival: `{time, 'A'}`
   * Departure: `{time, 'D'}`
2. Collect all $2n$ events in a list and sort them.
   * If times are equal, put departure ('D') before arrival ('A') if departure frees a platform immediately, OR arrival first if they overlap.
3. Iterate through the sorted list, updating the platform counter and tracking the maximum.

### Complexity Analysis
* **Time Complexity**: $O(n \log n)$ due to sorting $2n$ events.
* **Space Complexity**: $O(n)$ to store the events.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

struct Event {
    int time;
    char type;
    
    // Sort chronologically. If times are equal, departure comes first
    bool operator<(const Event& other) const {
        if (time == other.time) {
            return type == 'D'; // 'D' comes before 'A'
        }
        return time < other.time;
    }
};

class SolutionBetter {
public:
    int findPlatform(const std::vector<int>& arr, const std::vector<int>& dep) {
        std::vector<Event> events;
        int n = arr.size();
        
        for (int i = 0; i < n; ++i) {
            events.push_back({arr[i], 'A'});
            events.push_back({dep[i], 'D'});
        }
        
        std::sort(events.begin(), events.end());
        
        int current_platforms = 0;
        int max_platforms = 0;
        
        for (const auto& event : events) {
            if (event.type == 'A') {
                current_platforms++;
            } else {
                current_platforms--;
            }
            max_platforms = std::max(max_platforms, current_platforms);
        }
        
        return max_platforms;
    }
};
```

---

## 7. Optimal Solution

### Algorithm
1. Make a copy of `arr` and `dep` (or sort the input arrays directly if modification is permitted).
2. Sort both arrays:
   * `std::sort(arr.begin(), arr.end())`
   * `std::sort(dep.begin(), dep.end())`
3. Initialize pointers: `i = 0` (for arrivals), `j = 0` (for departures).
4. Initialize counters: `platforms_needed = 0`, `max_platforms = 0`.
5. Loop while `i < n` and `j < n`:
   * If `arr[i] <= dep[j]`:
     * A train is arriving before the earliest departure. We need an extra platform.
     * `platforms_needed++`
     * `i++`
   * Else:
     * A train departs, freeing a platform.
     * `platforms_needed--`
     * `j++`
   * Update `max_platforms = max(max_platforms, platforms_needed)`.
6. Return `max_platforms`.

### Why this is Optimal
* Sorting takes $O(n \log n)$ time.
* The two-pointer sweep takes $O(n)$ time.
* No extra data structures are used, so space complexity is $O(1)$ auxiliary.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    int findPlatform(std::vector<int>& arr, std::vector<int>& dep) {
        int n = arr.size();
        
        // Sort arrival and departure times independently
        std::sort(arr.begin(), arr.end());
        std::sort(dep.begin(), dep.end());
        
        int platforms_needed = 0;
        int max_platforms = 0;
        
        int i = 0; // Pointer for arrivals
        int j = 0; // Pointer for departures
        
        while (i < n && j < n) {
            // If next event is an arrival
            if (arr[i] <= dep[j]) {
                platforms_needed++;
                i++;
            }
            // If next event is a departure
            else {
                platforms_needed--;
                j++;
            }
            // Update the peak number of platforms
            max_platforms = std::max(max_platforms, platforms_needed);
        }
        
        return max_platforms;
    }
};
```

---

## 8. Code Walkthrough

* **Line 9-10**: Sort `arr` and `dep` independently. Sorting them is correct because we only care about the times of events, not which train is which.
* **Line 12-13**: Initialize `platforms_needed` (current occupancy) and `max_platforms` (peak occupancy).
* **Line 15-16**: Pointers `i` and `j` start at `0`.
* **Line 18**: `while (i < n && j < n)`
  We scan the arrivals and departures. Once `i == n`, no more trains will arrive, so the platform count can only decrease from there. We can safely terminate the loop.
* **Line 20-23**: `if (arr[i] <= dep[j])`
  If a train arrives before or at the same time the current earliest departure occurs, we must allocate a platform. Advance arrival pointer `i`.
* **Line 25-28**: `else`
  If the next departure time `dep[j]` is strictly less than the next arrival time `arr[i]`, a platform is cleared. Advance departure pointer `j`.
* **Line 30**: Record the peak number of platforms needed at any point in time.

---

## 9. Dry Run

Input:
`arr = [900, 940, 950, 1100, 1500, 1800]`
`dep = [910, 1200, 1120, 1130, 1900, 2000]`

Sorted inputs:
`arr = [900, 940, 950, 1100, 1500, 1800]`
`dep = [910, 1120, 1130, 1200, 1900, 2000]`

| Step | `i` | `j` | `arr[i]` | `dep[j]` | Comparison | Action | `platforms_needed` | `max_platforms` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Start** | 0 | 0 | 900 | 910 | - | - | 0 | 0 |
| **1** | 0 | 0 | 900 | 910 | `900 <= 910` | `platforms++`, `i++` | 1 | 1 |
| **2** | 1 | 0 | 940 | 910 | `940 > 910` | `platforms--`, `j++` | 0 | 1 |
| **3** | 1 | 1 | 940 | 1120 | `940 <= 1120` | `platforms++`, `i++` | 1 | 1 |
| **4** | 2 | 1 | 950 | 1120 | `950 <= 1120` | `platforms++`, `i++` | 2 | 2 |
| **5** | 3 | 1 | 1100 | 1120 | `1100 <= 1120` | `platforms++`, `i++` | 3 | 3 |
| **6** | 4 | 1 | 1500 | 1120 | `1500 > 1120` | `platforms--`, `j++` | 2 | 3 |
| **7** | 4 | 2 | 1500 | 1130 | `1500 > 1130` | `platforms--`, `j++` | 1 | 3 |
| **8** | 4 | 3 | 1500 | 1200 | `1500 > 1200` | `platforms--`, `j++` | 0 | 3 |
| **9** | 4 | 4 | 1500 | 1900 | `1500 <= 1900` | `platforms++`, `i++` | 1 | 3 |
| **10** | 5 | 4 | 1800 | 1900 | `1800 <= 1900` | `platforms++`, `i++` | 2 | 3 |

Loop terminates because `i = 6` ($i == n$).
Result: `3`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ | Checks all pairs |
| Event Sorting | $O(n \log n)$ | $O(n)$ | Allocates space for structures |
| **Two Pointers (Optimal)** | **$O(n \log n)$** | **$O(1)$ auxiliary** | **Sorts in place, linear sweep** |

---

## 11. Common Mistakes

1. **Thinking we can't sort the arrays independently**:
   * A common mistake is believing that sorting `arr` and `dep` separately breaks the pairing of arrival and departure for a specific train.
   * *Clarification*: We do not care which specific train arrives or departs; we only care about the volume of trains currently occupied at any time slot.
2. **Incorrect handling of equal arrival and departure times**:
   * If a train arrives at `1100` and another departs at `1100`, the comparison `arr[i] <= dep[j]` is true, which counts them as overlapping. This is the correct behavior if the platform cannot be immediately re-allocated.

---

## 12. Edge Cases

| Edge Case | Example | Why it matters | Solution Behavior |
| :--- | :--- | :--- | :--- |
| Only 1 train | `arr = [900]`, `dep = [1000]` | Minimum inputs. | Returns `1` (Correct) |
| Multiple trains arrive at the same time | `arr = [900, 900]`, `dep = [930, 940]` | Peak occupancy at arrival. | Returns `2` (Correct) |
| No overlaps | `arr = [900, 1000]`, `dep = [930, 1030]` | Successive non-overlapping intervals. | Returns `1` (Correct) |
| Complete overlapping | `arr = [900, 910]`, `dep = [1000, 950]` | Nested intervals. | Returns `2` (Correct) |

---

## 13. Interview Explanation

"To find the minimum number of platforms required, we can model this as finding the maximum number of overlapping intervals at any point in time. 

If we sort the arrival and departure times independently, we can sweep through the timeline chronologically using two pointers: one pointer for arrivals and one for departures.
We compare the current arrival time with the current departure time:
* If the arrival time is less than or equal to the departure time, it means a train is arriving before a platform becomes free. We increment our platform count and advance the arrival pointer.
* Otherwise, a train has departed, freeing a platform. We decrement our platform count and advance the departure pointer.
Throughout this process, we keep track of the peak platform count. Sorting takes $O(n \log n)$ time, and the linear sweep takes $O(n)$ time, resulting in a total time complexity of $O(n \log n)$ and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **How do we return the actual schedules/assignments of trains to specific platforms?**
   * *Answer*: If we need to assign specific trains to platform numbers (e.g., Train A on Platform 1, Train B on Platform 2), we can sort intervals by start time and use a min-heap to keep track of the end times of active platforms.
2. **What if the schedule is given as strings like "09:00"?**
   * *Answer*: Convert the time strings to minutes from midnight (e.g., `"09:00"` $\to 9 \times 60 + 0 = 540$ minutes) to simplify numeric comparisons.

---

## 15. Variations

1. **Merge Intervals**: Merging overlapping intervals.
2. **Meeting Rooms II**: Find minimum meeting rooms required (identical problem).

---

## 16. Revision Notes

* **Concept**: Chronological Event Sweeping.
* **Key Trick**: Sort `arr` and `dep` independently.
* **Loop logic**: `arr[i] <= dep[j] ? platforms++, i++ : platforms--, j++`.
* **Complexity**: $O(n \log n)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// O(n log n) Time, O(1) Space. Independent sort and two pointers.
int findPlatform(vector<int>& arr, vector<int>& dep) {
    sort(arr.begin(), arr.end());
    sort(dep.begin(), dep.end());
    int i = 0, j = 0, n = arr.size();
    int active = 0, max_platforms = 0;
    while (i < n && j < n) {
        if (arr[i] <= dep[j]) {
            active++;
            i++;
        } else {
            active--;
            j++;
        }
        max_platforms = max(max_platforms, active);
    }
    return max_platforms;
}
```
