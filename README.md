
---

# UAV Strategic Deconfliction System


---

## **1. Overview**

This project implements a **scalable, modular, and explainable conflict detection and rerouting system** for drones operating in shared airspace.
The solution uses a **spatio-temporal occupancy grid** to efficiently detect conflicts between thousands of drones and applies a simple explainable (XAI) rerouting heuristic to resolve detected conflicts.

---

## **2. Problem Statement**

**Goal:**
Given the planned path of a primary drone and a dataset of other drone flight schedules,

* Detect any spatial-temporal conflicts (risk of collision in 3D space at a specific time).
* Explain the location, time, and cause of each conflict.
* Propose and explain a conflict-free alternative path where possible.
* Support large-scale simulation (10,000+ drones) and clear visualization.

---

## **3. Core Approach**

### 3.1 Spatio-Temporal Grid Occupancy Model

What is the Grid Occupancy Model?
The grid occupancy model discretizes continuous airspace and time into a 4D grid: (x, y, z, t).

Each grid cell represents a small volume of 3D space at a specific moment in time.

Each drone’s path is a set of waypoints (positions and timestamps).

| Feature        | Naive (pairwise)     | Grid Occupancy           |
| -------------- | -------------------- | ------------------------ |
| Performance    | O(n²)                | O(n) build, O(1) lookup  |
| Real-world use | Impractical at scale | Used in industry/ATC     |
| Buffer         | Manual, error-prone  | Built-in (cell size)     |
| Distributed?   | Hard                 | Easy to shard            |
| Uncertainty    | Complex              | Cell inflation/weight    |
| Visualization  | Hard                 | Built-in heatmap/3D plot |
| Explainable?   | Yes, but verbose     | Yes, and simple          |


Instead of checking every waypoint against every other (which is O(n²) and infeasible at scale), we map each waypoint to a grid cell.
The grid cell acts as a “bucket”: if two (or more) drones are mapped to the same cell at the same time, a conflict is detected.

* The airspace is discretized into a **4D grid**: (x, y, z, t) where each axis represents a position in space and time.
* **Waypoints**: Each drone’s path consists of a list of waypoints `(x, y, z, t)`.
* **Grid Cell Mapping**: Each waypoint is mapped to a grid cell using:

  ```
  cell = (int(x // cell_size), int(y // cell_size), int(z // cell_size), int(time // time_step))
  ```
* **Occupancy Grid**: A hash map (dictionary) is used to record which drones occupy each grid cell at each time step.
* **Conflict Detection**:

  * For each waypoint of the primary drone, the system checks if another drone already occupies the same grid cell at the same time (excluding itself).
  * If so, it records the conflict (drone id, location, time).

### 3.2 Rerouting Logic (Explainable XAI)

* If a conflict is found, the system attempts a **simple reroute**:

  * Shift only the conflicting waypoints along the x-axis (by `cell_size`), up to a configurable max number of attempts.
  * After each shift, re-check for conflicts.
  * Rerouting stops when the path is conflict-free or when maximum attempts are exhausted.
* **XAI**:

  * The system outputs an explanation: which waypoints were shifted, how many times, and the final result (conflicts resolved or not).

---

## **4. Code Structure**

* `Waypoint` class: 3D position and time.

* `Drone` class: Unique drone ID and a list of waypoints.

* `OccupancyGrid` class:

  * Builds and queries the spatio-temporal grid.

* `DroneDeconflictor` class:

  * Detects conflicts for a primary drone.
  * Attempts XAI rerouting and provides detailed explanations.


## **5. Example: Grid Occupancy Conflict Detection**

**Waypoints mapping example:**
Suppose `cell_size = 1.0`,

* Drone A at `(2.3, 3.7, 5.9)` at `t=4` maps to cell `(2, 3, 5, 4)`
* If Drone B has a waypoint at `(2.1, 3.9, 6.0)` at `t=4`, it also maps to `(2, 3, 6, 4)`
* Conflict is detected **only if all grid cell indices (x, y, z, t) match**.

**This discretization ensures efficient O(1) lookup for conflicts**,
and supports high scalability (large airspace and drone counts).


---

## **6. Technical Highlights**

* **Scalable:**

  * Occupancy grid allows constant-time lookups and linear construction time,
    making it practical for 10,000+ drones and real-time updates.
* **Modular:**

  * Each component (loading, grid, detection, reroute) is cleanly separated for maintenance and future upgrades (e.g., A\* pathfinding, cloud API integration).
* **Explainable:**

  * Conflict and reroute logic is transparent and auditable; every decision is reported.
* **Efficient I/O:**

  * Data loaded from CSV for flexible, real-world input.
* **Visualization Ready:**

  * Supports both 2D (x/y) and 3D (x/y/z) plots; integrates with Matplotlib and Plotly for interactive results.

---


## **7. Scalability and Extensibility**

* **Distributed Deployment:**

  * Airspace can be divided into sectors, with each running a grid server for local drones.
  * Real-time updates are supported via message passing or cloud APIs.
* **Enhancements:**

  * Advanced path planners (e.g., A\* in grid), dynamic cost layers (wind/weather), probabilistic occupancy for uncertainty.
  * REST API for integration with operational drone platforms (like FlytBase).

---

## **8. Limitations and Further Work**

* Current reroute logic uses a simple shift (along x).
  Can be upgraded to multi-axis search, smarter algorithms, or full path re-planning.
* Probabilistic (uncertainty-aware) grid occupancy can be added for more robust, real-world scenarios.

---

## **9. References**


---

## **10. Author**

**\Aditi Dekhane**
For FlytBase Robotics Assignment 2025 – AI/Robotics Engineer Application

---

**End of README**
