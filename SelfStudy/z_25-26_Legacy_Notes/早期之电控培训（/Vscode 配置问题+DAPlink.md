---

---
- tasks.json

```javascript
{
  "version": "2.0.0",
  "tasks": [
    {      
        "label": "flash-openocd",
        "type": "shell",
        "command": "openocd",
        "args": [
            "-f",
            "interface/cmsis-dap.cfg",
            "-f",
            "target/stm32f4x.cfg",
            "-c",
            "program build/Debug/RM26_F4.elf reset exit"
        ],
        "problemMatcher": []
    }
  ]
}
```

- c_cpp_properties.json

```json
{
    "configurations": [
        {
            "name": "STM32",
            "compileCommands": "${workspaceFolder}/build/Debug/compile_commands.json"
        }
    ]
}
```

- launch.json

```javascript
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        
        {
            "type": "stlinkgdbtarget",
            "request": "launch",
            "name": "STM32Cube: STM32 Launch ST-Link GDB Server",
            "origin": "snippet",
            "cwd": "${workspaceFolder}",
            "preBuild": "${command:st-stm32-ide-debug-launch.build}",
            "runEntry": "main",
            "imagesAndSymbols": [
                {
                    "imageFileName": "${command:st-stm32-ide-debug-launch.get-projects-binary-from-context1}"
                }
            ]
        },
        {
            "name": "STM32 Debug (DAP)",
            "type": "cortex-debug",
            "request": "launch",
            "servertype": "openocd",
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/build/Debug/RM26_F4.elf",
            "configFiles": [
                "${workspaceFolder}/daplink.cfg"
            ],
            "runToEntryPoint": "main",
            "device": "STM32F407IGHx",
            "liveWatch": {
                "enabled": true,
                "samplesPerSecond": 4
            },
            // "svdFile": "${workspaceFolder}/STM32H7.svd"
        },

    ]
}



```

- settings.json

```javascript
{
    "cmake.cmakePath": "cube-cmake",
    "cmake.configureArgs": [
        "-DCMAKE_COMMAND=cube-cmake"
    ],
    "cmake.preferredGenerators": [
        "Ninja"
    ],
    "stm32cube-ide-clangd.path": "cube",
    "stm32cube-ide-clangd.arguments": [
        "starm-clangd",
        "--query-driver=${env:CUBE_BUNDLE_PATH}/gnu-tools-for-stm32/13.3.1+st.9/bin/arm-none-eabi-gcc.exe",
        "--query-driver=${env:CUBE_BUNDLE_PATH}/gnu-tools-for-stm32/13.3.1+st.9/bin/arm-none-eabi-g++.exe"
    ],
    "cortex-debug.gdbPath.windows": "D:\\RM\\ToolChain\\OpenOCD\\arm-none-eabi-gdb.exe",
    "cortex-debug.JLinkGDBServerPath": "D:\\RM\\ToolChain\\SEGGER\\JLink_V722b\\JLinkGDBServerCL.exe",
    "cortex-debug.gdbPath": "D:\\RM\\ToolChain\\arm-none-eabi-gdb.exe",
}
```