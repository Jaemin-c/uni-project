# Project 4: Pokemon - EECS 281

> 소스 코드는 EECS 281 수업 정책상 공개할 수 없습니다.  
> 이 문서는 구현 개요와 설계 요점, 사용 기술을 요약한 문서입니다.

This project is a complete C++ implementation of three approaches to the **Traveling Salesman Problem (TSP)**.  
Each mode offers a different balance of accuracy and performance:

- **MST** — Minimum Spanning Tree construction  
- **FASTTSP** — Fast greedy heuristic for approximating TSP  
- **OPTTSP** — Optimal TSP using branch and bound

---

### 1. MST Mode (Minimum Spanning Tree)

Constructs a minimum spanning tree from a set of 2D coordinates.

- **Input:** A list of coordinates.  
- **Output:**  
  - Total weight of the MST (2 decimal places)  
  - List of edges (vertex pairs)

- **Special Condition:**  
  If both land and sea points exist without a coastline point to connect them, the MST cannot be built.

- **Implementation:**  
  Uses Prim’s algorithm without a priority queue for determinism.

---

### 2. FASTTSP Mode (Heuristic TSP)

Provides a fast, approximate solution to the TSP using greedy insertion.

- **Algorithm:**
  - Start with an initial triangle.
  - Iteratively insert each remaining point at the position where it causes the least increase in path length.

- **Output:**
  - Approximate total path length
  - Vertex visitation order

---

### 3. OPTTSP Mode (Optimal TSP)

Finds the optimal (shortest possible) tour using branch and bound.

- **Algorithm Steps:**
  1. Start from a greedy FASTTSP tour as the upper bound.
  2. Use recursive permutation generation (`genPerms()`).
  3. Apply pruning with:
     - Lower bound estimation using MST on unvisited vertices.
     - Distance estimates from start and end of partial path.

- **Output:**
  - Optimal tour length
  - Exact visiting order of vertices

---