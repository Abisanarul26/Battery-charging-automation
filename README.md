# Home Assistant Battery Charging Automation (GoodWe + Predbat)

## Overview
This project automates the charging and discharging behavior of a **GoodWe hybrid inverter battery system** using **Home Assistant OS**.

The system integrates:

- Dynamic grid tariff data
- Solar generation forecasts
- Predictive battery optimization

to intelligently control when the battery should charge, discharge, or remain idle.

The optimization engine is **Predbat**, which calculates the optimal charging schedule based on:

- Solar forecast from **Solcast**
- Dynamic electricity pricing from **Octopus Energy**
- Current battery state of charge

However, **Predbat does not natively support GoodWe inverters**, so a custom control layer was implemented using **Home Assistant automations** that interpret Predbat's plan and translate it into GoodWe EMS commands.

The system ensures:

- Battery charges during **cheap tariff periods**
- Solar surplus is stored instead of exported
- **Grid export is minimized to near zero** (since export tariff is not enabled)

---

![Predicted charging plan](https://github.com/Abisanarul26/Battery-charging-automation/blob/main/images/charging_plan.png)

---

# Integrations & Add-ons Used

### Core Platform
- Home Assistant OS

### Add-ons
- HACS (Home Assistant Community Store)
- MQTT Broker
- SSH & Web Terminal
- File Editor

### Integrations
- GoodWe Inverter
- Octopus Energy
- Solcast Solar
- Predbat

---

# System Architecture

```
Solcast Forecast
        │
        ▼
Octopus Tariff Data
        │
        ▼
     Predbat
 (Charging Optimization)
        │
        ▼
 Home Assistant Automations
 (Custom Control Layer)
        │
        ▼
  GoodWe EMS Mode Control
```

Predbat calculates the **charging plan**, while **Home Assistant automations execute the plan on the inverter**.

---

# Setup & Installation

## 1. Connect the GoodWe Inverter
- Installed the **GoodWe integration** in Home Assistant.
- Connected to the inverter locally using its **IP address**.
- Verified that entities such as:

  - Battery State of Charge (SoC)
  - Battery Power
  - House Load
  - PV Generation

were correctly reporting in Home Assistant.

---

## 2. Configure Solcast Solar Forecast

1. Created a **rooftop site profile** in Solcast.
2. Entered system parameters:
   - Panel capacity
   - Tilt
   - Azimuth
3. Generated an **API key**.
4. Installed the **Solcast integration** in Home Assistant.
5. Confirmed that forecast entities were updating successfully.

---

## 3. Integrate Octopus Energy Tariff

1. Installed the **Octopus Energy integration**.
2. Authenticated the account.
3. Imported dynamic **electricity price data**.

This allows Predbat to identify **low-cost charging windows**.

---

## 4. Install Predbat via HACS

Predbat was installed using **HACS**.

Steps:

1. Install Predbat repository.
2. Install **MQTT Broker** add-on.
3. Configure MQTT connection for Predbat.

Predbat then creates prediction entities including:

- Charging periods
- Demand periods
- Hold periods
- Predicted battery behavior

---

## 5. Custom Inverter Template

### Problem
Predbat does **not natively support GoodWe inverters**.

Therefore, the default inverter template did not match the available GoodWe entities.

Example issue:

```
Missing entity: charge_limit
```

### Solution
A **custom inverter template** was created by:

- Accessing the Home Assistant filesystem using **SSH**
- Editing configuration files via **File Editor**
- Mapping GoodWe entities manually

Predbat was then able to read:

- Battery state
- Charging status
- System load

and trigger control actions through Home Assistant automations.

---

# Automation Control Layer

To bridge Predbat's charging plan with the GoodWe inverter, **9 Home Assistant automations** were developed.

These automations interpret Predbat's status signals and dynamically control the inverter's **EMS mode and power limits**.

---

# Automation Logic

## 1. Battery Charging Mode
This automation switches the inverter **EMS mode to Charge** when Predbat indicates a charging window.

Purpose:

- Charge the battery during **low electricity price periods**
- Reach the **target state of charge** before peak tariffs

---

## 2. Battery Discharging Mode
This automation activates when Predbat enters a **Demand period**.

Action:

- Set inverter **EMS mode to Discharge**
- Battery supplies the home load instead of importing expensive grid electricity.

---

## 3. Battery Standby Mode
When Predbat signals **Hold Charging**, the inverter is switched to **Standby**.

Purpose:

- Maintain current battery energy
- Prevent unnecessary charging or discharging

---

## 4. Fixed Charging EMS Limit
During scheduled charging periods, the EMS **charge power limit is set to a fixed value**.

Purpose:

- Utilize the **maximum allowable charging rate**
- Capture as much cheap electricity as possible during the tariff window.

---

## 5. Dynamic Discharge EMS Limit
This automation dynamically adjusts the **battery discharge power limit**.

Goal:

```
Grid export ≈ 0 W
```

Since the system does **not receive export payments**, discharging is limited so that battery output only covers **home consumption**.

---

## 6. PV Surplus Override Charging
If the inverter is in **Demand or Hold mode**, but:

```
PV generation > House Load
```

then the system temporarily switches the inverter to **Charge mode**.

Purpose:

- Capture **excess solar energy**
- Prevent exporting free energy to the grid.

---

## 7. Surplus-Based Charging Power Control
When PV surplus charging is active, this automation dynamically adjusts the **EMS charge limit**.

Charge power is set equal to:

```
PV generation - House load
```

This ensures:

- Battery absorbs **only the surplus power**
- **Grid import is avoided**

---

## 8. EMS discharge power limit zero during solar surplus threshold
When PV surplus charging is active, this automation set the ems dicharge limt to zero until the **Surplus-Based Charging Power Control** automation triggers.

Discharge power is set zero; this ensures unnecessary battery power to be exported.

This automation ensures the spikes formation during mode switching frequently, by ensuring the threshold.

## 9. EMS Mode Reset
When the PV surplus condition ends, the inverter mode is **restored to Predbat's original plan**.

Example:

If Predbat planned **Demand mode**, the system switches back from temporary charging to discharging.

This ensures the **optimization strategy remains intact**.

---

# Key System Behavior

The automation system ensures:

### Cost Optimization
Battery charges during **cheap tariff periods**.

### Solar Utilization
Excess PV is stored instead of exported.

### Export Avoidance
Grid export is minimized to **near zero**.

### Predbat Compatibility
Predbat predictions are translated into **GoodWe-compatible commands**.

---

# Current Status

**Status: Fully Operational**

The system is successfully:

- Reading **solar forecasts**
- Importing **dynamic electricity tariffs**
- Generating **Predbat charging plans**
- Executing the plan via **Home Assistant automations**
- Dynamically controlling the **GoodWe inverter EMS mode**

This results in **fully automated cost-optimized battery management**.
