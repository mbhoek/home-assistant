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
