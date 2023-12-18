# STM32CubeMX 配置

STM32CubeMX新工程配置备忘。

## 步骤

1. 新建工程，选择MCU

2. 配置DEBUG端口

   <img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230123202034720.png" alt="image-20230123202034720" style="zoom:50%;" />

3. 配置RCC晶振

   <img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230123202126917.png" alt="image-20230123202126917" style="zoom:50%;" />

4. 配置时钟树

   <img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230123202332299.png" alt="image-20230123202332299" style="zoom:50%;" />

5. 配置USB
6. 配置其他外设
7. 配置工程信息，使用Makefile、为每个外设生成源文件
8. 生成整个工程

然后，去其他工程下把`openocd.cfg`、`STM32H7xxxx.svd`复制到工程目录下。

在VSCode下配置，task.json：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make",
            "group": "build",
            "type": "shell",
            "command": "make all -j 16",
            "problemMatcher": {
                "owner": "cpp",
                "fileLocation": "autoDetect",
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        },
        {
            "label": "make clean",
            "group": "build",
            "type": "shell",
            "command": "make clean"
        },
        {
            "label": "flash",
            "group": "build",
            "type": "shell",
            "command": "openocd -f openocd.cfg -c \"program build/注意这里是你的工程名.elf preverify verify reset exit\""
           
        }
    ]
}
```

和launch.json：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceFolder}",
            "executable": "./build/f411.elf",
            "request": "launch",
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "servertype": "openocd",
            "showDevDebugOutput": "parsed",
            "configFiles": [
                "openocd.cfg"
            ],
            "svdFile": "这里是你的SVD文件.svd",
            "device": "stlink",
            "preLaunchTask": "make all"
        }
    ]
}
```





