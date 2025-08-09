Certainly! Below is the **complete and updated mathematical model** for your green ammonia power plant optimization, incorporating **all your constraints and inputs**, including:

* Hourly solar/wind generation in MW (actual, not per-MW installed)
* Transmission losses
* Electrolyzer and ammonia plant efficiencies
* Storage (battery and hydrogen)
* Grid interaction
* Minimum ammonia plant loading
* Capital cost minimization objective

---

# 📘 **Green Ammonia Plant Optimization Model**

---

## 🎯 **Objective**

**Minimize total capital investment**, while meeting the ammonia production target using on-site renewable energy (solar + wind), hydrogen electrolysis, storage systems, and optional grid withdrawals (capped).

---

## 🧩 **Decision Variables**

| Variable              | Description                           |
| --------------------- | ------------------------------------- |
| $C^{\text{solar}}$    | Installed solar capacity (MW)         |
| $C^{\text{wind}}$     | Installed wind capacity (MW)          |
| $C^{\text{elyz}}$     | Electrolyzer capacity (MW)            |
| $C^{\text{bat}}$      | Battery energy storage capacity (MWh) |
| $C^{\text{H2\_stor}}$ | Hydrogen storage capacity (kg)        |
| $C^{\text{NH3}}$      | Ammonia loop capacity (tonnes/hour)   |

For each hour $t = 1, \ldots, 8760$:

| Time-Indexed Variable               | Description                             |
| ----------------------------------- | --------------------------------------- |
| $G_t^{\text{solar}}$                | Solar generation (MW, given input)      |
| $G_t^{\text{wind}}$                 | Wind generation (MW, given input)       |
| $B_t^{\text{in}}, B_t^{\text{out}}$ | Battery charge/discharge (MWh)          |
| $S_t^{\text{bat}}$                  | Battery SOC (MWh)                       |
| $E_t^{\text{elyz}}$                 | Power to electrolyzer (MWh)             |
| $H_t^{\text{prod}}$                 | Hydrogen produced (kg)                  |
| $H_t^{\text{used}}$                 | Hydrogen consumed in NH₃ synthesis (kg) |
| $S_t^{\text{H2}}$                   | Hydrogen storage (kg)                   |
| $A_t$                               | Ammonia produced (tonnes)               |
| $G_t^{\text{grid\_in}}$             | Power withdrawn from grid (MWh)         |
| $G_t^{\text{grid\_out}}$            | Power exported to grid (MWh)            |

---

## 📥 **Given Inputs**

| Symbol                                      | Description                                 |
| ------------------------------------------- | ------------------------------------------- |
| $SolarHourly[t]$                            | Solar power generation at hour $t$ (MW)     |
| $WindHourly[t]$                             | Wind power generation at hour $t$ (MW)      |
| $\eta^{\text{elyz}} = \frac{1}{55}$         | H₂ per kWh: 1 kg H₂ per 55 kWh              |
| $\eta^{\text{NH3}} = 0.18$                  | H₂ needed per tonne NH₃ (≈ 180 kg H₂/tonne) |
| $\eta^{\text{bat}}$                         | Battery roundtrip efficiency (e.g., 90%)    |
| $\eta^{\text{trans}} = 0.955$               | Transmission efficiency (i.e., 4.5% loss)   |
| $E^{\text{NH3}} = 1 \text{ kWh/kg NH₃}$     | Ammonia process energy consumption          |
| $A^{\text{target}} = 200{,}000$             | Annual ammonia production (tonnes)          |
| $G^{\text{grid\_max}} = 50{,}000$           | Max annual grid withdrawal (MWh)            |
| $C^{\text{NH3\_max}} = 690$                 | Max daily NH₃ production (tonnes)           |
| $A^{\text{min}} = 0.2 \cdot C^{\text{NH3}}$ | Minimum hourly production (20% load)        |

---

## 📐 **Constraints**

---

### 1. **Installed Capacity Constraints**

Ensure installed capacity is at least equal to max hourly generation:

