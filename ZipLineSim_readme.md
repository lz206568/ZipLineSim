# ZipLineSim | Drones Delivery Simulation | Manual Documentation

## 1. Overview

**ZipLineSim** is a discrete-event simulation (DES) framework designed to model and evaluate the operations of a **drone-based last-mile delivery network**. Built entirely in Python using SimPy, it provides a flexible and interactive environment for analyzing the end-to-end logistics flow, from order intake to drone dispatch, aerial delivery, return, and recharge.

This model is inspired by real-world use cases such as medical and vaccine delivery in resource-constrained or time-sensitive environments. It incorporates complex dynamics including **order prioritization (emergency / scheduled / EOD)**, **resource reservation**, **cold-chain support for vaccines**, and **multi-stage delivery logic**.

The simulation enables users to explore how changes in system configurations affect key operational outcomes such as:

-**Delivery timeliness (on-time performance)**
- **Waiting time at each process stage (e.g., assembly, launch)**
- **Utilization of limited resources (drones, batteries, operators)**
- **SLA failure rate across priority classes**
- **Vaccine-specific delivery efficiency under cold chain constraints**

By modeling realistic constraints like **limited fleet size**, **variable order arrivals**, and **resource queuing**, **ZipLineSim** helps decision-makers evaluate and optimize logistics strategies before real-world deployment.

## 2. Requirements

Tested on Python 3.8+ with the following packages:

```bash
pip install simpy numpy pandas matplotlib seaborn ipywidgets
```
###  Package Overview

| Package           | Purpose                                                                 |
|-------------------|-------------------------------------------------------------------------|
| `simpy`           | Core discrete-event simulation engine for modeling events and resources |
| `numpy`           | Numerical operations and random distributions                           |
| `pandas`          | Structured data storage and manipulation                                |
| `matplotlib`      | Plotting library for visualizing performance metrics                    |
| `seaborn`         | Statistical data visualization (e.g., SLA trendlines)                   |
| `ipywidgets`      | Jupyter widgets for building interactive user interfaces                |
| `dataclasses`     | Clean and simple class definitions for simulation objects               |
| `typing`          | Type hints to improve code readability and safety                       |
| `IPython.display` | Utility functions to display output in Jupyter notebooks                |

## 3. Running the Notebook

The way to run the codes is rather simple:

1. The file is in a Jupyter Notebook format. Jupyter is required to run this notebook.

2. Open `ZipLineSim.ipynb` in your preferred environment (Jupyter Notebook, JupyterLab, VS Code, or Google Colab).

3. (Optional) Install any missing packages (see **2. Requirements**).

4. In your environment, click “Run All Cells” to execute the simulation end-to-end.

Note: If you prefer manual control, run each cell in sequence. You can change the parameters by manually change the values in the *Global Settings* and *Key Param Editors* sectors, by the ways that are introduced in the following.

## 4. Notebook Structure

Below is a high-level map of the main sections and their functionalities:

| Section                              | Purpose                                                                 |
|--------------------------------------|-------------------------------------------------------------------------|
| **1. Preparation**                  | Set up environment and key simulation variables                         |
| ├─ 1.1 Dependencies                 | Install and import required Python packages like `simpy`, `numpy`       |
| ├─ 1.2 Global Parameters            | Define core parameters such as fleet size, service time, or capacity    |
| **2. Order Generation**            | Create synthetic order streams for simulation                           |
| ├─ 2.1 Time Distribution Input      | Define order volume per hour using widgets                              |
| ├─ 2.2 Priority Distribution Input  | Choose priority ratios (emergency, scheduled, eod) over time            |
| ├─ 2.3 Vaccine Tagging              | Assign vaccine status to orders for conditional process paths           |
| **3. Call Handling**               | Simulate order intake and call center queueing                          |
| ├─ 3.1 Call Class                   | SimPy resource that models call arrival and handling                    |
| ├─ 3.2 Summary Function             | Analyze waiting time, call load, and visualize distribution             |
| **4. Time Window Assignment**      | Assign delivery windows for scheduled orders                            |
| ├─ 4.1 Window Logic                 | Generate delivery windows using a normal distribution + rounding        |
| **5. Dispatch Simulation**         | Simulate the full delivery workflow (pick, load, fly, return, charge)   |
| ├─ 5.1 Dispatch Class               | Core SimPy logic for assigning and running drone delivery tasks         |
| ├─ 5.2 Queue Management             | FIFO or priority-based queues for staging deliveries                    |
| **6. Output & Evaluation**         | Post-simulation analysis and reporting                                  |
| ├─ 6.1 Delivery Summary            | Waiting time summary per stage + overall utilization metrics            |
| ├─ 6.2 SLA Analysis                | Calculate on-time delivery rate by priority and hour                    |
| ├─ 6.3 Visualization               | Generate bar and line plots for intuitive performance understanding     |

