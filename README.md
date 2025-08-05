# System Monitor

A comprehensive Linux desktop system monitoring application built with C++ and Dear ImGui, providing real-time visualization of system resources including CPU, memory, network, thermal, and fan information.

## Overview

This project is a feature-rich system monitor that leverages the Linux `/proc` and `/sys` filesystems along with hardware sensors to provide detailed insights into system performance. The application uses Dear ImGui for an immediate-mode graphical user interface, offering interactive charts, tables, and controls for monitoring various system components.

## Features

### System Information Window
- **Operating System Detection**: Displays the current Linux distribution
- **User Information**: Shows logged-in username and hostname  
- **Process Overview**: Real-time count of running, sleeping, zombie, and stopped processes
- **CPU Information**: Detailed CPU model and specification display using CPUID

### Performance Monitoring (Tabbed Interface)
- **CPU Monitoring**:
  - Real-time CPU usage percentage with overlay text
  - Interactive performance graph with historical data (100 data points)
  - Configurable FPS control (1-30 FPS)
  - Adjustable Y-axis scale (100-200%)
  - Pause/Resume functionality
  - Thread-safe data collection with mutex protection

- **Thermal Monitoring**:
  - Current temperature display with overlay text and Fahrenheit conversion
  - Temperature history graph with same controls as CPU
  - Support for multiple thermal sensor paths (`/sys/class/thermal/`, `/sys/class/hwmon/`)
  - Temperature status indicators (Normal/Caution/Warning)
  - Graceful handling of systems without thermal sensors

- **Fan Monitoring**:
  - Fan status display (Active/Inactive)
  - Current fan speed (RPM) and PWM level information
  - Fan speed history visualization with configurable scale (2000-8000 RPM)
  - Speed indicators (Low/Medium/High/Stopped)
  - Multi-fan system support with hwmon detection

### Memory and Process Management
- **Memory Usage Visualization**:
  - Physical RAM usage with progress bars and percentages
  - Virtual memory (SWAP) usage display with availability detection
  - Root filesystem (/) disk usage monitoring with color-coded indicators
  - Smart unit conversion (bytes to KB/MB/GB/TB)
  - Color-coded usage levels (Green <70%, Yellow 70-90%, Red >90%)

- **Process Table**:
  - Comprehensive process listing with PID, Name, State, CPU%, Memory%
  - Sortable columns with click-to-sort functionality
  - Real-time process filtering by name (case-insensitive)
  - Multi-row selection with Ctrl+Click support
  - Color-coded process states (Running=Green, Sleeping=Blue, Zombie=Red, etc.)
  - CPU usage calculation with time-based tracking
  - Live updates every 3 seconds with cached data

### Network Monitoring
- **Interface Detection**: Automatic discovery of all network interfaces using `getifaddrs()`
- **IP Address Display**: IPv4 addresses for each detected interface
- **Statistics Tables** (Tabbed RX/TX):
  - **RX Table**: Bytes, Packets, Errors, Drops, FIFO, Frame, Compressed, Multicast
  - **TX Table**: Bytes, Packets, Errors, Drops, FIFO, Collisions, Carrier, Compressed
  - Data parsed from `/proc/net/dev` with thread-safe access
- **Visual Usage Display**:
  - Progress bars for network usage (0-2GB scale)
  - Smart unit conversion (B/KB/MB/GB) with precision handling
  - Color-coded RX (Green) and TX (Blue) visualization tabs
  - Separate tabs for statistics and usage visualization

## Technical Architecture

### Core Components
- **main.cpp**: Application entry point, SDL2/OpenGL setup, and main rendering loop
- **system.cpp**: System information gathering, CPU/thermal/fan monitoring with thread-safe data collection
- **mem.cpp**: Memory usage analysis and process management with CPU tracking
- **network.cpp**: Network interface detection and statistics monitoring
- **header.h**: Shared data structures, function declarations, and global variable definitions

### Data Structures
```cpp
struct SystemInfo {
    string os_name, hostname, username, cpu_model;
    int total_processes, running_processes, sleeping_processes;
    int zombie_processes, stopped_processes;
};

struct CPUStats {
    long long int user, nice, system, idle, iowait;
    long long int irq, softirq, steal, guest, guestNice;
};

struct Proc {
    int pid;
    string name;
    char state;
    long long int vsize, rss, utime, stime;
};

struct MemoryInfo {
    unsigned long total_ram, available_ram, used_ram;
    unsigned long total_swap, used_swap;
    unsigned long total_disk, used_disk;
};

struct ThermalInfo {
    float temperature;
    bool available;
};

struct FanInfo {
    int speed, level;
    bool active, available;
};

struct Networks {
    vector<IP4> ip4s;
};

struct RX/TX {
    int bytes, packets, errs, drop, fifo;
    // Additional fields for comprehensive network statistics
};
```

### Data Sources
- **System Information**: `/proc/stat`, `/proc/sys/kernel/hostname`, CPUID instructions
- **CPU Statistics**: `/proc/stat` for usage calculation with time-based sampling
- **Memory Data**: `/proc/meminfo`, `statvfs()` system calls for disk usage
- **Process Information**: `/proc/[pid]/stat`, `/proc/[pid]/comm` with directory enumeration
- **Network Statistics**: `/proc/net/dev`, `getifaddrs()` system calls
- **Thermal Data**: Multiple paths including `/sys/class/thermal/thermal_zone*/temp`, `/sys/class/hwmon/hwmon*/temp*_input`
- **Fan Information**: `/sys/class/hwmon/hwmon*/fan*_input`, `/sys/class/hwmon/hwmon*/pwm*`

## Prerequisites

### System Requirements
- Linux operating system (Ubuntu, Debian, CentOS, etc.)
- GCC compiler with C++11 support or newer
- SDL2 development libraries
- OpenGL support

