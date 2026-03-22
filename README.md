# 🧠 Operating Systems Exploration

![Windows](https://img.shields.io/badge/OS-Windows-blue)
![macOS](https://img.shields.io/badge/OS-macOS-black)
![Linux](https://img.shields.io/badge/OS-Linux-yellow)
![Shell](https://img.shields.io/badge/Shell-Bash-green)
![PowerShell](https://img.shields.io/badge/Script-PowerShell-306998)

## 🏁 Overview

This repository is a **hands-on exploration of the three major desktop operating systems** — **Windows**, **macOS**, and **Linux**. 

![Operating Systems](./images/os_lineup.png)

It documents and compares how each OS handles everyday system operations, from file management to disk utilities and GUI troubleshooting tools.

The goal is to **develop cross-platform literacy**, automate common tasks, and understand how system concepts differ across environments.

## 🗂️ Repository Structure

```
os_exploration/
|
|----images/
|----linux/
|    |----commands/
|    |    |----advanced_tools.md
|    |    |----package_managament.md
|    |----gui/
|    |    |----screenshots/
|    |    |----interface.md
|    |----scripts/
|    |    |----file_directory_management.md
|    |    |----task_management.md
|----mac/
|    |----commands/
|    |    |----advanced_tools.md
|    |    |----package_managament.md
|    |----gui/
|    |    |----screenshots/
|    |    |----interface.md
|    |----scripts/
|    |    |----file_directory_management.md
|    |    |----task_management.md
|----windows/
|    |----commands/
|    |    |----advanced_tools.md
|    |    |----package_managament.md
|    |----gui/
|    |    |----screenshots/
|    |    |----interface.md
|    |----scripts/
|    |    |----file_directory_management.md
|    |    |----task_management.md
|----README.md

```

## 🧩 Topics Covered

### 🧭 Command-Line Exploration
- File and directory management  
- Disk and filesystem operations  
- Process and resource monitoring  
- User and permission management  
- Package management and updates  

### 🖥️ GUI Interfaces
- System utilities and diagnostics  
- Task Manager (Windows)  
- Activity Monitor (macOS)  
- System Monitor (Linux)  
- Control Panel / Settings equivalents  

### ⚙️ Automation Scripts
- System cleanup (cache, temp, recycle/trash)  
- Log and configuration backups  
- Disk usage reports  
- Basic update & upgrade automation  

## 🧠 Learning Objectives

Through this exploration, I aimed to:

- Strengthen my understanding of **system internals** across different OS families.  
- Learn how to **troubleshoot and maintain** systems via command line and GUI tools.  
- Practice **scripting and automation** using `.bat`, `.ps1`, and `.sh`.  
- Build a structured documentation habit for technical learning.

## 💡 Sample Highlights

| Area | Windows | macOS | Linux |
|------|----------|--------|--------|
| Disk Usage | `wmic logicaldisk` | `df -h` | `du -sh *` |
| System Info | `systeminfo` | `system_profiler` | `uname -a` |
| Process Mgmt | Task Manager | Activity Monitor | `top` / `htop` |
| Cleanup | PowerShell scripts | Bash cleanup scripts | `apt clean && autoremove` |

## 📘 How to Use This Repo

You can browse each OS folder to view:
- **Commands**: Markdown guides explaining syntax, examples, and output.
- **GUI**: Screenshots and notes about system interfaces.
- **Scripts**: Ready-to-run maintenance and automation examples.

This repo is **educational**, not meant for production use.

## 🪄 Reflections

Building this repository helped me see how *conceptually similar yet implementation-wise different* these systems are.  
Despite using distinct tools, their goals — resource management, security, and user control — are fundamentally aligned.

## 🔗 Connect

If you found this interesting or want to collaborate on similar learning projects:

- 💼 [LinkedIn](https://www.linkedin.com/in/kavinda-rajapaksha-65034333a)
- 🧰 [GitHub](https://github.com/kavinda85346)

> 📚 *“Understanding systems deeply starts with exploring how they differ.”*


