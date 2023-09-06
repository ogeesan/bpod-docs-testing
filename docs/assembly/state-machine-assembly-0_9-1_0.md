# State Machine 0.9 - 1.0

The difference between 0.9 and 0.8 only affects the BNC input channels.

On v0.5-0.8, an optoisolator IC was used to isolate the BNC inputs (and also the wire terminal inputs, as of Bpod 0.7).

Unfortunately, that part was discontinued by the manufacturer.

For v0.9, we switched to a different, pin-compatible optoisolator IC. We provided a series resistor with a different value (1k, previously 220 ohm) and a decoupling capacitor for each IC.  From the perspective of the MATLAB/Python software these hardware versions are identical.

## Bill of Materials
<iframe width=1000 height=800 jsname="L5Fo6c" jscontroller="usmiIb" jsaction="rcuQ6b:WYd;" class="YMEQtf L6cTce-purZT L6cTce-pSzOP KfXz0b" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-popups-to-escape-sandbox allow-downloads allow-modals" frameborder="0" aria-label="Spreadsheet, Bpod r0_9 bill of materials" allowfullscreen="" src="https://docs.google.com/spreadsheets/d/13VvUd7kMTB0mf_6j1GdOC7ZPwJeu4CeL2oaa-YLmj8w/htmlembed?authuser=0"></iframe>