## 5. Parameters Input

In the parts **1.b) Global Settings** and **1.c) Key Param Editors**, all the input parameters can be customized manually. After variables changing, don't forget to run the corresponding cells again.

### 5.1 Variable Dictionary:

| **Stage**      | **Variable**                | **Meaning**                                          | **Type**     | **Default**      | **Change Method**                                   |
| -------------- | --------------------------- | ---------------------------------------------------- | ------------ | ---------------- | --------------------------------------------------- |
| **Global**     | `N_ORDERS`                  | Number of total orders simulated                     | Integer      | `200`            | UI: `OrderNumberInput` Widget                       |
| **Global**     | `ORDER_DIST`                | 12-hour hourly order distribution (7h–18h)           | List\[float] | Bell-curve style | UI: `OrderDistributionInput` Widget                 |
| **Global**     | `HRS_GEN`                   | Number of active delivery hours                      | Integer      | `12`             | Defined in simulation setup                         |
| **Call**       | `call_op`                   | Number of available call operators                   | Integer      | `2`              | Global Settings                                     |
| **Call**       | `call_avg`                  | Average call handling time per order (s)             | Float        | `240`            | Global Settings                                     |
| **Call**       | `call_std`                  | Standard deviation of call duration (s)              | Float        | `60`             | Global Settings                                     |
| **Pick-up**    | `pick_op`                   | Number of pickup operators                           | Integer      | `2`              | Global Settings                                     |
| **Pick-up**    | `tn_gen(300, 90)`           | Non-vaccine pickup duration (truncated normal)       | Distribution | μ=`300`, σ=`90`  | Hardcoded in `Dispatch.pick_process()`              |
| **Pick-up**    | `tn_gen(480, 120)`          | Vaccine pickup duration (truncated normal)           | Distribution | μ=`480`, σ=`120` | Hardcoded in `Dispatch.pick_process()`              |
| **Assembly**   | `ass_avg`                   | Assembly duration mean (s)                           | Float        | `50`             | Global Settings                                     |
| **Assembly**   | `ass_std`                   | Assembly duration std (s)                            | Float        | `10`             | Global Settings                                     |
| **Launch**     | `lauch_avg`                 | Launch duration mean (s)                             | Float        | `80`             | Global Settings (note typo: should be `launch_avg`) |
| **Launch**     | `launch_std`                | Launch duration std (s)                              | Float        | `40`             | Global Settings                                     |
| **Flight**     | `zip_speed_fix`             | Fixed outbound/inbound drone speed (m/s)             | Float        | `23.1`           | Global Settings                                     |
| **Flight**     | `zip_speed_std`             | Std deviation of drone speed (m/s)                   | Float        | `4.0`            | Global Settings                                     |
| **Flight**     | `distance_low`              | Minimum delivery distance (used in speed truncation) | Float        | `5.0`            | Defined in `Dispatch.flight_process()`              |
| **Recovery**   | `recov_avg`                 | Recovery operation average duration (s)              | Float        | `30`             | Global Settings                                     |
| **Recovery**   | `recov_std`                 | Recovery operation std (s)                           | Float        | `10`             | Global Settings                                     |
| **Charging**   | `battery_avg`               | Battery charging average time (s)                    | Float        | `600`            | Global Settings                                     |
| **Charging**   | `battery_std`               | Battery charging std (s)                             | Float        | `120`            | Global Settings                                     |
| **Zip**        | `zips`                      | Number of regular drones available                   | Integer      | `6`              | Global Settings                                     |
| **Zip**        | `zip_reserve`               | Reserved drones for emergency use                    | Integer      | `2`              | Global Settings                                     |
| **Battery**    | `batt`                      | Number of regular batteries available                | Integer      | `8`              | Global Settings                                     |
| **Battery**    | `reserved_batteries`        | Reserved batteries for emergency use                 | Integer      | `2`              | Global Settings                                     |
| **Priority**   | `PriorityDistributionInput` | Defines ratios of emergency/scheduled/eod priority   | Widget       | (0.4, 0.3, 0.3)  | UI: Uniform, by hour, or by period                  |
| **Window**     | `window_after_avg`          | Scheduled delivery time window offset mean (s)       | Float        | `3600`           | `assign_window()`                                   |
| **Window**     | `window_after_std`          | Scheduled delivery time window offset std (s)        | Float        | `900`            | `assign_window()`                                   |
| **Window**     | `window_interval`           | Scheduled delivery window duration (s)               | Float        | `1800`           | `assign_window()`                                   |
| **Simulation** | `sim_time`                  | Total simulation time (s)                            | Integer      | `43200` (12h)    | Global Settings                                     |

