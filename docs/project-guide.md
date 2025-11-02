# Home Assistant Automation & Configuration Project Guide

## 1. Introduction

This document outlines the goals, scope, existing hardware, desired automations, and collaboration guidelines for developing and managing automations and configurations within Home Assistant (HA) for this smart home. It serves as a central reference for both the user and the assisting AI (Gemini) throughout the project.

**Project Goal:** To create a robust, reliable, and intuitive smart home experience using Home Assistant, leveraging the existing hardware infrastructure to automate various aspects of the home environment, enhancing convenience, security, efficiency, and energy management, particularly optimizing energy costs and usage with Amber Electric data.

## 2. Existing Hardware & Integrations

This section lists the core components currently integrated or planned for integration with Home Assistant.

* **Lighting & Control:**
    * **System:** C-Bus
    * **Integration Method:** HA Core CBus Integration (interfacing with C-Gate, potentially via MQTT using tools like `cgateweb` - Ref: https://github.com/dougrathbone/cgateweb)
    * **Notes:** Need to map C-Bus Group Addresses (GAs) for lights to HA entities. Define standard naming conventions.
* **Blind Control:**
    * **System:** C-Bus Controlled Blinds
    * **Integration Method:** HA Core CBus Integration (using C-Gate, similar to lighting)
    * **Notes:** Need to map C-Bus Group Addresses (GAs) for blinds/covers to HA entities (`cover` domain).
* **Climate Control (AC):**
    * **System:** Controlled via CoolAutomation
    * **Integration Method:** CoolMasterNet Integration (or specific CoolAutomation integration if available)
    * **Notes:** Allows control and monitoring of AC units.
* **Climate Control (Heating):**
    * **System:** Gas Heating controlled via IR
    * **Integration Method:** Broadlink Integration (using Broadlink IR transmitters)
    * **Notes:** Requires learning IR codes for the heating system.
* **Sensors & Switches (General):**
    * **Technology:** Zigbee
    * **Integration Method:** Zigbee Coordinator (e.g., ZHA, Zigbee2MQTT - *Specify which one is used/preferred*)
    * **Device Types:** Motion sensors, door/window sensors, smart plugs, light switches (confirm specific types).
* **Sensors (Indoor Climate):**
    * **Technology:** Shelly WiFi Sensors
    * **Integration Method:** Shelly Integration (or MQTT if configured that way)
    * **Device Types:** Temperature & Humidity sensors.
* **Sensors (Weather):**
    * **System:** Ecowitt Weather Station
    * **Integration Method:** Ecowitt Integration
    * **Device Types:** Outdoor temperature, humidity, wind speed/direction, rain gauge, light/UV sensor.
* **Access Control (Locks):**
    * **Technology:** Z-Wave
    * **Integration Method:** Z-Wave JS UI (formerly Z-Wave JS to MQTT) or Z-Wave JS integration. (*Specify which one is used/preferred*)
    * **Device Types:** Smart door locks (specify models if known).
* **Security (Cameras):**
    * **System:** Unifi Protect
    * **Integration Method:** Unifi Protect Integration
    * **Device Types:** Cameras (specify models/locations if relevant for specific automations). Features like person/vehicle detection will be leveraged.
* **Access Control (Intercom):**
    * **System:** 2N Intercom
    * **Integration Method:** (Requires investigation - potentially via API, SIP integration like Asterisk/FreePBX, or specific HA integration if available. *Specify preferred/known method*).
    * **Device Types:** Door station, potentially indoor units. Key features: Doorbell press events, video feed, potentially triggering door release.
* **Energy (Retailer/Pricing):**
    * **System:** Amber Electric
    * **Integration Method:** Amber Electric Integration
    * **Features:** Real-time energy pricing, price forecasts, percentage of renewables in grid, grid status (demand), control signals (e.g., price spikes). Used for smart charging/discharging and appliance control.
* **Energy (Storage):**
    * **System:** Tesla Powerwall
    * **Integration Method:** Tesla Powerwall Integration
    * **Features:** Monitoring battery charge/discharge, grid status, solar production (often combined with solar integration). Control potentially influenced by Amber Electric data.
* **Energy (Solar):**
    * **System:** Enphase Solar
    * **Integration Method:** Enphase Envoy Integration
    * **Features:** Monitoring solar production, potentially individual panel data.
* **Network:**
    * **System:** Unifi Network
    * **Integration Method:** Unifi Network Integration
    * **Features:** Presence detection, device tracking.
* **Voice Assistant:**
    * **System:** Amazon Alexa
    * **Integration Method:** Alexa Integration (via Nabu Casa or manual setup)
    * **Features:** Voice control of exposed HA entities (lights, switches, scenes, scripts, climate, covers, etc.). Triggering HA automations via Alexa routines.
* **Other:** (List any other relevant hardware or services, e.g., smart appliances connected via Zigbee plugs)

## 3. Key Automation Areas

Identify the primary categories for automation development:

* **Lighting:** Automated control based on time, presence, ambient light (Ecowitt), scenes.
* **Blinds/Covers:** Automated control based on time, sun position, light levels (Ecowitt), temperature (Ecowitt/Shelly), presence, or scenes.
* **Security:** Alarm system logic, notifications, camera actions, lock management.
* **Access Control:** Intercom events, door unlocking logic, notifications.
* **Presence Detection:** Automations based on who is home or away.
* **Climate Control:** Thermostat scheduling, adjustments based on presence, indoor/outdoor temperature sensors (Shelly/Ecowitt), energy prices (Amber), or solar production. Control of AC (CoolAutomation) and Heating (Broadlink).
* **Energy Management:** Monitoring solar production (Enphase) and battery status (Powerwall). Optimizing energy usage by leveraging Amber Electric data (real-time/forecast prices, renewables %, grid conditions) to control Powerwall charging/discharging and high-consumption appliances (via smart plugs/direct integration).
* **Weather-Based Actions:** Automations triggered by rain (Ecowitt), wind speed (Ecowitt), outdoor light levels (Ecowitt), or temperature (Ecowitt).
* **Notifications:** Centralized system for important alerts (security, access, system status, energy price spikes/dips, weather alerts).
* **Scenes & Routines:** Combining multiple actions into single triggers (e.g., "Good Morning", "Movie Time", "Away", "Energy Saving Mode"), potentially triggered by Alexa.
* **Voice Control:** Exposing relevant entities and creating scripts/scenes for easy control via Alexa.

## 4. Desired Automations (Initial List)

This section will list specific automations we aim to create. We can add to this list collaboratively.

*(Examples - User to refine/add)*

* **Lighting:**
    * `automation_lights_welcome_home`: Turn on entry lights when first person arrives home after sunset (or based on Ecowitt light sensor).
    * `automation_lights_evening_scene`: Set living room lights to a specific scene at 7 PM.
    * `automation_lights_motion_hallway`: Turn on hallway light when motion detected, turn off after 2 minutes of no motion. (Specify C-Bus GAs)
    * `automation_lights_all_off_away`: Turn off all C-Bus lights when house mode changes to 'Away'.
* **Blinds:**
    * `automation_blinds_close_sunset`: Close specific C-Bus blinds (e.g., living room) at sunset.
    * `automation_blinds_open_morning`: Open specific C-Bus blinds at 8 AM on weekdays.
    * `automation_blinds_close_high_sun`: Close west-facing C-Bus blinds if Ecowitt light sensor is high and sun is in the west (afternoon) to reduce heat gain.
* **Security:**
    * `automation_security_notify_door_left_open`: Notify if a Zigbee door sensor is left open for more than 10 minutes.
    * `automation_security_arm_away`: Arm security system (virtual or linked) when house mode is 'Away' and all Z-Wave locks are locked.
    * `automation_security_camera_person_detection`: Send notification with snapshot from relevant Unifi camera when a person is detected in the backyard after 10 PM.
* **Access Control:**
    * `automation_access_notify_doorbell`: Send notification (and potentially camera snapshot) when 2N Intercom doorbell button is pressed.
    * `automation_access_unlock_front_door`: (Requires careful security consideration) Potentially allow unlocking the front Z-Wave lock via HA interface or specific secure trigger (maybe via Alexa with confirmation).
    * `automation_access_notify_lock_unlocked`: Notify when a Z-Wave lock is unlocked manually or via code.
* **Presence:**
    * `automation_presence_set_mode_away`: Set house mode to 'Away' when all tracked residents (via Unifi Network integration) are away.
    * `automation_presence_set_mode_home`: Set house mode to 'Home' when the first tracked resident arrives.
* **Climate:**
    * `automation_climate_preheat_home`: Turn on heating (Broadlink) if outdoor temperature (Ecowitt) is below X and someone is arriving home.
    * `automation_climate_turn_off_ac_away`: Turn off AC (CoolAutomation) when house mode changes to 'Away'.
    * `automation_climate_ac_precool_on_low_price`: Pre-cool house using AC (CoolAutomation) if Amber price forecast shows low prices before an expected hot period (using Ecowitt/Shelly temp sensors).
    * `automation_climate_adjust_based_on_room_temp`: Adjust AC/Heating target based on specific Shelly room sensor readings rather than just the main thermostat.
* **Energy:**
    * `automation_energy_notify_low_battery`: Send notification if Tesla Powerwall charge drops below 20%.
    * `automation_energy_charge_battery_offpeak_amber`: Prioritize charging Powerwall from the grid only when Amber Electric price is below a defined threshold (e.g., $0.15/kWh) or during negative price events.
    * `automation_energy_run_dishwasher_on_low_price_or_solar`: Run dishwasher (via smart plug) only when Amber price is low OR there is sufficient solar surplus (Enphase/Powerwall sensors).
    * `automation_energy_reduce_load_on_price_spike`: Turn off non-essential high-load devices (e.g., pool pump via smart plug, reduce AC setpoint) if Amber price spikes above a high threshold (e.g., $1.00/kWh).
    * `automation_energy_notify_amber_price_spike`: Send notification if Amber real-time price exceeds a defined threshold.
* **Weather:**
    * `automation_weather_notify_rain_start`: Send notification when Ecowitt rain sensor detects start of rain.
    * `automation_weather_close_blinds_high_wind`: Partially close C-Bus blinds if Ecowitt wind speed exceeds a certain threshold (requires careful implementation).
* **Scenes:**
    * `scene_movie_time`: Dim C-Bus lights in the living room, close living room blinds, potentially turn on AV equipment (if integrated). Triggerable by Alexa.
    * `scene_goodnight`: Turn off all lights except bedroom lamps, close all blinds, lock all Z-Wave doors, arm security system (night mode), set thermostat to night temperature. Triggerable by Alexa.

## 5. Configuration Standards & Best Practices

* **File Structure:** Automations will be stored centrally in the `/automations.yaml` file included by default in `configuration.yaml`. Other configurations (scripts, scenes, sensors, etc.) may be split into separate files or organized using packages if desired later.
* **Naming Conventions:**
    * Entities: Use clear, consistent names (e.g., `light.living_room_main`, `cover.living_room_blinds`, `sensor.hallway_motion_occupancy`, `lock.front_door`, `climate.living_room_ac`, `sensor.powerwall_battery_level`, `sensor.amber_general_price`, `sensor.ecowitt_outdoor_temperature`, `sensor.shelly_bedroom_temperature`). Follow HA defaults where possible.
    * Automations/Scripts/Scenes: Use descriptive IDs (e.g., `automation.lights_turn_on_entry_on_arrival`). Prefix with domain (e.g., `automation.`, `script.`, `scene.`). Using the UI editor for automations is recommended as it handles unique IDs automatically.
    * Helpers: Prefix with type (e.g., `input_boolean.guest_mode`, `timer.hallway_light`, `input_number.amber_low_price_threshold`).
* **YAML:** Use consistent indentation (2 spaces). Add comments (`#`) to explain complex logic or non-obvious configurations, especially if manually editing `automations.yaml`.
* **Secrets:** Use `secrets.yaml` for all sensitive information (API keys, passwords, C-Bus keys if applicable, Amber API Key, Ecowitt keys).
* **Documentation:** Add descriptions to automations and scripts within HA where possible (easily done via UI editor). Maintain this `project-guide.md` file.
* **Testing:** Test automations thoroughly after creation or modification. Use HA's tracing and debugging tools.

## 6. Collaboration Guidelines

* **Primary Tool:** This `project-guide.md` document.
* **Requesting Automations:** Refer to section 4 or add new ideas there. Be specific about triggers, conditions, and actions. Provide relevant entity IDs (e.g., Amber price sensor, Powerwall battery sensor, smart plug entity, Ecowitt light sensor, Shelly temp sensor, C-Bus cover entity).
* **Providing Information:** When requesting help, provide relevant entity IDs, device details, C-Bus GAs, and desired logic.
* **Code Generation:** AI will generate YAML code snippets or full configuration files based on requests. These snippets will be intended for inclusion in `/automations.yaml` or other relevant files.
* **Review & Implementation:** User will review generated code, adapt as necessary (especially entity IDs, GAs, price thresholds, temperature values), test, and implement in their HA configuration (often via copy/paste into the YAML editor or recreating via the UI editor).
* **Feedback:** User provides feedback on whether the generated code worked, any errors encountered, or desired modifications.
* **Updates:** Both user and AI can update this document. AI will typically update based on user feedback or new requests.

## 7. Future Ideas & Backlog

*(A place to capture ideas that aren't immediate priorities)*

* More granular appliance control based on specific Amber price forecast blocks.
* Using Amber renewables percentage data to prioritize usage during green periods.
* More sophisticated climate control logic using multiple indoor/outdoor sensors and predictive elements (e.g., based on weather forecast + Amber forecast).
* Advanced scene control based on multiple conditions (e.g., time + presence + weather + energy price).

---
*Last Updated: 2025-04-27*
