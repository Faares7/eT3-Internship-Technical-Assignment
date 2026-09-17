# Delivery Route Planner

---

## Reasoning & Technical Decisions

### 1. Explain your solution approach in your own words.
I used a greedy algorithmic approach. The program reads the delivery requests from an Excel file into lists. It dispatches one vehicle at a time, repeatedly scanning the remaining unassigned packages to find the best fit that keeps the total weight under 10.0kg. The "best fit" is determined by a strict hierarchy: 
1. Highest urgency (lowest priority number).
2. If priorities are tied, it selects packages heading to the exact same area as the first package loaded into the vehicle.
3. If still tied (or neither matches the area), it selects the lightest package to save vehicle space. 

Once a package is selected, its exact index is identified, and it is simultaneously popped from all lists to prevent it from being processed twice.

### 2. What was the most difficult part of the assignment?
Managing the dynamic shrinking of the lists during iteration. When a package is assigned to a trip, it must be removed from the primary queue. Removing items while looping over them easily leads to `IndexError` crashes or skips packages. The solution was to decouple the search from the removal: the program searches the list to find the `best_index`, stores that index, and only uses `.pop()` after the search loop completes. Incorporating the "Area" constraint without overcomplicating the search logic was also an interesting challenge, solved by locking in a `tripArea` variable the moment the first package is loaded.

### 3. Are there situations where your algorithm may not produce the best possible grouping? Explain.
Yes. Because it uses a greedy approach, it makes the best *local* choice at the exact moment of assignment without looking ahead at the broader picture. 
For example, if a vehicle has 3kg of space left, and there are two available Priority 2 packages weighing 2kg and 3kg, the algorithm's tie-breaker will pick the lighter 2kg package. This leaves 1kg of wasted space in the current vehicle and pushes the 3kg package to the next trip, which might result in an extra vehicle being dispatched overall. It prioritizes strict rule adherence over global weight optimization (like a Knapsack algorithm would).

### 4. If the input contained 1,000,000 delivery requests, what part of your solution might become slow or memory-intensive?
The current solution has a time complexity of roughly O(N^2). For every single package added to a trip, the code loops through the entire remaining unassigned list to find the next minimum priority. With 1,000,000 requests, this nested looping will become incredibly slow. Additionally, using `.pop()` on standard Python lists requires the computer to shift every subsequent item in the list one spot to the left in memory. Doing this hundreds of thousands of times will cause severe performance bottlenecks.

### 5. What would you improve if you had another day to work on the solution?
*   **Data Structures:** I would replace the separate parallel lists (`ID`, `Area`, `Priority`, `Weight`) with a single list of dictionaries or a dedicated `Package` class. This would eliminate the risk of the parallel lists becoming misaligned and make the code much cleaner.
*   **Sorting Upfront:** Instead of looping to find the minimum values every single time, I would sort the initial data primarily by Priority, then Area, then Weight using Python's `.sort(key=...)`. This would reduce the time complexity to O(N log N) and make the routing assignment nearly instantaneous, even for 1,000,000 rows.

---

##  Extension: Vehicle Utilization Metric
**Feature Added:** Fleet Efficiency Summary
At the end of the routing process, the program calculates and outputs the overall vehicle capacity utilization percentage (e.g., "Overall Fleet Utilization: 85.0%"). 

**Reasoning:** In a real-world logistics environment, routing isn't just about emptying the queue; it is highly dependent on unit economics and minimizing wasted space to save on fuel and fleet costs. This simple metric provides immediate feedback on how efficiently the grouping algorithm packed the dispatched vehicles against their maximum 10kg limit.