### 5.2 To use UI Widgets

#### 5.2.1 `OrderDistInput`

The `OrderDistInput` widget allows users to manually specify the hourly distribution of orders over a 12-hour period (from 07:00 to 19:00). It is especially useful for simulating different demand patterns across the day.

![OrderDistInput UI](./order_dist_input.png)

**Usage Instructions:**

- Each input box corresponds to one hour block (e.g., 07:00–08:00, 08:00–09:00, etc.).
- Enter a decimal value (between 0 and 1) representing the proportion of orders assigned to that hour.
- The **sum of all 12 values must be exactly 1.0**. A dynamic label shows the current sum.
- If the total is invalid (not 1.0), the interface will highlight errors and prevent saving.
- Click **Save Distribution** to apply the distribution.

**Example:**

If you expect higher demand at lunch time, you can input higher proportions for 11:00–14:00. In the image above, the distribution peaks between 11:00 and 13:00.

**Impact:**

Once saved, the input is used to populate the `ORDER_DIST` variable, which controls how many orders are generated per hour in the simulation. This enables testing the drone dispatch system under varying demand curves.

#### 5.2.2 `PrioDistInput`

The `PrioDistInput` widget allows users to define the **priority distribution** of orders—how likely each order is to be an Emergency, Scheduled, or End-of-Day (EOD)—across the 12-hour operational window (07:00–19:00).

![PrioDistInput UI](./prio_dist_input.png)

**Step-by-Step Usage:**

1. **Enter Global Priority Distribution:**
   - Input three float values for Emergency, Scheduled, and EOD.
   - The values must **sum to 1.0** to be valid.
   - The widget provides immediate feedback on whether the input is valid.

2. **Select Assignment Mode:**
   - **Uniform for all hours:** The same distribution applies to every hour.
   - **Manual by hour:** Customize a different (emergency/scheduled/eod) mix for each hour.
   - **Manual by period:** Define hour ranges (e.g., 07–10, 11–14) with different priority distributions.

3. **Configure Distribution Details:**
   - Based on the selected mode, users are prompted to either preview or configure the details.
   - Validation ensures each hourly or period-level distribution also sums to 1.

**Example:**
In the image above, each hour will use a fixed split of:
- Emergency: **40%**
- Scheduled: **30%**
- EOD: **30%**

**Impact:**
This distribution directly affects the `priority` field in the generated `Order` objects. It allows realistic modeling of situations where, for example, emergencies are more likely in certain hours or periods of the day.

#### 5.2.3 `OrderNumberInput`

The `OrderNumberInput` widget allows the user to define the **total number of delivery orders** to simulate in the system.

![OrderNumberInput UI](./OrderNumberInput.png)

**How to Use:**

- Simply enter the number of orders you'd like to simulate in the input box.
- Press the green **"Save"** button to confirm your input.
- The value is stored and used to generate synthetic order calls across the defined time window (e.g., 07:00–19:00).

