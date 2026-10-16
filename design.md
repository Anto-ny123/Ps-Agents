# ICS 3202 Practical Exercise 3: Self-Driving Trailer Agent

**Course:** Artificial Intelligence  
**Repository:** PS-Agents  
**Group members:**  
**ANTHONY MUTUA, EMMANUEL BOSIRE, GLORIA MULUKA**

---

## 0. Evidence of Group Work

Photo evidence sent separately

**Task allocation:**

| **Task**                                    | **Member**      |
|---------------------------------------------|-----------------|
| PEAS and environment properties             | Anthony Mutua   |
| Graph and distance collection (Google Maps) | Emmanuel Bosire |
| Search tree and abstraction                 | Gloria Muluka   |
| Path cost calculation and write-up          | All Members     |

---

## 1. Agent Design

### 1.1 PEAS Framework

The agent is a **self-driving trailer** that hauls cargo between Kenyan towns along the national highway network.

| **PEAS element**        | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Performance measure** | Shortest route / the least travel expenses to get to the target destination; safe arrival with zero collisionsdelivery of completed cargo; delivery on time; fuel efficiency; obeying traffic laws and regulations; minimal delays due to traffic jams, road repairs, or breakdowns                                                                                                                                                                                             |
| **Environment**         | Highways in Kenya (the A104 road connecting Nairobi to Nakuru and Eldoret and the A1 highway connecting Kericho to Kisumu); other vehicles, buses or matatus, private cars, and other trucks; motorcycles/boda boda; pedestrians and domestic animals; road condition (potholes, detours, downhill roads such as Mai Mahiu and Mau Summit); weather (rainfall, foggy conditions, wet roads); traffic signals, traffics signs, traffic circles, road controls, and police stops. |
| **Actuators**           | Driving wheel, accelerator, brakes (including engine/exhaust brakes), gear change, signaling, lights and emergency lights, horn, wipers, coupling trailers, and air-brake controls as well as sending information to the fleet operator.                                                                                                                                                                                                                                        |
| **Sensors**             | GPS and digital map, LiDAR technology, radars, cameras, ultrasonic sensors, speedometer, odometer, fuel and tyre pressure sensors, temperature sensors, weight sensors, and accelerometer/gyroscopes (IMU).                                                                                                                                                                                                                                                                     |

### 1.2 Environment Properties

| **Property**                      | **Classification**                                                                                                                                                                          | **Justification**                                                                                                                                                                                                                                                                     |
|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Fully vs Partially observable** | **Partially observable**                                                                                                                                                                    | Due to limited range of sensors, they are rendered ineffective by trucks, turns and hills. The agent is unable to detect road damage in advance or understand intentions of other cars or the pedestrian ready to cross, thereby having to use its internal model of the environment. |
| **Single vs multi-agent**         | **Multi-agent** (competitive and cooperative)                                                                                                                                               | The roads are used by other cars, cyclists, boda boda drivers and pedestrians. These road users work together by following rules and giving way to each other and compete by overtaking or merging.                                                                                   |
| **Deterministic vs Stochastic**   | **Stochastic**                                                                                                                                                                              | The next stage cannot be accurately predicted based on the current stage of the process and the action. The stopping distance depends on road wetness, the road users may behave unpredictably, owing to possibility of tyre blowouts and engine breakdowns.                          |
| **Episodic vs Sequential**        | **Sequential**                                                                                                                                                                              | Each decision (route choice, overtaking, speed) affects later states and the trip outcome. Taking a wrong junction affects every later step.                                                                                                                                          |
| **Static vs Dynamic**             | **Dynamic**                                                                                                                                                                                 | The environment changes while the agent deliberates: traffic, weather and road conditions change continuously, so the agent must respond in real time.                                                                                                                                |
| **Discrete vs Continuous**        | **Continuous** for driving control (positions, speed and steering are real-valued, time is continuous). **Discrete** at the route-planning level (a finite set of towns and road segments). | The route-planning abstraction in Section 2 is discrete.                                                                                                                                                                                                                              |
| **Known vs Unknown**              | **Partially known**                                                                                                                                                                         | The rules of driving and the road map are known; local road conditions and closures are not.                                                                                                                                                                                          |

---

## 2. Problem Formulation, Directed Graph and Search Tree

### 2.1 Problem formulation (route-planning level)

| **Component**        | **Definition**                                       |
|----------------------|------------------------------------------------------|
| **States**           | The town/city where the trailer currently is         |
| **Initial state**    | Nairobi                                              |
| **Actions**          | Drive along a highway segment to an adjacent town    |
| **Transition model** | RESULT(Town A, Drive to Town B) = Town B             |
| **Goal test**        | Current state = Kisumu                               |
| **Path cost**        | Sum of road distances (km) between consecutive towns |

### 2.2 Regions and towns used

| **Region**              | **Towns**                                           |
|-------------------------|-----------------------------------------------------|
| **Nairobi**             | Nairobi                                             |
| **Central Highlands**   | Thika, Murang'a, Nyeri, Nanyuki                     |
| **Rift Valley**         | Naivasha, Nakuru, Nyahururu, Molo, Kericho, Eldoret |
| **Lake Victoria Basin** | Kisumu, Kakamega, Kisii                             |

### 2.3 Distances between towns

| **Road segment**   | **Distance (km)** | **Highway (approx.)**   |
|--------------------|-------------------|-------------------------|
| Nairobi – Thika    | 42                | A2 / Thika Superhighway |
| Thika – Murang'a   | 40                | A2                      |
| Murang'a – Nyeri   | 55                | A2                      |
| Nyeri – Nanyuki    | 62                | A2                      |
| Nyeri – Nyahururu  | 85                | B5 / C-roads            |
| Nairobi – Naivasha | 92                | A104                    |
| Naivasha – Nakuru  | 64                | A104                    |
| Nakuru – Nyahururu | 90                | B5                      |
| Nakuru – Molo      | 50                | A104                    |
| Molo – Kericho     | 65                | A104 / A1               |
| Kericho – Kisumu   | 80                | A1                      |
| Kericho – Kisii    | 100               | B-roads                 |
| Kisii – Kisumu     | 117               | A1 / B-roads            |
| Nakuru – Eldoret   | 155               | A104                    |
| Eldoret – Kakamega | 85                | A1 / B-roads            |
| Kakamega – Kisumu  | 52                | A1                      |

### 2.4 Directed graph

Edge labels show distance in km.

Photo evidence sent separately

**Adjacency list (each entry is a directed edge: from → to, cost):**

| **From**  | **To (cost in km)**                                     |
|-----------|---------------------------------------------------------|
| Nairobi   | Thika (42), Naivasha (92)                               |
| Thika     | Nairobi (42), Murang'a (40)                             |
| Murang'a  | Thika (40), Nyeri (55)                                  |
| Nyeri     | Murang'a (55), Nanyuki (62), Nyahururu (85)             |
| Nanyuki   | Nyeri (62)                                              |
| Nyahururu | Nyeri (85), Nakuru (90)                                 |
| Naivasha  | Nairobi (92), Nakuru (64)                               |
| Nakuru    | Naivasha (64), Nyahururu (90), Molo (50), Eldoret (155) |
| Molo      | Nakuru (50), Kericho (65)                               |
| Kericho   | Molo (65), Kisumu (80), Kisii (100)                     |
| Eldoret   | Nakuru (155), Kakamega (85)                             |
| Kakamega  | Eldoret (85), Kisumu (52)                               |
| Kisii     | Kericho (100), Kisumu (117)                             |
| Kisumu    | Kericho (80), Kisii (117), Kakamega (52)                |

### 2.5 Search tree with abstraction

**Abstraction is used.** The actual driving process comprises hundreds of minutiae: every intersection, curve, local road, arch bridge and shopping mall. The route planner selects, removing unnecessary detail that might affect the route selection process and creates a two-level hierarchy.

| **Level**            | **What is represented**                                                      | **What is abstracted away**                                                                                                   |
|----------------------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| **Level 1: Regions** | Nairobi → Central Highlands / Rift Valley → Lake Victoria Basin              | Individual towns and roads                                                                                                    |
| **Level 2: Towns**   | Cities and towns as states; one edge per highway segment with total distance | Junctions, intermediate villages, lane-level decisions, speed, traffic, weather (handled by the low-level driving controller) |

