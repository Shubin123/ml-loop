# Mini Library Loop — Route Planner

An advanced, browser-based, high-performance geospatial route planner specifically optimized for touring Little Free Libraries (LFLs) across Greater Victoria. By treating the route generation as a **Travelling Salesman Problem (TSP)**, this application leverages non-blocking asynchronous heuristics and hardware-accelerated GPGPU matrix computations to find near-optimal paths across up to 1,000+ coordinates directly inside the browser.

---

## 🚀 Technical Architecture Overview

The application is engineered completely client-side using vanilla HTML5, modern ES6+, and WebGL2. It avoids freezing the main UI thread during intense optimization steps via time-sliced, chunked event-loop cycles.

### 1. Distance Matrix Computation Engine
To build the routing cost matrix, the planner evaluates $O(N^2)$ coordinate pairs using two alternate distance configurations:
* **Direct Mode (Haversine Formula):** Computes exact great-circle paths over a spherical earth radius using the standard trigonometric formula:
    $$d = 2R \arcsin\left(\sqrt{\sin^2\left(\frac{\Delta\phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta\lambda}{2}\right)}\right)$$
* **Driving Mode (OSRM API):** Queries the public Open Source Routing Machine (OSRM) demo server using the `/table` and `/route` endpoints. For datasets where $N \le 150$, the algorithm optimizes directly against the true topological driving matrix. For larger datasets, it resolves the sequence via straight-line metrics and layers the real road polyline details over the final route topology.

### 2. GPGPU Acceleration Layer (WebGL2)
When processing dense spatial segments ($N \ge 40$), the engine binds an `RGBA32F` data texture loaded with raw coordinates converted to radians. A custom fragment shader calculates the pairwise Haversine grid in parallel across GPU execution cores.
* **Tiled Framework:** Includes automatic tiling boundaries ($2048 \times 2048$ pixels) to maintain memory safety across mobile and desktop architectures.
* **Drift Protection Protocol:** Every GPU-computed block is cross-checked against a randomized CPU sample matrix. If float32 rounding errors or driver variants cause a delta drift $> 50\text{ m}$, the engine triggers a silent runtime fallback to the standard CPU matrix loop.

### 3. Optimization Heuristics (Iterated Local Search)
The underlying TSP solver executes a combinatorial pipeline that outperforms standard Nearest-Neighbor (NN) setups by a wide margin:
* **Initialization:** Generates a fast greedy tour using a spatial Nearest-Neighbor heuristic pinned to the user's selected start position.
* **2-Opt Inversion:** Iteratively selects edge pairs $(i, i+1)$ and $(j, j+1)$ and flips the intervening sequence if the swap reduces the path cost:
    $$\Delta = (d_{i,j} + d_{i+1,j+1}) - (d_{i,i+1} + d_{j,j+1}) < 0$$
* **Or-Opt Insertion:** Evaluates single-node relocations along the tour matrix to minimize path cross-over sequences.
* **Double-Bridge Perturbation:** When local minima are reached, the application applies a 4-edge dislocation swap (Double-Bridge kick) before re-entering local refinement loops. This escape technique helps prevent the solution from getting stuck in local minimum traps.

---

## 🛠 Features

* **Dynamic Radius Filtering:** Automatically adjusts inclusion zones centered on a chosen anchor site using a real-time range slider.
* **Flexible Tour Modes:** Computes standard closed-loop round-trips or slices the structural longest-link edge to return custom one-way paths.
* **KML Interoperability:** Accepts custom `.kml` coordinates via drag-and-drop file ingestion, automatically rebuilding dropdown lookups, markers, and maps.
* **Low-Overhead Data Streaming:** Clean exports to both standard `.csv` indices and track-segment `.gpx` files for easy import into Garmin, Strava, or handheld GPS units.

---

## 📦 Deployment and Execution

Because the application is completely decoupled from any persistent database or node server architecture, deployment is instantaneous:

1.  Clone or download the codebase.
2.  Open `index.html` inside any modern web browser supporting WebGL2 (Chrome, Firefox, Safari 15+, Edge).
3.  Click **"Solve Route"** to run the optimization engine.

---

## 🎨 Design and UI System

The front-end design uses a vintage, editorial style system dubbed **Paper & Barn**, which stands out from typical flat tech interfaces:
* **Background Base (`--paper`):** `#F7F1E3` / Warm organic background.
* **Primary Elements (`--pine`):** `#3F5D42` / Foliage green for active paths and primary status indicators.
* **Accents (`--barn` / `--mustard`):** `#A63D2C` / `#D7A32E` / Used for stop numbers, visual progression, and map endpoint elements.
* **Progressive Color Shading:** The map markers use a multi-stop color gradient matching their order along the route, making path sequencing clear at a glance.



<img width="1445" height="978" alt="image" src="https://github.com/user-attachments/assets/12b350cc-2b0d-4d94-8a34-bb166f0ada39" />


<img width="911" height="662" alt="image" src="https://github.com/user-attachments/assets/1f0e2ba7-f847-4f85-8c23-9d952b272e90" />