**Example:**
In the image above, the user has chosen to simulate **120 orders**. These will be distributed across the 12-hour simulation window according to the time and priority distributions specified by the other widgets (`OrderDistInput` and `PrioDistInput`).

**Impact:**
This input determines the workload level for the simulation and influences:
- System congestion
- Resource utilization
- Waiting times
- Fulfillment rate

It is especially useful for **stress-testing** resource limits and evaluating dispatch performance under different demand levels.


## 6. Model Methodology

This simulation is built using a **Discrete Event Simulation (DES)** framework powered by `SimPy`, a Python-based process-based simulation library. The model captures the **end-to-end lifecycle of a drone delivery order**, from order arrival to final completion. It simulates the **interaction between delivery demand and limited operational resources**, allowing for detailed performance evaluation under varying conditions.

### 6.1 Stages

The ZipLineSim simulation models the end-to-end delivery process in the following discrete stages:

1. **Call Intake**  
   Customer orders arrive according to a user-defined hourly distribution. A limited number of call operators handle incoming calls. Calls may experience wait time if all operators are busy.

2. **Order Dispatch & Queueing**  
   After a call is completed, the corresponding order is queued for processing. If the order contains multiple packages, each becomes a separate delivery unit.

3. **Pick-up**  
   A pick-up operator is assigned to retrieve the order. If the order is a vaccine delivery, it uses a separate (usually longer) distribution for pick-up time. Orders wait in queue if no operator is available.

4. **Assembly**  
   An assembly operator prepares the drone (Zip) and package for launch. This step includes mounting the payload and preparing flight configuration.

5. **Launch**  
   Once a launcher becomes available, the drone is launched. The time taken depends on a launch time distribution. Orders may queue here due to launcher contention.

6. **Flight (Outbound + Inbound)**  
   The drone flies to the destination and then returns. Both outbound and inbound speeds are drawn from a stochastic distribution. The total flight time is logged.

7. **Recovery**  
   Upon return, a recovery operator is required to bring the drone back. This step may experience delays depending on recovery resource availability.

8. **Charging**  
   The battery is detached and charged at a charging station. Charging time follows a stochastic distribution. Once charged, the battery is returned to the available pool.

9. **Resource Reassignment**  
   After completion, drones and batteries are returned either to the regular pool or to a dedicated emergency reserve pool depending on their IDs.

Each of these stages competes for constrained shared resources (e.g., pick-up operators, drones, batteries, chargers), and the simulation tracks delay, bottlenecks, and performance metrics at each step.

### 6.2 Resources

#### Operators

| Resource Type      | Count | Description                                                                |
| ------------------ | ----- | -------------------------------------------------------------------------- |
| Call Operators     | 3     | Handle incoming orders; each call takes \~300s on average                  |
| Pick-up Operators  | 3     | Retrieve and prepare the order; 300–600s depending on whether it's vaccine |
| Assemble Operators | 1     | Responsible for drone assembly and launch (shared role)                    |
| Recovery Operators | 1     | Handle drone recovery and post-flight checks                               |

#### Drone & Battery System
| Resource Type           | Count | Description                                                         |
| ----------------------- | ----- | ------------------------------------------------------------------- |
| Zips (Drones)           | 18    | Deliver packages to destinations                                    |
| Reserved Zips (for EM)  | 2     | Set aside exclusively for emergency orders                          |
| Batteries               | 40    | Each flight consumes one battery, which needs recharging afterwards |
| Reserved Batteries (EM) | 2     | Reserved battery units for emergency drone missions                 |
| Chargers                | 20    | Recharge batteries; each charge cycle takes \~30 minutes            |

#### Fixed Infrastructure
| Resource Type     | Count | Description                                                   |
| ----------------- | ----- | ------------------------------------------------------------- |
| Launchers         | 1     | Needed to launch zips; used briefly                           |
| Charging Stations | 20    | Recharge depleted batteries                                   |
| Recovery Station  | 1     | Recovers zips post-mission; shared resource, \~2 minutes/task |

