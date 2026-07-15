# robot_description_Mahmoud-Hassan

https://github.com/user-attachments/assets/97bc0ee6-c840-4d55-a362-c81c9f5a8efb

## Project Overview

This package contains a `xacro`-based URDF description of a two-wheeled robot whose chassis is built as a **3-step pyramid**, with each step colored to match the Egyptian flag (black, white, red — widest to narrowest, bottom to top). The **camera** is mounted on the middle (white) step and the **LIDAR** sits on top of the highest (red) step; both are recolored **gold** as an accent that echoes the golden eagle on the flag. The base is propelled by **two continuous-joint wheels** in a differential-drive layout.

The file demonstrates several core `xacro`/URDF concepts:

- **`xacro:property`** — reusable named constants (dimensions, masses, colors) so the whole robot can be resized or recolored by editing values in one place.
- **`xacro:arg`** — an external, launch-time-configurable argument (`mesh_path`) used to locate mesh files without hardcoding a package path.
- **`${...}` expressions** — inline math (e.g. computing inertia tensors, stacking offsets, and mirrored wheel positions) evaluated at xacro-processing time.
- **`xacro:macro`** — a parameterized wheel definition (`prefix`, `y_reflect`) instantiated twice to generate the left and right wheels without duplicating XML.
- **Link/joint hierarchy** — a kinematic tree starting at `base_footprint` (a virtual, massless reference frame at ground level) and branching into the chassis, sensors, and wheels.
- **Fixed vs. continuous joints** — `fixed` joints rigidly attach the steps, camera, and lidar to the chassis; `continuous` joints (with a Y-axis rotation) drive the two wheels.
- **Inertial properties** — mass, center of mass, and inertia tensors are defined per-link (computed via standard box/cylinder inertia formulas) so the model is ready for physics simulation, not just visualization.
- **Optical frame convention** — a separate `camera_optical_link`, rotated from `camera_link`, follows the standard ROS convention of aligning Z-forward/X-right for use with vision and perception pipelines.

## Robot Structure

```
base_footprint (virtual root, ground-level reference frame)
 └── base_link                    [Step 1 — BLACK, largest]
      ├── middle_step_link        [Step 2 — WHITE, mounted on top of Step 1]
      │    └── camera_link        [GOLD — mounted on front edge of Step 2]
      │         └── camera_optical_link
      │    └── top_step_link      [Step 3 — RED, mounted on top of Step 2]
      │         └── lidar_link    [GOLD — mounted on top of Step 3]
      ├── left_wheel_link         [continuous joint, Y-axis]
      └── right_wheel_link        [continuous joint, Y-axis]
```

| Link | Role | Color | Joint type to parent |
|---|---|---|---|
| `base_footprint` | Virtual ground reference | — | — |
| `base_link` | Step 1 / chassis base | Black | Fixed (to `base_footprint`) |
| `middle_step_link` | Step 2 | White | Fixed (to `base_link`) |
| `top_step_link` | Step 3 | Red | Fixed (to `middle_step_link`) |
| `camera_link` | ZED camera | Gold | Fixed (to `middle_step_link`) |
| `camera_optical_link` | Optical frame | — | Fixed (to `camera_link`) |
| `lidar_link` | LIDAR sensor | Gold | Fixed (to `top_step_link`) |
| `left_wheel_link` | Drive wheel | Bronze | Continuous (to `base_link`) |
| `right_wheel_link` | Drive wheel | Bronze | Continuous (to `base_link`) |

> **Note:** with only two wheels and no caster, this base is not statically stable on its own — it needs either a passive caster/support point or active balance control to stay upright in a physics simulation.

## Folder Structure

```
my_robot_description/
├── urdf/
│   └── robot.urdf.xacro          # main xacro file described above
├── meshes/
│   ├── lidar.STL                 # LIDAR visual mesh
│   └── zed.stl                   # ZED camera visual mesh
└── CMakeLists.txt                # installs urdf/ and meshes/ so they're locatable via package:// paths
```

**`CMakeLists.txt`** should install the `urdf` and `meshes` folders so downstream tools (RViz, Gazebo, the VS Code visualizer, etc.) can resolve `package://my_robot_description/...` paths:

```cmake
cmake_minimum_required(VERSION 3.8)
project(my_robot_description)

find_package(ament_cmake REQUIRED)

install(
  DIRECTORY urdf meshes
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
```

## Previewing the Robot in VS Code (without ROS)

You can preview the robot directly in VS Code using the **URDF Visualizer** extension, with no ROS installation required:

**Extension:** [URDF Visualizer](https://marketplace.visualstudio.com/items?itemName=morningfrog.urdf-visualizer)

### Setup

1. Open your workspace (the folder containing `my_robot_description`) in VS Code.
2. Press `Ctrl + Shift + P` → select **Preferences: Open Workspace Settings (JSON)**.
3. Add a `urdf-visualizer.packages` entry mapping your package name to its path on disk:

   ```json
   {
     "urdf-visualizer.packages": {
       "my_robot_description": "/root/workspaces/my_robot_ws/src/robot_description_Mahmoud-Hassan"
     }
   }
   ```

   The path must point to the root of your `my_robot_description` package (the folder that directly contains `urdf/` and `meshes/`), so that `package://my_robot_description/...` references inside the xacro file resolve correctly.

### Previewing

1. Open `urdf/robot.urdf.xacro` in the editor.
2. On the right-hand side of the file's tab bar, click the **Preview URDF/Xacro** icon.
3. The extension will process the xacro file and render the robot — chassis steps, camera, lidar, and wheels — in an interactive 3D view as shown in the above demo.
4. The stl files can also be viewed inside VS code using the following extension **Extension:** [stl Visualizer] (https://marketplace.visualstudio.com/items?itemName=mtsmfm.vscode-stl-viewer)
