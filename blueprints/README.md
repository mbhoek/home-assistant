# Blueprints

## Room lighting (`automation/room_lighting/room_lighting.yaml`)

One automation per room. It controls up to three lighting layers from Hue buttons and an optional motion sensor. Adaptive Lighting keeps handling brightness and colour temperature, so there are no brighten or dim controls.

| Layer | Use for |
|---|---|
| **Direct** | Downlights, spots, ceiling lights |
| **Indirect** | Uplights, strips, ambient/bounce light |
| **Task** | Reading, desk and other focused lights |

When a layer has several bulbs, select the room/category **Light Group** helper rather than the individual bulbs (see [DASHBOARD.md](../DASHBOARD.md)).

### Behaviour

| Input | Action |
|---|---|
| Main button, short press | Steps to the next **light mode**: **1 Indirect → 2 Indirect + Direct → 3 Indirect + Direct + Task → Off**, then back to 1. Layers the room doesn't have are skipped, and repeated modes are merged. If the room is on in a combination that is not one of the modes (for example only Direct, switched on from the dashboard), the next press switches it off. |
| Main button, long press | Room off. |
| Motion | If the room is off, no manual override is active and it is darker than the lux threshold: **mode 1** (Indirect) on. |
| No motion for *N* minutes (default 5) | Room off, unless the manual override is on. |
| Room switched off (any source, including the dashboard) | Clears the manual override and resets Adaptive Lighting's manual-control flags. |

**Hue button events.** Home Assistant's Hue integration knows `initial_press`, `repeat`, `short_release`, `long_press`, `long_release` and `double_short_release`, but each device only sends a subset. The entity's `event_types` attribute (*Developer tools → States*) lists what yours supports, and `event_type` shows the last one received. A short press is `short_release`. A hold normally gives `initial_press`, then `repeat` events while held, then `long_release`; some setups also send `long_press`. The blueprint switches the room off on the first `long_press` or `repeat`, which is safe because switching off can be repeated. Both lists can be changed per room in the blueprint inputs.

**Manual override.** Motion normally switches a room off after *N* minutes without motion. That would turn off a light you chose with a button, for example while you sit still reading. So using a button switches on the room's **manual override** helper, and while it is on motion never switches the room off. The override clears only when the room goes off, so lights you set with a button stay on until you switch them off (long press, or cycle past the last mode). There is deliberately no timeout. Without a motion sensor the override does nothing, so rooms without one don't need the helper.

### Setup

1. **Import the blueprint**
   - From GitHub, once this repo is pushed: *Settings → Automations & scenes → Blueprints → Import blueprint* with the file's GitHub URL.
   - Or copy `room_lighting.yaml` to `/config/blueprints/automation/room_lighting/` and reload automations.
2. **Create an override helper** for each room that has a motion sensor: *Settings → Devices & services → Helpers → Toggle*.
3. **Find your button events.** Buttons must be `event.*` entities (Hue, Zigbee2MQTT and Matter remotes provide these). Press each button and check *Developer tools → States* for the entity's `event_type` attribute. Hue uses `initial_press`, `short_release`, `double_short_release`, `repeat`, `long_press` and `long_release`.
4. **Create the automation** from the blueprint (*Create automation → Room lighting*) and fill in the room's inputs.

### Example configuration

| Input | Value |
|---|---|
| Direct lights | Select the room's direct-light entities |
| Indirect lights | Select the room's indirect-light entities |
| Main buttons | Select the room's `event.*` button entities |
| Motion sensor | Select the room's motion sensor (optional) |
| Lux sensor | Select the room's illuminance sensor (optional) |
| Manual override helper | Create a room-specific `input_boolean` helper |
| Adaptive Lighting switch | Select the room's Adaptive Lighting switch (optional) |

Keep lights with a separate purpose outside the layer groups and control them separately.

### Notes and limitations

- **The current mode is read from which layers are on.** A press after motion switched Indirect on therefore continues with mode 2. After switching lights on, the automation waits up to 3 s for them to report `on`, so quick repeated presses (they queue) see the right state. If your lights report their state slowly, a very fast second press may repeat a mode.
- **A layer counts as on when any of its lights is on.** A task light shared with another layer would confuse the mode detection.
- **Light groups.** `light.turn_on` on a group switches all its members. Make sure no bulb appears in more than one layer.
- **Restart.** The motion-off delay does not survive a Home Assistant restart; if motion stopped just before it, the room stays on until the next motion cycle.
- **Without an override helper,** motion will also switch off lights that you turned on with a button.
- **Night Mode wake detection.** Select the shared **Manual light-on timestamp** helper in each room that should count towards waking up. A main-button short press records a timestamp only when it turns an entirely off room on. Turning off, cycling to the next mode, motion, and dashboard/app controls do not record it.