#### Resource Usage Timing (Mean ± Std in Seconds)
| Stage    | Non-Vaccine Orders       | Vaccine Orders           |
| -------- | ------------------------ | ------------------------ |
| Call     | 300 ± 120                | —                        |
| Pick-up  | 300 ± 60                 | 600 ± 100                |
| Assembly | 180 ± 60                 | 180 ± 60                 |
| Launch   | 20 ± 5                   | 20 ± 5                   |
| Recovery | 120 ± 30                 | 120 ± 30                 |
| Charging | 1800 ± 600 (30 mins ±10) | 1800 ± 600 (30 mins ±10) |


### 6.3 Core Logic of Codes

#### Process Definition

#####  Class Definitions

######  `Order`
Defines an incoming order and tracks its attributes and progress throughout the simulation. Includes:
- Metadata: `id`, `prio`, `gen_at`, `distance`, `weight`
- Time windows: `win_start`, `win_end`
- Vaccine status: `is_vac`
- Call process: `call_start`, `call_end`, `call_op_id`, `call_dur`
- Completion status: `complete`, `ontime` (whether it was delivered on time)

###### `Delivery`
Represents the entire delivery process of an order once it enters the logistics flow. Tracks:
- Associated order and metadata: `order_id`, `prio`, `distance`, `is_vac`
- **Pick-up stage**: `pick_start`, `pick_dur`, `pick_op_id`, etc.
- **Assembly stage**: `assm_start`, `assm_op_id`, etc.
- **Launch stage**: `lnch_start`, `lnch_id`
- **Flight stage**: outbound/inbound speeds and durations, `arrive_time`, `flight_end`
- **Recovery stage**: `reco_start`, `reco_op_id`, `reco_end`
- **Drone & battery**: `zip_id`, `batt_id`, `chgr_id`, charging time
- **Result**: `ontime` status flag

###### `ResourceUnit`
A minimal class that wraps a resource (e.g., a drone, battery, or operator) with a unique ID, used for resource assignment.

##### Useful Functions

###### `tn_gen()`
Generates a value from a truncated normal distribution given a mean (`avg`), standard deviation (`std`), and bounds (`lower`, `upper`). Used for sampling durations like pick-up time, charging time, etc.

###### `TimeTick` class
Converts simulation timestamps (in seconds) into human-readable day-hour-minute-second strings. Also stores derived values like `self.dd` (day of simulation), `timetag` (formatted string), and `durtag` (duration in readable form).

###### `df_str(df: pd.DataFrame)`
Formats a simulation result DataFrame to be human-readable:
- Converts raw time columns into formatted strings (e.g., `Start`, `End`, `GenerateTime`)
- Rounds numerical values for distances, weights, and speeds
- Ensures IDs are shown cleanly as integers

##### Main Process

###### `Call`
A class that manages order (call) generation during the simulation:
- Takes parameters like simulation duration, number of call operators, average and standard deviation of call durations
- Responsible for generating and queuing orders at appropriate times

###### Deliveries Dispatch-out
Handles the core delivery pipeline after an order is confirmed.  
It manages resource allocation (e.g., drones, batteries, operators), queues deliveries for pick-up and flight, and simulates the full delivery lifecycle using SimPy.

- Initializes all resource pools and internal queues.
- Converts orders to deliveries and schedules them.
- Coordinates all stages: pick-up, assembly, launch, flight, recovery, and charging.

###### Aggregated
Defines the full order generation and processing pipeline.  
It generates orders based on a pre-defined schedule, assigns dynamic attributes (priority, weight, distance, etc.), and then calls both `Call` and `Dispatch` to simulate the complete lifecycle.

- Generates new orders, assigning them an ID, priority, distance, weight, and time window.
- Coordinates the execution of both the call-taking and dispatching stages for each order.

###### Whole Simulation
Creates `Call` and `Dispatch` system objects, triggers the order generation process, and runs the full simulation for a specified duration.

###### Outcome Conversion
Convert the internal simulation results into two pandas DataFrames for easier post-simulation analysis:
- Extracts order-level information like ID, priority, call timing, and vaccine status.
- Extracts delivery-level information including pickup, assembly, launch, flight, recovery, and resource usage metrics.


## 7. Simulation and Output

### 7.1 Run the Simulation

