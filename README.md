# E-commerce Warehouse Optimization and Robotization  

---
## Overview
This project implements a **discrete and reproducible simulator** of a **mobile robot fleet** operating in a typical e-commerce Fulfillment Center (FC). The technological proposal is part of a digital transformation strategy oriented towards an automated fulfillment center, designed for a leading company in the sector.

The system allows you to:
*   Generate **realistic warehouse layouts** (shelves, aisles, and stations).
*   Fragment the FC into 6 operational quadrants to optimize the storage of high-rotation products.
*   Generate **discrete orders** (discrete-time events), with temporal control using ticks.
*   Simulate the **complete FC operation**, including order allocation, route planning, and a "parking" policy for inactive robots.
*   Obtain **quantitative performance metrics** of the system, such as percentage of completed orders, throughput, deadlocks, waiting ticks, and the P95 indicator.

> The project is explicitly designed as an **academic challenge and experimental benchmark**, seeking to validate improvements through structured experimentation.

![Python 3.8+](https://img.shields.io/badge/Python_3.8+-3776AB?logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?logo=numpy&logoColor=white) 
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557c) 
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0)
![JSON](https://img.shields.io/badge/JSON-000000?logo=json&logoColor=white)

--- 

## Scope and Purpose
The goal of the benchmark is to study how different local decisions impact the global performance of the system. Through experimental design, the following hypothesis is proposed and validated:
*   A strategic redistribution of the warehouse (zoning into 6 quadrants) and the allocation of rest areas for robots allows for a 20% increase in completed orders and a 10% reduction in average delivery time.

The simulator provides a technically consistent case study, allowing the comparison of the base scenario (benchmark) with intervened scenarios to justify engineering improvements through the analysis of operational trade-offs.

---

## Demo


https://github.com/user-attachments/assets/3b18e775-bde6-4095-a8e3-2de11ef7d70a


---

## Project Structure
```text
sim_almacen/
├─ README.md
├─ requirements.txt
├─ a_estrella.py
├─ demo_final.py
├─ generador_layout.py
├─ generador_pedidos.py
├─ out_paths.py
├─ sim_core.py
├─ tabla_reservas.py
├─ visualiza_simulacion.py
├─ pruebas/
│  ├─ prueba_a_estrella.py
│  └─ prueba_tabla_reservas.py
└─ outputs/
   └─ <scenario>/
      ├─ anaqueles.json
      ├─ estaciones.json
      ├─ metricas.json
      ├─ pedidos.json
      ├─ spawn.json
      ├─ zonas.json
      ├─ layout.npy
      ├─ heatmap_esperas.png
      ├─ heatmap_ratio.png
      ├─ heatmap_visitas.png
      ├─ layout.png
      └─ simulacion.mp4
```
## Scenario Convention
All benchmark results are grouped under the folder:
outputs/<scenario>/

## Valid examples used in the experimental validation:  
* outputs/Escenario-S1-SB/ (Operational baseline without burst).  
* outputs/Escenario-S2-CB-x2/ (Double fleet stress with burst).  
* outputs/Escenario-S1-SB-EF/ (Baseline with strong zones and parking).  

---

## Recommended Execution Flow

### Step 0 – Install dependencies
Make sure to install the required libraries via:
pip install -r requirements.txt

### Step 1 – Generate the FC layout
The experimental design of this FC establishes a much wider base spatial matrix for testing.  
python generador_layout.py --escenario seed44 --seed 44 --ancho 300 --alto 200 --estaciones 20

* The --seed 44 parameter guarantees the pseudo-random generation of a 60,000 square unit layout, internally segmented into 6 quadrants.  
* Parking areas are enabled in the 4 corners of the warehouse.  

### Step 2 – Generate discrete orders
python generador_pedidos.py --escenario seed44 --pedidos 600 --burst --distribucion "5:15:5:20:35:20"

* Orders are introduced through stochastic sampling to force a strong heterogeneous distribution (ZF), bringing the most requested products closer to the delivery area.  
* The --burst flag simulates critical demand peaks.  

### Step 3 – Run the simulation (benchmark)
python demo_final.py --escenario seed44 --robots 20 --ticks 10000

* Generates metrics such as deadlock reduction (close to 100% thanks to the parking policy) and a throughput of up to 61.64 orders/1000t in an optimal operational configuration.  

### Step 4 – Visualize the simulation
python visualiza_simulacion.py --escenario seed44 --fps 30 --pasos_por_frame 10

* Generates the simulation video, highlighting robots returning to base/parking in red to facilitate the visual analysis of congestions.  

---

## Main System Components

### sim_core.py
The simulator's core with rescue interventions and priorities:
* A rescue mechanism was incorporated for robots stuck in the return phase (A_CARGA), forcing their inactive state after 10 ticks without movement.  
* Priority is given to resolving reservations for robots with active orders over those in the resting phase.  

### generador_layout.py
Generates the environment integrating the new zoning:
* Divides the storage zone into 6 quadrants using a 3x2 grid, writing the coordinates to zonas.json.  

### generador_pedidos.py
Implements the stochastic requirements system:
* Replaces uniform generation with a probabilistic selection of shelves dictated by the weights of the --distribucion flag, concentrating demand in the most accessible quadrants.  

### visualiza_simulacion.py
* Expands the color map to identify the A_CARGA state (#e74c3c).  
* Improves grid centering and adjusts rendering to prevent cropping in the metrics panels.  

### Support Algorithms (a_estrella.py and tabla_reservas.py)
* Maintain the temporal coordination mechanism to avoid vertex collisions and space-time inconsistencies, now delegating decongestion to the rest area system.  

---

## Results and Robust Evaluation
The combination of a strong zonal distribution and parking areas demonstrated technical superiority over the original benchmark:  
* Throughput: Notable increase, achieving up to a 169.27% improvement under fleet stress (x3) without burst.  
* Deadlocks: Almost total elimination of stagnation, reducing massive incidents (from 5,336 to only 3 in critical scenarios).
* Distance: Reduction of up to 21.83% in travel routes by bringing high-rotation products closer.
* Completeness: Reaches 100% success in scenarios where the original version only achieved 82.8% within the time limit.

## Reproducibility
The system is completely deterministic. Setting the --seed 44 parameter allows repeated validation of demand stress, fleet stress (x2, x3), and zonal distribution variants.

## Academic Use
This integrative benchmark simulates an industrial challenge with real strategic implications:
* Change Management: Evaluates how operational variables (such as adding rest zones) resolve high-cost infrastructure bottlenecks.
* Technical-Operational Analysis: Enables budgeting (Capex/Opex), risk registration, and software specifications derived from the digital model's behavior.

## Design Philosophy
The system deliberately separates policies from mechanisms, creating a controlled environment for analyzing trade-offs. For instance, while the exclusive use of strong zones reduces travel distances, it can increase waiting ticks; compensating for this requires the parking mechanism, which validates the need for a systemic approach to warehouse redesign.

## Glossary
* Order allocation: Process by which an order is assigned to an available robot.
* Benchmark: Reference implementation designed to compare strategies.
* Burst: Order generation pattern that simulates temporal load peaks.
* FC (Fulfillment Center): Abstraction of a distribution center where reception, inventory, and deliveries are coordinated.
* Deadlock: Temporary stagnation event due to spatial congestion.
* High events: Occasions when the system detects high activity, dense interactions between robots, or an immediate inability to determine a free route.
* P95: Metric indicating the exact tick at which the system manages to complete 95% of the required order load.
* Parking (Rest Areas): Strategic zones (typically the 4 corners of the layout) designated for inactive robots to remain without obstructing the traffic flow of active robots.
* Tick: Basic unit of temporal advancement.
* Zoning (Quadrants): Segmentation of the warehouse into specific areas to concentrate high-rotation inventory, reducing travel distances to delivery stations.

---

## Authors

* Rafael Soto Padilla
* Arturo Barrios Mendoza 
* Lucio Arturo Reyes Castillo
* Mariana Balderrábano Aguilar
* Maximiliano De La Cruz Lima
* Lizbeth Islas Becerril
* Carlos Alberto Zamudio Velázquez 

## References
1. Mercado Libre. (n.d.). E-commerce Warehouse Layout Strategies and Multi-Robot Coordination.
2. Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). A Formal Basis for the Heuristic Determination of Minimum Cost Paths (Reference for a_estrella.py). IEEE Transactions on Systems Science and Cybernetics.
3. Wurman, P. R., D'Andrea, R., & Mountz, M. (2008). Coordinating Hundreds of Cooperative, Autonomous Vehicles in Warehouses. AI Magazine.
4. Internal Project Documentation: Benchmark de planeación, asignación y coordinación multi-robot.