### Dependencies
```bash
# Ubuntu/Debian
sudo apt install libsdl2-dev build-essential

# CentOS/RHEL/Fedora
sudo yum install SDL2-devel gcc-c++
# or for newer versions:
sudo dnf install SDL2-devel gcc-c++

# Arch Linux
sudo pacman -S sdl2 gcc
```

## Installation and Build

### Clone and Build
```bash
# Clone the repository
git clone https://learn.zone01kisumu.ke/git/seodhiambo/system-monitor
cd system-monitor

# Build the project
make

# Run the application
./monitor
```

### File Structure
```
system-monitor/
├── header.h                    # Shared headers and data structures
├── main.cpp                    # Main application loop and window management
├── system.cpp                  # System info, CPU, thermal, and fan monitoring
├── mem.cpp                     # Memory usage and process management
├── network.cpp                 # Network interface and statistics monitoring
├── Makefile                    # Build configuration with OpenGL loader
├── LICENSE                     # Project license
├── imgui.ini                   # ImGui configuration file
└── imgui/                      # Dear ImGui library
    └── lib/
        ├── backend/            # SDL2 + OpenGL3 backend implementation
        ├── gl3w/              # OpenGL function loader
        └── [imgui files]      # Core ImGui implementation files
```

## Usage

### Navigation
- **System Window**: Overview of system information and performance graphs
- **Memory Window**: RAM/SWAP usage and process management
- **Network Window**: Network interface statistics and usage visualization

### Interactive Controls
- **Graph Controls**: Use pause/resume buttons to freeze data collection for all monitoring tabs
- **FPS Slider**: Adjust graph update frequency (1-30 FPS) independently for CPU, thermal, and fan
- **Scale Slider**: Modify Y-axis range (CPU: 100-200%, Thermal: 60-120°C, Fan: 2000-8000 RPM)
- **Process Filtering**: Type in the filter box to search processes by name (case-insensitive)
- **Multi-Selection**: Use Ctrl+Click for multiple process selection
- **Column Sorting**: Click column headers to sort process table by different criteria

### Performance Tips
- Reduce FPS for lower CPU usage by the monitor itself
- Use pause functionality when analyzing specific time periods
- Filter processes to focus on specific applications

## Development Roadmap

### Completed Features ✅
- Basic system information display with CPUID detection
- Real-time CPU monitoring with interactive graphs and thread-safe data collection
- Memory usage visualization with color-coded indicators
- Process table with sorting, filtering, and CPU usage tracking
- Network interface detection and comprehensive statistics
- Thermal monitoring with multiple sensor support and temperature warnings
- Fan speed monitoring with PWM level display and speed categorization
- Thread-safe data collection with mutex protection
- Smart unit conversion and progress visualization

### Planned Enhancements 🔄
- [ ] Advanced process management (kill, priority changes)
- [ ] Configuration file support for customizable settings
- [ ] Export functionality for performance data
- [ ] Historical data persistence
- [ ] System alerts and notifications
- [ ] GPU monitoring support
- [ ] Custom theme support

## Performance Characteristics

### Resource Usage
- **CPU Usage**: < 5% on modern systems (configurable via FPS settings)
- **Memory Footprint**: < 50MB typical usage
- **Update Intervals**:
  - System info: Every 2 seconds
  - CPU/Thermal/Fan: Configurable (1-30 FPS, default 10 FPS)
  - Memory: Real-time on window refresh
  - Processes: Every 3 seconds with CPU data caching
  - Network: Every 2 seconds with thread-safe parsing

## Error Handling

The application gracefully handles common issues:
- Missing `/proc` filesystem entries
- Permission denied errors
- Unavailable hardware sensors (thermal, fan)
- Network interfaces going offline
- Process enumeration race conditions

When features are unavailable, the application displays "N/A" and continues monitoring available components.

## Troubleshooting

### Common Issues
- **Blank thermal data**: System may not have accessible thermal sensors
- **Missing fan information**: Fan sensors might not be available or accessible
- **Permission errors**: Some system files may require elevated privileges
- **Network statistics not updating**: Check if `/proc/net/dev` is accessible

### Debug Information
The application provides built-in status information:
- Graph data point counts and update rates
- Sensor availability status
- Thread-safe data collection status
- Process selection and filtering feedback

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-monitoring`)
3. Commit your changes (`git commit -am 'Add new monitoring feature'`)
4. Push to the branch (`git push origin feature/new-monitoring`)
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- **Dear ImGui**: Excellent immediate-mode GUI library
- **SDL2**: Cross-platform development library
- **Linux Kernel**: For providing the `/proc` and `/sys` filesystems
- **Community**: Various online resources and documentation

## Technical Notes

### Dear ImGui Integration
This project uses Dear ImGui's immediate-mode rendering approach with SDL2 + OpenGL3 backend. The application features:
- Real-time data visualization with interactive controls
- Tabbed interfaces for organized monitoring
- Sortable tables with multi-selection support
- Color-coded status indicators and progress bars
- Responsive layout with window management

### Linux Filesystem Usage
Extensive use of Linux virtual filesystems:
- `/proc`: Process and kernel information (`/proc/stat`, `/proc/meminfo`, `/proc/net/dev`)
- `/sys`: Hardware device information (`/sys/class/thermal/`, `/sys/class/hwmon/`)
- Standard system calls (`getifaddrs()`, `statvfs()`, `getpwuid()`)
- CPUID instructions for CPU identification

### Thread Safety
Robust thread-safe implementation:
- Mutex protection for shared data structures (CPU, thermal, fan, network data)
- Atomic variables for real-time status updates
- Lock guards for safe data access across threads
- Separate data collection and rendering threads

---