Use the code to run the simulation:
```
caller, dispatcher = sim_run()
```

You can easily run the simulation with different number of orders in the `()`, for example:
```
caller, dispatcher = sim_run(150)
```
will run the simulation with 150 orders.

#### 7.1.1 orders2df(caller)
This DataFrame contains all generated orders and their properties.

| Field                      | Description                                          |
| -------------------------- | ---------------------------------------------------- |
| `Priority`                 | Order type: `'Emergency'`, `'Scheduled'`, or `'EOD'` |
| `GenerateTime`             | Timestamp when the order was created                 |
| `Distance(km)`             | Delivery distance in kilometers                      |
| `Weight(kg)`               | Weight of the delivery payload                       |
| `CallStart`, `CallEnd`     | Start and end times of call handling                 |
| `CallDuration`             | Duration of the call operation                       |
| `CallOpID`                 | ID of the assigned call operator                     |
| `WindowStart`, `WindowEnd` | Scheduled delivery window (if applicable)            |
| `IsVaccine`                | Boolean flag indicating if cold-chain is required    |

This data is helpful for understanding order patterns and system load.

#### 7.1.2 deliveries2df(dispatcher)
This DataFrame contains all completed deliveries and records detailed performance metrics across each operational stage.

| Field Category          | Description                                                                                           |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| **Timing & Duration**   | Timestamps and durations for pick-up, assembly, launch, flight, recovery, and charging                |
| **Flight Metrics**      | Outbound/inbound speeds, delivery arrival time, total flight duration                                 |
| **Resource IDs**        | IDs for pick-up operator, assembly operator, launcher, recovery operator, drone, battery, and charger |
| **Delivery Attributes** | Priority type, delivery distance, delivery time window, vaccine requirement                           |

This is the main dataset for evaluating delivery efficiency, bottlenecks, and on-time rates.

### 7.2 Performance - Waiting Time
This section analyzes how long orders wait at each stage of the delivery process. It also identifies overweight orders that require multiple trips and calculates the number of deliveries needed, completed deliveries, and waiting time distribution.

####Summary Table
| Stage    | Avg Wait Time | Median Wait | Max Wait | Min Wait |
| -------- | ------------- | ----------- | -------- | -------- |
| Call     | 08m 09s       | 02m 38s     | 31m 42s  | 00m 00s  |
| Pick-up  | 01m 50s       | 00m 35s     | 14m 24s  | 00m 00s  |
| Assembly | 38m 29s       | 07m 47s     | 2h 56m   | 00m 00s  |
| Launch   | 00m 00s       | 00m 00s     | 00m 00s  | 00m 00s  |
| Recovery | 00m 42s       | 00m 00s     | 05m 57s  | 00m 00s  |
| Charging | 00m 00s       | 00m 00s     | 00m 00s  | 00m 00s  |


### 7.3 Performance - On-Time Rate
This section evaluates the timeliness of deliveries by comparing actual delivery times with priority-specific expectations.

#### Logic Behind the Calculation
1. Emergency Orders must be delivered within 2 hours of generation.
2. Scheduled Orders must arrive before their time window ends.
3. End-of-Day (EOD) Orders must arrive within 6 hours.
4. The model checks each order against these criteria and calculates:
5. Delivery time (arrival time minus generation time)
6. Whether the delivery was on-time or late
7. Whether the order was fully completed (i.e., all deliveries finished)

#### Emergency Delivery Time Summary

| Metric                            | Time        |
|-----------------------------------|-------------|
| Average Delivery Time             | 01h 06m 56s |
| Median Delivery Time              | 01h 04m 53s |
| Maximum Delivery Time             | 02h 16m 00s |
| Minimum Delivery Time             | 26m 17s     |
| 75% of Orders Arrived Within      | 01h 20m 57s |
| 90% of Orders Arrived Within      | 01h 36m 30s |

#### Scheduled Delivery Time Summary

| Metric                            | Time        |
|-----------------------------------|-------------|
| Average Delivery Time             | 01h 00m 59s |
| Median Delivery Time              | 01h 16m 02s |
| Maximum Delivery Time             | 01h 44m 00s |
| Minimum Delivery Time             | 23m 08s     |
| 75% of Orders Arrived Within      | 01h 23m 00s |
| 90% of Orders Arrived Within      | 01h 32m 11s |