### Upgrading from the previous version

The inputs **Default layers**, **Brighten / Dim / Off buttons**, **Dim step**, **Multi-press window** and **Manual override timeout** were removed, along with dimming. Edit the YAML of each existing automation (⋮ → *Edit in YAML*) and remove any of `default_layers`, `dim_up_buttons`, `dim_down_buttons`, `off_buttons`, `dim_step`, `multi_press_window` and `override_timeout`, otherwise the automation will refuse to load. **Long press event types** now means "switch off" and defaults to `long_press` and `repeat`. The inputs were regrouped (the override helper moved into **Motion**, Adaptive Lighting and the Night Mode timestamp into **Adaptive Lighting & Night Mode**), but their names are unchanged, so existing automations keep their values. Reload automations afterwards.

### Setup In Particular

- go to *Settings → Devices & services → Helpers*
- Create Helper *Group → Light Group*
- "Room Direct Lights" and "Room Indirect Lights" and "Room Task Lights"
- go to *Settings → Devices & services → Integrations*
- choose "Adaptive Lighting" and then "Add Entry"
- duplicate an existing room (for the same settings), adjust entities
- go to *Settings → Automations & scenes → Blueprints*
- create a new automation using the "Room lighting" blueprint
- after creation adjust the "Area" from the main list

## House modes

### Night Mode (`automation/house_modes/night_mode.yaml`)

Night Mode uses an `input_boolean` helper as the single source of truth for the house state. The in-bed sensor turns the helper on. When the bed sensor goes `off`:

- **Button already pressed:** if a room-lighting button turned an off room on since you last got into bed, Night Mode ends immediately. Getting back into bed re-arms this, so an earlier press (for example a night-time toilet trip) does not trigger the morning exit.
- **Wake window:** otherwise a wake window starts (default 10 minutes, 1-30). A button press during it ends Night Mode, and so does the window running out.
- **Cancel:** if the sensor goes back to `on` during the window, the window is cancelled and Night Mode stays on.
- **Manual on:** turning the helper on while the sensor already reads `off` starts the wake window then.

When the helper turns on, the automation enables the selected Adaptive Lighting sleep-mode switches. Once you have then been in bed for the **bedtime delay** (default 15 minutes, 1-60), it turns off the selected lights, including lights in rooms without a bed sensor. Getting out of bed cancels the delay and getting back in restarts it. Lights switched on during the delay are turned off at its end, so leave the bedroom lights out of the list if you want to keep them. When the helper turns off, it disables the sleep-mode switches. Lights are not automatically restored.

Setup:

1. Create an `input_boolean` helper for Night Mode.
2. Create an automation from the **Night Mode** blueprint.
3. Select the helper, in-bed sensor, all rooms' Adaptive Lighting sleep-mode switches, and the household lights to turn off.
4. Add the helper to a dashboard if you want a convenient manual way to end Night Mode.
5. To enable automatic wake detection, create a **Date and/or time** helper with **both date and time** enabled. Select this same helper as **Manual light-on timestamp** in Night Mode and in every room-lighting automation whose buttons should count.
6. Adjust the **Wake window** if needed (default 10 minutes). It is how long the bed sensor must stay `off` before Night Mode ends without a button press.

Requires Home Assistant 2024.10 or newer. The bedtime delay and wake window timers do not survive a Home Assistant restart: if HA restarts while the timer is running, it only starts again on the next sensor or helper change. Turning lights off at bedtime does not end Night Mode. Motion lighting and dashboard/app controls do not count as a manual light-on. A press from before the current Night Mode session, or from before you last got into bed, never counts. Unknown/unavailable bed states do not start the wake window.

Leave the timestamp helper unselected to end Night Mode only through the wake window or by turning the helper off manually.

The **Exit buttons** and **Exit press event types** inputs have been removed. If an existing automation still sets `exit_buttons`, remove it from the automation's YAML. After updating **both** blueprints, reload automations and select the shared timestamp helper in the affected automations.

Exiting Night Mode does not block future bed-sensor activation: a later off-to-on transition of the in-bed sensor can enter Night Mode again.

Use the helper as the canonical mode state: sensors and buttons should request a transition by changing the helper, while mode-specific automations react to its on/off transitions. This keeps triggers separate from the effects and allows future modes to follow the same pattern. Away Mode is not included yet; choose a presence source and decide how returning home affects the mode before adding it.
