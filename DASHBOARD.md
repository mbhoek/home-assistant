# Dashboard notes

## Lighting controls

Use separate controls for each room's lighting layers:

- **Direct** for downlights, spots, and ceiling lights.
- **Indirect** for uplights, strips, and ambient lighting.
- **Task** for reading, desk, and other focused lighting.

Use an existing light entity when a layer has one fixture. For multiple fixtures,
create a Light Group helper and use that group for the layer. Keep each physical
light in only one layer.

## Dashboard maintenance

- Replace unavailable or disabled entity references with active entities or
  suitable Light Group helpers.
- Confirm that devices are assigned to the intended areas before grouping them.
- Keep lights with a separate purpose as independent controls.
- After creating groups, verify the dashboard controls and any integrations that
  also manage those lights. Avoid overlapping groups that control the same bulb.
