## 3. Methodology for Autonomous Operation

### 3.1 Autonomous Flight Algorithm

#### 3.1.1 System State Machine

The autonomous flight architecture is orchestrated using an event driven Hierarchical Finite State Machine framework which divides the overall mission into multiple sequential operational phases. The system architecture is organized into top level operational super states such as Take-off, Arena Scanning, Corridor Navigation and Target Interaction within which individual localized tasks operate as child states. This hierarchical topology ensures that specific operational parameters, tracking logic, and sensor filtering profiles are explicitly bound to their corresponding stage of flight.

A key feature of this HFSM setup is its conditional transition logic, where the system progresses to the next mission phase only after predefined sensor and mission conditions are satisfied. In addition, a confidence-based verification layer continuously validates sensor observations across multiple readings before executing autonomous actions, improving flight reliability and reducing false detections.

#### 3.1.2 Global Path Planning & Coverage

The global search phase was implemented using a Boustrophedon or Lawnmower Path based coverage strategy over a structured rectangular arena of dimensions **40 m × 30 m**, resulting in a total operational search space of **1200 m²**. Since the environment is convex, direct sweep line generation was performed without requiring Boustrophedon Cellular Decomposition, thereby reducing computational overhead and simplifying waypoint generation.

The generated traversal paths were aligned along the longer axis of the arena to minimize turning frequency and improve path continuity, which is a standard optimization criterion in Coverage Path Planning literature.

The coverage geometry was derived using the downward facing camera with a horizontal field of view of **110°** and a vertical field of view of **94°**, operated at a flight altitude of **10 m**. To improve search robustness and reduce sensing discontinuities, both forward overlap and side lap were maintained at **70%**, resulting in the generation of four parallel sweep trajectories to ensure deterministic full area traversal with bounded overlap redundancy and reliable visual continuity across the Region of Interest (ROI).

Due to the deterministic nature of the sweep generation process, the algorithm exhibits linear scalability with respect to the number of generated sweep tracks, enabling computationally efficient real time deployment on the Raspberry Pi 4. The selected planning methodology provides high empirical coverage efficiency in structured environments owing to its predictable traversal geometry, low memory footprint, reduced waypoint complexity, and guaranteed completeness of coverage.

| Algorithm               | Time Complexity | Space Complexity |
| ----------------------- | --------------- | ---------------- |
| Boustrophedon algorithm | O(n)            | O(n)             |


The proposed Boustrophedon Longitudinal Sweep strategy was benchmarked against Boustrophedon Transverse Sweeps and a Stochastic Frontier exploration approach. Comparative evaluation showed that the longitudinal sweep configuration achieved the best overall performance, with the performance analysis for algorithm selection presented in the Appendix.

#### 3.1.3 Local Obstacle Avoidance

An obstacle avoidance planning algorithm is used for dynamic red zones discovered on-the-fly and added to the global Boustrophedon coverage planning algorithm as local planners for short-term corrections along the current global sweep path.

The red zones are computed from the images taken from the camera mounted at the bottom of the drone and correspond to planar forbidden areas in the global coordinate frame that are projected into the local coordinate frame around the UAV.

The Vector Field Histogram (VFH) approach maps the obstacle map produced by the camera to an angular representation around the drone, which considers each sector in the field of view to be associated with certain collision risk depending on the number and intensity of red zones present within the area.

This allows generating a polar occupancy histogram map, in which higher occupancy corresponds to a riskier direction, whereas lower occupancy corresponds to a feasible motion corridor. The next move vector is chosen through collision risk minimization and optimal heading toward the global Boustrophedon waypoint.

To improve motion smoothness and ensure dynamic feasibility, the Dynamic Window Approach (DWA) is applied over the VFH selected safe direction set. DWA selects safe and feasible UAV velocity commands by simulating short future trajectories and scoring them based on target direction and obstacle avoidance.

The optimal velocity command is then executed, ensuring that geometric safety from VFH is refined into physically consistent UAV motion.

Once the red zone is cleared, a rejoining mechanism restores continuity with the global coverage path by computing the nearest projection onto the predefined sweep line structure and generating a re-entry target on the closest valid lane. The local planner then smoothly guides the UAV back to the global trajectory, after which control is handed over to resume Boustrophedon coverage.

| Algorithm | Time Complexity | Space Complexity |
| --------- | --------------- | ---------------- |
| VFH       | O(n)            | O(n)             |
| DWA       | O(m·T)          | O(m·T)           |

where **n** denotes the number of angular sectors in the VFH histogram, **m** represents the number of sampled velocity pairs (v,ω) in DWA, and **T** is the finite time horizon used for trajectory prediction.