#### EOD (End of Day) Delivery Time Summary

| Metric                            | Time        |
|-----------------------------------|-------------|
| Average Delivery Time             | 01h 21m 01s |
| Median Delivery Time              | 01h 08m 27s |
| Maximum Delivery Time             | 02h 46m 50s |
| Minimum Delivery Time             | 22m 26s     |
| 75% of Orders Arrived Within      | 01h 37m 35s |
| 90% of Orders Arrived Within      | 02h 16m 11s |

### 7.4 Performance - Resource Summary
Analyzes how resources are used throughout the delivery simulation, summarizing both usage counts and total working durations per resource.

#### Call Operators Summary

| Operator | Times Operated | Total Working Duration |
| -------- | -------------- | ---------------------- |
| 0        | 69 times       | 05h 41m 51s            |
| 1        | 65 times       | 05h 30m 14s            |
| 2        | 65 times       | 05h 25m 33s            |

#### Pick-up Operators Summary
| Operator | Times Operated | Total Pick-up Duration |
| -------- | -------------- | ---------------------- |
| 0        | 43 times       | 04h 53m 41s            |
| 1        | 39 times       | 04h 46m 15s            |
| 2        | 39 times       | 04h 33m 59s            |

#### Assembly Operators & Launchers
1 assembly operator handled 121 operations for a total of 06h 24m 44s
1 launcher was used 121 times, totaling 40m 16s

Assembly is significantly more time-consuming than launch, highlighting a potential bottleneck.

#### Chargers Summary (Excerpt)
| Charger ID | Times Used | Charging Time |
| ---------- | ---------- | ------------- |
| Charger 0  | 7          | 03h 21m 13s   |
| Charger 1  | 6          | 03h 11m 28s   |
| …          | …          | …             |

#### Zip Usage Summary
| Zip ID | Deliveries | Total Distance (km) |
| ------ | ---------- | ------------------- |
| Zip 0  | 8          | 634.57 km           |
| Zip 1  | 7          | 749.66 km           |
| …      | …          | …                   |

#### Battery Summary (Excerpt)
| Battery ID | Flights | Total Flight Time | Charging Time |
| ---------- | ------- | ----------------- | ------------- |
| Battery 0  | 4       | 03h 58m 00s       | 01h 04m 53s   |
| Battery 1  | 3       | 03h 56m 40s       | 01h 01m 36s   |
| …          | …       | …                 | …             |

This resource-level summary enables bottleneck identification and infrastructure planning — for example:

High assembly time suggests a need for more assembly operators.

Zip and battery usage metrics support drone fleet optimization.

Charger usage informs whether the number of chargers is adequate under current load.

## 8. Conclusions

The ZipLineSim model offers a flexible and realistic framework for simulating last-mile autonomous drone delivery operations. By integrating discrete event simulation (DES) with configurable resource pools, delivery priorities, and time-window constraints, the model captures the complexity of real-world logistics systems. It provides visibility into system bottlenecks, such as operator overload or drone scarcity, and quantifies their impact on key performance metrics like on-time rate and waiting time. However, the current model assumes static resource allocation and does not incorporate predictive or dynamic scheduling, which may limit adaptability under shifting demand patterns. 

Several improvements could improve the efficiency and realism of the current ZipLineSim model. First, adding critical bottleneck resources, especially more assembly operators and expanding the drone or battery fleet, would help alleviate congestion during peak hours. In addition, the model currently prioritizes heavily towards urgent orders, often at the expense of regularly scheduled deliveries; adjusting the delivery sequencing logic to balance urgency and fairness could improve SLA. In addition, the midday peak could be reduced by modifying the order generation distribution to smooth out changes in demand over time. Incorporating batching or grouping logic could potentially improve operational efficiency, especially for lightweight orders that currently trigger a full delivery workflow. Finally, the system could employ a dynamic resource allocation strategy that allows for temporary reallocation of urgently reserved drones or batteries during off-peak hours to support regularly scheduled deliveries without compromising critical readiness. 
