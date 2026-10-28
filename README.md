# pLAN

**pLAN** is a browser-based tool for visualizing physical network infrastructure on top of architectural floor plans (SVG). It maps network equipment, patch connections, and inter-rack uplinks directly to physical rooms and racks. 

The entire application is self-contained in a single HTML file and runs client-side in any modern web browser without a backend server, database, or dependencies.

---

## How to Use

1. **Open the application:** Double-click `index.html` in your browser (or host it on GitHub Pages).
2. **Load a database:** Click the prompt or use the upload icon in the header to select a configuration file (e.g. `facility_a.json` or `facility_b.json`).
3. **Navigate the map:**
   * **Pan & Zoom:** Click and drag the map background to pan. Use the mouse wheel to zoom
   * **Select a rack:** Click a rack marker on the map to inspect its installed devices in the right sidebar. Uplinks to other racks are highlighted with animated dashed lines.
   * **Move racks:** Click the pencil icon in the top-left corner of the map to enter edit mode, then drag racks to their physical locations. Click the `T` icon in the rack header to adjust label orientation.
4. **Inspect & edit ports:**
   * Click any port square to view connection details, IP address, and notes.
   * Click the pencil icon inside the port detail box to edit its description or note.
   * Click the downward arrow next to a device name to switch to the list view and view/edit device notes.
5. **Export changes:** Click the green save icon in the header to download the updated JSON file. Changes are also saved automatically to browser local storage.

---

## Database Structure

All facility data is stored in a single `.json` file. Below is the minimum required structure with all available fields:

```json
{
    "location_name": "Facility A - Logistics Hub",
    "images_folder": "racks/",
    "map_svg": "<svg viewBox=\"0 0 1100 650\"><rect x=\"50\" y=\"50\" width=\"1000\" height=\"550\" stroke=\"#2c3e50\" fill=\"none\"/></svg>",
    "rack_coordinates": {
        "server_room": {
            "x": 160,
            "y": 140,
            "label_side": "bottom",
            "label_align": "center",
            "comment": "rack door lock sticks"
        },
        "office": {
            "x": 500,
            "y": 300,
            "label_side": "top",
            "label_align": "center"
        }
    },
    "devices": [
        {
            "name": "TL-SG3428X",
            "ip": "10.10.0.2",
            "location": "server_room",
            "comment": "core distribution switch",
            "ports": [
                {
                    "port": "E01",
                    "row": 1,
                    "label": "01",
                    "connected": "Veeam Backup Repository Server",
                    "ip": "10.10.10.20",
                    "category": "server",
                    "comment": "management access on port 5301"
                },
                {
                    "port": "E02",
                    "row": 2,
                    "label": "02",
                    "connected": "Workstation Accounting",
                    "ip": "10.10.20.15",
                    "category": "pc"
                },
                {
                    "port": "SFP25",
                    "row": 2,
                    "space": true,
                    "label": "25",
                    "connected": "office switch uplink",
                    "ip": "10.10.0.3",
                    "category": "it",
                    "is_uplink": true,
                    "target_location": "office"
                }
            ]
        }
    ]
}
```

### Field Definitions

#### Top-Level
* **`location_name`** *(string)*: Name of the site displayed in the header.
* **`images_folder`** *(string, optional)*: Subfolder path for rack elevation PNG images (e.g. `"racks/"`).
* **`map_svg`** *(string)*: Architectural SVG markup containing the building walls and rooms.
* **`rack_coordinates`** *(object)*: Map of rack IDs to coordinates and settings.
* **`devices`** *(array)*: List of network devices installed across the facility.

#### Racks (`rack_coordinates[id]`)
* **`x`**, **`y`** *(number)*: Center coordinates of the rack marker in SVG viewBox space.
* **`label_side`** *(string)*: `"top"`, `"bottom"`, `"left"`, or `"right"`.
* **`label_align`** *(string)*: `"center"`, `"start"`, or `"end"`.
* **`comment`** *(string, optional)*: Optional maintenance note for the rack.

#### Devices (`devices[]`)
* **`name`** *(string)*: Device model or identifier.
* **`ip`** *(string)*: Management IP (clickable in the UI).
* **`location`** *(string)*: ID of the rack housing this device (matches key in `rack_coordinates`).
* **`comment`** *(string, optional)*: Optional device note, displayed in the port list view.
* **`ports`** *(array)*: List of physical ports on the device.

#### Ports (`devices[].ports[]`)
* **`port`** *(string)*: Technical interface name (e.g. `"E01"`, `"SFP25"`).
* **`row`** *(number)*: Row number on the switch faceplate (`1` for top, `2` for bottom).
* **`space`** *(boolean, optional)*: Set to `true` to insert a visual gap before this port (e.g. between RJ45 and SFP blocks).
* **`label`** *(string, optional)*: Text displayed above the port square. If omitted, derived from the port name.
* **`connected`** *(string)*: Description of the connected endpoint or cable destination.
* **`ip`** *(string)*: IP address of the connected device.
* **`category`** *(string)*: Functional category color:
  * `empty` — White (unconnected / reserve)
  * `it` — Green (network infrastructure / uplinks / APs)
  * `camera` — Red (CCTV surveillance)
  * `pc` — Light blue (workstations)
  * `printer` — Purple (printers / scanners)
  * `server` — Dark slate (servers / NAS)
  * `phone` — Orange (VoIP phones)
  * `machine` — Blue (PLCs / industrial machines / CNC)
  * `tablet` — Yellow (HMI operator panels)
  * `other` — Gray (miscellaneous devices)
* **`comment`** *(string, optional)*: Specific maintenance note for this port.
* **`is_uplink`** *(boolean, optional)*: Set to `true` to mark as an inter-rack trunk line.
* **`target_location`** *(string, optional)*: Rack ID to connect uplink lines to on the map.
