# Config Reader

A strongly typed, header only Lua-based config file reader for C++ 11. Supports runtime reloading of parameter values on config file save.

[![Build Status](https://travis-ci.com/ut-amrl/config-reader.svg?token=rBLDT1qXfkKmkTerGLzY&branch=master)](https://travis-ci.com/ut-amrl/config-reader)

# Example Usage

```C++
#include <csignal>
#include <iostream>

#include "config_reader/config_reader.h"

bool running = true;

void SigIntHandler(int signum) { running = false; }

int main() {
  signal(SIGINT, SigIntHandler);

  CONFIG_INT(test_int, "testInt");
  CONFIG_FLOAT(test_float, "tree.stree.number");
  CONFIG_STRING(test_string, "testString");
  config_reader::ConfigReader reader({"config.lua", "config2.lua"});
  while (running) {
    int local_int = CONFIG_test_int;
    std::cout << local_int << std::endl;
    if (local_int < 42) {
      std::cout << "It's less than 42!" << std::endl;
    }
    std::cout << CONFIG_test_float << std::endl;
    std::cout << CONFIG_test_string << std::endl;
  }
  return 0;
}
```

# Dependencies

 - Lua 5.1 Development Package
 - C++ compiler with C++ 11 support (e.g. `clang++`)

 Ubuntu packages can be installed via `InstallPackages`.

 # Currently Supported C++ Types

  - `int`
  - `unsigned int`
  - `double`
  - `float`
  - `std::string`
  - `bool`
  
 # Inotify Limits
 
 The config reader library uses inotify file watches to automatically re-load configurations. It is common to have a low limit on the number of concurrent inotify watches. Under such circumstances, the config reader will fail to add watches with the following error:
```
ERROR: Couldn't add watch to the file: config.lua
Reason: No space left on device
```
The current limit can be viewed by running:
```
cat /proc/sys/fs/inotify/max_user_watches
```
The limit can be increased to its maximum by editing `/etc/sysctl.conf` and adding this line to the end of the file:
```
fs.inotify.max_user_watches=524288
```
The new value can then be loaded in by running `sudo sysctl -p`.       
More details: https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers
