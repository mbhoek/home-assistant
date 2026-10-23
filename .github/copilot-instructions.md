# Repository guidance

## Scope and documentation

This repository contains reusable Home Assistant automation blueprints, not a complete Home Assistant configuration. `README.md` is the repository overview; `blueprints/README.md` is the source of truth for importing and configuring the room-lighting blueprint and its expected behavior.

Never put personal information, instance URLs, tokens, or other secrets in tracked files; use non-sensitive placeholders in examples.

## Build, test, and lint

There are no repository-defined build, test, or lint commands or test suite. The room-lighting blueprint is a standalone YAML artifact; there is no single-test command. Its documented setup and behavior are in `blueprints/README.md`.

## Architecture and conventions

- `blueprints/automation/room_lighting/room_lighting.yaml` implements the reusable **Room lighting** automation blueprint. Users import it into Home Assistant and create a separate automation for each room, supplying that room's entities and options. Keep room-specific entity IDs and configuration out of the blueprint.
- The blueprint divides its inputs into Lights, Buttons, Motion, and Manual override & Adaptive Lighting sections. Preserve the input selectors, defaults, and optional-input behavior when changing these sections; optional entity selectors may be empty, and the templates normalize selectors before using entity lists.
- Lighting is organized into Direct, Indirect, and Task layers. Button actions use the selected default layers, quick-repeat scene cycling, and dedicated dim/off controls; motion turns on ambient lighting (or the default layers when no ambient layer exists). Keep those behaviors and their layer fallback consistent with `blueprints/README.md`.
- Triggers are routed by their `id` through the action `choose` branches. Shared template variables derive the room's available/on layers, button event type, lux eligibility, and override state. Prefer extending this existing flow over adding parallel or room-specific logic.
- Adaptive Lighting is responsible for brightness and color temperature. The blueprint only resets its manual-control state when the room turns off. Manual override prevents motion from switching off button-controlled lights and is cleared on room-off or timeout; preserve these lifecycle rules.
- Button event types vary by device, so use the configurable short-press and long-press event-type inputs rather than assuming a single remote's events.