Each road segment connecting two cities is treated as a single action and one cost, even though in reality, it involves many subtasks (steering, braking, overtaking) which are managed by the lower-level driving controller.

#### Level 1: Abstract (region-level) tree

Photo evidence sent separately

The goal (Kisumu) lies in the Lake Victoria Basin, which can only be reached from the Rift Valley. The planner therefore focuses on the Rift Valley and expands towns at Level 2.

#### Level 2: Detailed (town-level) search tree

Starting at Nairobi (root). Repeated states that are reached by a more expensive path, and routes back to a town already on the same branch are pruned. g is the cumulative path cost in km.

Photo evidence sent separately

---

## 3. Optimal Path from Nairobi to Kisumu

### 3.1 Method

**Uniform-Cost Search (UCS)** is used because the path cost is the road distance (non-negative, varying edge costs). UCS always expands the frontier node with the lowest cumulative cost g(n) and tests for the goal when a node is **expanded** (popped), which guarantees an optimal solution.

### 3.2 Expansion order

| **Step** | **Node expanded** | **g (km)** | **New frontier nodes (g)**                                          |
|----------|-------------------|------------|---------------------------------------------------------------------|
| 1        | Nairobi           | 0          | Thika (42), Naivasha (92)                                           |
| 2        | Thika             | 42         | Murang'a (82)                                                       |
| 3        | Murang'a          | 82         | Nyeri (137)                                                         |
| 4        | Naivasha          | 92         | Nakuru (156)                                                        |
| 5        | Nyeri             | 137        | Nanyuki (199), Nyahururu (222)                                      |
| 6        | Nakuru            | 156        | Molo (206), Eldoret (311), Nyahururu (246, discarded: 222 is lower) |
| 7        | Nanyuki           | 199        | none (dead end)                                                     |
| 8        | Molo              | 206        | Kericho (271)                                                       |
| 9        | Nyahururu         | 222        | Nakuru (312, discarded)                                             |
| 10       | Kericho           | 271        | Kisumu (351), Kisii (371)                                           |
| 11       | Eldoret           | 311        | Kakamega (396)                                                      |
| 12       | **Kisumu**        | **351**    | **Goal test passed**                                                |

### 3.3 Candidate routes compared

| **Route**                                                                         | **Calculation**                       | **Total (km)** |
|-----------------------------------------------------------------------------------|---------------------------------------|----------------|
| **Nairobi → Naivasha → Nakuru → Molo → Kericho → Kisumu**                         | 92 + 64 + 50 + 65 + 80                | **351** ✅     |
| Nairobi → Naivasha → Nakuru → Eldoret → Kakamega → Kisumu                         | 92 + 64 + 155 + 85 + 52               | 448            |
| Nairobi → Naivasha → Nakuru → Molo → Kericho → Kisii → Kisumu                     | 92 + 64 + 50 + 65 + 100 + 117         | 488            |
| Nairobi → Thika → Murang'a → Nyeri → Nyahururu → Nakuru → Molo → Kericho → Kisumu | 42 + 40 + 55 + 85 + 90 + 50 + 65 + 80 | 507            |

### 3.4 Result

**Optimal path:** Nairobi → Naivasha → Nakuru → Molo → Kericho → Kisumu  
**Total path cost:** **351 km**

The A104 road is followed through the Rift Valley, and then the A1 is used to reach Kisumu, avoiding routes via the Central Highlands and the longer route through Eldoret and Kakamega.

---

## 4. Assumptions and Limitations

- It has been estimated that the distance between town centres is driving distance on the nearest highway without factoring in traffic, road works, weather or steepness.

- Roads have been assumed to be two-way and have the same prices.

- Only the towns cited in Section 2.2 are used for this data; other towns such as Narok, Bomet, Kitale, and Embu could also be included to make it more accurate.

- Real-time data is accounted for by the low-level driving controller and re-planning if any area is blocked.

---

## 5. References

- Russell, S. & Norvig, P. *Artificial Intelligence: A Modern Approach* (PEAS, task environments, uninformed search)

- Google Maps (distance measurements)

- ICS 3202 class notes: self-driving trailer agent discussion
