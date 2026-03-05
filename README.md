# Home Assistant Battery Charging Automation

## Overview
This project automates the charging of a GoodWe home battery system using Home Assistant OS. By combining dynamic energy tariffs from Octopus Energy with solar generation forecasts from Solcast, the system uses the **Predbat** integration to intelligently calculate the optimal times to charge the battery from the grid, ensuring maximum cost savings. 

Since there is no export tariff activated for this site, the system is strictly configured to **Control charge**, capturing excess solar and utilizing cheap grid blocks without force-discharging to the grid.

## Integrations & Add-ons Used
* **Home Assistant OS** (Core platform)
* **HACS** (Home Assistant Community Store)
* **MQTT** (Message broker required for Predbat)
* **SSH & Web Terminal / File Editor** (For configuration editing)
* **GoodWe** (Inverter integration)
* **Octopus Energy** (Import tariff data)
* **Solcast Solar** (Solar generation forecasting)
* **Predbat** (Predictive battery charging optimization algorithm)

---

## Setup & Installation Steps

### 1. Connect the GoodWe Inverter
* Installed the GoodWe integration in Home Assistant.
* Connected to the inverter locally using its IP address.
* Verified that battery state of charge (SoC) and control entities were actively reporting.

### 2. Configure Solcast for Solar Prediction
* Created a rooftop site profile on the Solcast website with the array's specifications.
* Generated an API key.
* Installed the Solcast integration in HA, inputted the API key, and successfully pulled in the generation forecast data.

### 3. Integrate Octopus Energy
* Installed the Octopus Energy integration.
* Authenticated the account to pull in current and upcoming grid import tariff rates so Predbat knows exactly when electricity is cheapest.

### 4. Install Predbat via HACS
* Installed the Predbat integration through HACS.
* Set up the MQTT broker, which Predbat relies on to communicate entity states and predictions within Home Assistant.

### 5. Custom Inverter Template & Troubleshooting
* **The Challenge:** The default Predbat installation did not have a built-in template that perfectly mapped to this specific GoodWe setup.
* **The Solution:** Used the File Editor and SSH to create and edit a custom inverter template. Hand-mapped the correct `entity_id`s for the inverter controls (resolving missing entity errors like `charge_limit`).
* **Troubleshooting:** Actively monitored Home Assistant and AppDaemon logs to identify configuration warnings, adjusting the custom template until Predbat could successfully read and write to the inverter.

---

## Current Status
**Status: All Set & Operational.** Predbat is successfully reading the solar forecast and tariff rates, calculating the optimal charging plan, and directly controlling the GoodWe inverter to minimize grid import costs.
