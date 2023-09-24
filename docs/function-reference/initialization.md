# Initialization

### `Bpod()`
**Description**

Initializes Bpod and creates a global object representing the Bpod device (`BpodSystem`) in the base workspace.

- The function automatically searches all available serial ports and finds Bpod if one is connected.
- It then creates a `BpodSystem` object.
