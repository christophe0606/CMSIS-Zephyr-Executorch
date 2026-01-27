# ExecuTorch on Zephyr RTOS with CMSIS Toolbox

This example demonstrates running [ExecuTorch](https://pytorch.org/executorch/) inference on Arm Cortex-M with Ethos-U NPU acceleration using Zephyr RTOS, built and managed through the CMSIS Toolbox.

## Overview

The example runs a simple Add model on the Arm Corstone-300 FVP (Fixed Virtual Platform) with Ethos-U55 NPU. It showcases:

- ExecuTorch runtime integration with Zephyr RTOS
- Ethos-U NPU delegate for hardware acceleration
- CMSIS Toolbox for build management
- Docker-based AI layer generation workflow

## Prerequisites

### Required Tools

| Tool | Version | Purpose |
|------|---------|---------|
| [Docker](https://www.docker.com/) | Latest | AI layer build environment |
| [VS Code](https://code.visualstudio.com/) | Latest | Development environment |
| [Arm CMSIS Solution](https://marketplace.visualstudio.com/items?itemName=Arm.cmsis-csolution) | ≥1.64.0 | Build and debug integration |
| [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/getting_started/index.html) | 0.17.x | Zephyr build tools |

### VS Code Extensions

The workspace will automatically prompt to install required extensions:
- Arm CMSIS Solution
- Arm CMSIS Debugger

## Project Structure

```
├── executorch-example/          # Main application
│   ├── CMakeLists.txt           # Zephyr application CMake
│   ├── prj.conf                 # Zephyr configuration
│   ├── Kconfig                  # Application Kconfig
│   ├── boards/                  # Board overlays and FVP config
│   ├── model/                   # Model files and toolchain
│   ├── src/                     # Application source code
│   ├── ai_layer/                # Generated ExecuTorch libraries (after build)
│   └── zephyr/                  # Zephyr module
│       ├── module.yml           # Zephyr module definition
│       ├── .docker/             # Docker build environment
│       └── scripts/             # Build and workflow scripts
├── zephyr.csolution.yml         # CMSIS solution file
└── README.md                    # This file
```

## Quick Start

### Step 1: Build Docker Image

Build the Docker image containing the ExecuTorch build environment:

```bash
docker build -f executorch-example/zephyr/.docker/Dockerfile -t executorch-arm-builder .
```

### Step 2: Generate AI Layer

Run the Docker-based workflow to generate the ExecuTorch libraries and model:

```bash
docker run --rm -v $(pwd)/executorch-example:/workspace2 \
    executorch-arm-builder:latest \
    /workspace2/zephyr/scripts/local_workflow.sh
```

This generates:
- Pre-compiled ExecuTorch static libraries
- Model converted to `.pte` format and C header
- Build configuration and reports

### Step 3: Build Zephyr Application

Using VS Code with CMSIS extensions:

1. Open the workspace in VS Code
2. In the CMSIS view, open solution `zephyr.csolution.yml`
3. Select target: `AVH-SSE-300` and build type: `Debug`
4. Click **Build Solution**

Or use command line:

```bash
cbuild zephyr.csolution.yml --context executorch-example.Debug+AVH-SSE-300
```

### Step 4: Run on FVP 

You can use the Run button in the CMSIS view, to start the preconfigured FVP run. 

### Expected Output

```
I executorch:arm_executor_runner.cpp:427] method_allocator_input:    16 bytes
I executorch:arm_executor_runner.cpp:428] method_allocator_executor: 0 bytes
I executorch:arm_executor_runner.cpp:438] Model executed successfully.
I executorch:arm_executor_runner.cpp:445] Model outputs:
I executorch:arm_executor_runner.cpp:452]   output[0]: tensor scalar_type=Float numel=1
I executorch:arm_executor_runner.cpp:469]     [0] = 2.000000
I executorch:arm_executor_runner.cpp:487] SUCCESS: Program complete, exiting.
```

## VS Code Tasks

The workspace includes pre-configured tasks:

| Task | Description |
|------|-------------|
| `Docker: Build Image` | Build the Docker image |
| `Docker: Run AI Layer Generation` | Generate AI layer libraries |
| `CMSIS Load+Run` | Run on Corstone-300 FVP (local) |
| `FVP Run` | Run on Corstone-300 FVP (Docker) |

Access via **Terminal → Run Task** or the CMSIS panel.

## Customization

### Using a Different Model

1. Modify `executorch-example/model/aot_model.py` with your PyTorch model
2. Update operator list in `executorch-example/model/operators_minimal.txt`
3. Regenerate AI layer (Step 2)
4. Rebuild application (Step 3)

### Memory Configuration

Edit `executorch-example/prj.conf`:

```conf
# Stack and heap
CONFIG_MAIN_STACK_SIZE=8192
CONFIG_HEAP_MEM_POOL_SIZE=32768

# ExecuTorch memory pools
CONFIG_EXECUTORCH_METHOD_ALLOCATOR_POOL_SIZE=65536
CONFIG_EXECUTORCH_TEMP_ALLOCATOR_POOL_SIZE=262144
```
