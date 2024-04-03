# Visual Studio Code

Follow the recommendations below to set up features like IntelliSense and integrate Docker containers.

## Recommended Extensions

**File: `.vscode/extensions.json`**

```json
{
  "recommendations": [
    "ms-vscode-remote.remote-containers",
    "ms-vscode.cpptools",
    "ms-vscode.cmake-tools"
  ]
}
```

## Enabling IntelliSense

To add auto-complete, syntax highlighting, and other IntelliSense features for development in VSCode, set up the `c_cpp_properties.json` file in your `.vscode` directory. `.vscode/c_cpp_properties` is configured to use `./build/compile_commands.json`. If you build lantern in a different directory, make sure to update `.vscode` config appropriately in order to have IntelliSense working.

**File: `.vscode/c_cpp_properties.json`**

```json
{
  "configurations": [
    {
      "name": "Linux",
      "includePath": ["${workspaceFolder}/**"],
      "defines": [],
      "cStandard": "c99",
      "cppStandard": "c++11",
      "compileCommands": "./build/compile_commands.json",
      "intelliSenseMode": "linux-clang-x64"
    }
  ],
  "version": 4
}
```

## Integrating Docker Containers

Leverage the Dev Containers extension in VSCode and the provided configuration file to seamlessly use Docker containers for your development setup.

**File: `.devcontainer/devcontainer.json`**

```json
{
  "name": "Lantern Dev",
  "dockerFile": "../Dockerfile.dev",
  "context": "..",
  "runArgs": [
    "--cap-add=SYS_PTRACE",
    "--security-opt",
    "seccomp=unconfined",
    "-e",
    "POSTGRES_PASSWORD=postgres"
  ],
  "mounts": [
    "source=${localWorkspaceFolder},target=/lantern,type=bind,consistency=cached"
  ],
  "overrideCommand": false,
  "customizations": {
    "vscode": {
      "extensions": ["ms-vscode.cpptools", "ms-vscode.cmake-tools"],
      "settings": {
        "terminal.integrated.shell.linux": "/bin/bash"
      }
    }
  }
}
```