$$
C^{\text{solar}} \geq \max_t(SolarHourly[t]) \\
C^{\text{wind}} \geq \max_t(WindHourly[t])
$$

---

### 2. **Renewable Energy Availability**

$$
G_t^{\text{solar}} = SolarHourly[t] \\
G_t^{\text{wind}} = WindHourly[t] \\
G_t^{\text{RE}} = G_t^{\text{solar}} + G_t^{\text{wind}}
$$

---

### 3. **Energy Balance per Hour (with transmission losses)**

$$
G_t^{\text{RE}} + G_t^{\text{grid\_in}} + B_t^{\text{out}} = \frac{1}{\eta^{\text{trans}}} \left( E_t^{\text{elyz}} + E_t^{\text{NH3}} \right) + B_t^{\text{in}} + G_t^{\text{grid\_out}}
$$

Where:

$$
E_t^{\text{NH3}} = A_t \cdot E^{\text{NH3}} \cdot 1000 \text{ (kWh)} = A_t \cdot 1 \text{ MWh/tonne}
$$

---

### 4. **Electrolyzer and H₂ Production**

$$
H_t^{\text{prod}} = \eta^{\text{elyz}} \cdot E_t^{\text{elyz}} \\
E_t^{\text{elyz}} \leq C^{\text{elyz}} \quad \text{(capacity constraint)}
$$

---

### 5. **Battery Dynamics**

$$
S_t^{\text{bat}} = S_{t-1}^{\text{bat}} + \eta^{\text{bat}} \cdot B_t^{\text{in}} - \frac{B_t^{\text{out}}}{\eta^{\text{bat}}} \\
0 \leq S_t^{\text{bat}} \leq C^{\text{bat}}
$$

---

### 6. **Hydrogen Dynamics**

$$
S_t^{\text{H2}} = S_{t-1}^{\text{H2}} + H_t^{\text{prod}} - H_t^{\text{used}} \\
0 \leq S_t^{\text{H2}} \leq C^{\text{H2\_stor}}
$$

---

### 7. **Ammonia Production**

$$
A_t = \frac{H_t^{\text{used}}}{\eta^{\text{NH3}}} \\
A_t \leq C^{\text{NH3}} \quad \text{(max hourly capacity)} \\
A_t \geq 0.2 \cdot C^{\text{NH3}} \quad \text{(min load)}
$$

---

### 8. **Annual Ammonia Production Target**

$$
\sum_{t=1}^{8760} A_t \geq A^{\text{target}} = 200{,}000
$$

---

### 9. **Grid Withdrawal Cap**

$$
\sum_{t=1}^{8760} G_t^{\text{grid\_in}} \leq 50{,}000
$$

---

## 💰 **Objective Function: Capital Cost Minimization**

$$
\min \quad 
C_{\text{total}} = 
C^{\text{solar}} \cdot c^{\text{solar}} +
C^{\text{wind}} \cdot c^{\text{wind}} +
C^{\text{elyz}} \cdot c^{\text{elyz}} +
C^{\text{bat}} \cdot c^{\text{bat}} +
C^{\text{H2\_stor}} \cdot c^{\text{H2\_stor}} +
C^{\text{NH3}} \cdot c^{\text{NH3}} +
c^{\text{fixed}}
$$

Where:

* $c^{\text{solar}}, c^{\text{wind}}, \ldots$: Capital cost per unit
* $c^{\text{fixed}}$: One-time fixed cost

---

## 🧮 Model Summary

You now have a **linear programming model** (all equations are linear), suitable for:

* Pyomo / PuLP / Gurobi / CPLEX
* MILP extensions (e.g., for binary grid operations or stepwise CAPEX)

---

## 🧭 Next Steps

Would you like me to now:

1. ✅ Generate this in **code form** (Pyomo or PuLP)?
2. 📄 Create a clean **PDF version** of this formulation?
3. 🧪 Set up a starter **Jupyter Notebook with dummy hourly data**?

Let me know what output you'd like!
