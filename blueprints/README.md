# Blueprints

## Room lighting (`automation/room_lighting/room_lighting.yaml`)

One automation per room. It controls up to three lighting layers from buttons and an optional motion sensor. Adaptive Lighting keeps handling brightness and colour temperature.

| Layer | Use for |
|---|---|
| **Direct** | Downlights, spots, ceiling lights |
| **Indirect** | Uplights, strips, ambient/bounce light |
| **Task** | Reading, desk and other focused lights |

When a layer has several bulbs, select the room/category **Light Group** helper rather than the individual bulbs (see [DASHBOARD.md](../DASHBOARD.md)).

### Behaviour

| Input | Action |
|---|---|
| Main button, short press | Room off → **default layers** on. Room on → everything off. |
| Main button, short press **within 1 s** of the last light change | Cycle **Indirect → Indirect + Direct → All → Task → Indirect …** Layers the room doesn't have are skipped. So press once to turn on, then keep tapping to step through scenes. The window can be changed per room (*Multi-press window*). |
| Main button, long press / hold | Room off → default layers on. Otherwise dim one step per event; at the lowest step it jumps back to 100%. |
| Brighten / Dim buttons | Step brightness up or down. Brighten turns the default layers on when the room is off. |
| Off buttons | Whole room off. |
| Motion | If the room is off, no manual override is active and it is darker than the lux threshold: **Indirect** on. Rooms without Indirect lights use the default layers. |
| No motion for *N* minutes (default 5) | Room off, unless the manual override is on. |
| Room switched off (any source, including the dashboard) | Clears the manual override and resets Adaptive Lighting's manual-control flags. |

Using a button switches on the room's **manual override** helper. While the override is on, motion won't switch the room off. The override clears when the room goes off, or after the override timeout (default 120 min). After a timeout, the room switches off if there is no motion.

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
| Default layers | Indirect, Direct |
| Main buttons | Select the room's `event.*` button entities |
| Motion sensor | Select the room's motion sensor (optional) |
| Lux sensor | Select the room's illuminance sensor (optional) |
| Manual override helper | Create a room-specific `input_boolean` helper |
| Adaptive Lighting switch | Select the room's Adaptive Lighting switch (optional) |

Keep lights with a separate purpose outside the layer groups and control them separately.

### Notes and limitations

- **Scene cycling uses the lights' `last_changed` time**, which only moves when a light switches on or off; Adaptive Lighting adjustments don't affect it. No helper is needed, and it works on remotes without a native double press, such as the Hue Smart Button. Pressing the button within 1 s after motion switched the lights on also cycles instead of switching off. If your lights report their state slowly, increase the window.
- **Light groups.** `light.turn_on` on a group switches all its members. Make sure no bulb appears in more than one layer.
- **Adaptive Lighting.** Dimming makes AL mark lights as manually controlled (when *take over control* is enabled). They return to adaptive control when the room is switched off.
- **Without an override helper,** motion will also switch off lights that you turned on with a button.

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

Night Mode uses an `input_boolean` helper as the single source of truth for the house state. The in-bed sensor only turns the helper on; it never turns it off. Getting up during the night therefore does not end Night Mode. Turn the helper off from a dashboard or press a configured exit button when you want to leave Night Mode.

When the helper turns on, the automation enables the selected Adaptive Lighting sleep-mode switches and turns off the selected lights. When the helper turns off, it disables those sleep-mode switches. Lights are not automatically restored.

Setup:

1. Create an `input_boolean` helper for Night Mode.
2. Create an automation from the **Night Mode** blueprint.
3. Select the helper, in-bed sensor, all rooms' Adaptive Lighting sleep-mode switches, and the household lights to turn off.
4. Add the helper to a dashboard if you want a convenient manual way to end Night Mode.
5. Optionally select bedside remote `event.*` entities under **Exit buttons**. Press the remote and inspect its `event_type` attribute in *Developer tools → States*, then set **Exit press event types** to the matching value (default `short_release`; custom values are supported). These inputs expect event entities, not `switch.*` or `button.*` entities.

A matching exit-button event turns off the Night Mode helper, which disables the selected sleep-mode switches without turning lights on. Attribute-only updates and transitions from or to `unknown` or `unavailable` are ignored. Leave **Exit buttons** empty to retain dashboard-only exit control. After updating an existing blueprint, reload automations and edit its automation to configure the new inputs, then test the remote in Home Assistant.

Exiting Night Mode does not block future bed-sensor activation: a later off-to-on transition of the in-bed sensor can enter Night Mode again.

Use the helper as the canonical mode state: sensors and buttons should request a transition by changing the helper, while mode-specific automations react to its on/off transitions. This keeps triggers separate from the effects and allows future modes to follow the same pattern. Away Mode is not included yet; choose a presence source and decide how returning home affects the mode before adding it.
