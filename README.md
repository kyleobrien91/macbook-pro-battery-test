# MacBook Pro Battery Life Test

This project is designed to test laptop battery life under a simulated heavy workload. This repository contains a simple test script to emulate a relatively heavy workload for battery life testing. It downloads a copy of [Drupal VM](https://www.drupalvm.com) and repeatedly builds and destroys a Virtual Machine running Drupal.

The script does the following, in a loop:

  1. Write a counter, timestamp and the battery percentage (as reported by `pmset`) to a results file.
  3. Run `vagrant up` to configure a VM running Drupal on a standard LAMP stack.
  5. Run `vagrant destroy -f` to destroy the VM.
  6. Wait 10s.
  7. Repeat.

### `battery-test.sh` Detailed Workflow

The `battery-test.sh` script executes the following steps to simulate a heavy workload and log battery performance:

1.  **Power Check**: Verifies the laptop is running on battery power. If AC power is detected, it exits with a message asking to unplug the adapter.
2.  **Initialization**:
    *   Sets up a results directory (`results/`).
    *   Creates a timestamped CSV file (e.g., `results/YYYY-MM-DD_HH.MM.SS.csv`) to store the test data.
    *   Writes the header row `Counter,Time,Battery Percentage` to the CSV file.
3.  **Drupal VM Download and Configuration**:
    *   Downloads the latest version of Drupal VM from GitHub.
    *   Extracts the downloaded archive into a `drupal-vm-master` directory.
    *   Creates a minimal `config.yml` within `drupal-vm-master` to customize the VM settings for the test (e.g., hostname, synced folder type). This ensures a consistent and relatively quick VM setup.
4.  **Main Test Loop (Infinite)**:
    *   **Log Battery Status**:
        *   Records the current loop counter, timestamp, and battery percentage into the CSV file.
        *   Uses `pmset -g batt` on macOS and `cat /sys/class/power_supply/BAT0/capacity` on Linux to get battery status.
    *   **Provision VM**:
        *   Navigates into the `drupal-vm-master` directory.
        *   Executes `vagrant up` to build and start the Drupal virtual machine. This action simulates a heavy workload by utilizing CPU, memory, and disk resources.
    *   **Pause**: Waits for 10 seconds after the VM is up.
    *   **Destroy VM**:
        *   Removes any local Drupal files (`rm -rf drupal`).
        *   Executes `vagrant destroy -f` to forcefully shut down and delete the virtual machine.
        *   Navigates back to the project's root directory.
    *   **Increment Counter**: Increases the loop counter.
    *   The script repeats this loop until manually stopped (Ctrl+C) or the battery is depleted.

To run the script, you should already have the latest versions of [Vagrant](https://www.vagrantup.com/downloads.html) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed.

> **Note about Vagrant plugins**: The author runs the tests without `vagrant-cachier` installed for consistency's sake. If you use Vagrant regularly, check to make sure you don't have any plugins installed which could affect the consistency of this test using `vagrant plugin list`!

## Other Platforms

This test script should run on any platform which supports Vagrant and VirtualBox, though it's only been tested on macOS, Fedora, and Ubuntu at this time.

## Usage

### Before running the test script

  1. **Disable Sleep**: Go to System Preferences > Energy Saver, click on the 'Battery' tab, and drag the 'Turn display off after' slider all the way to 'Never' (alternatively, you could run `caffeinate` in a separate Terminal window).
  2. **Disable Screen Saver**: Open System Preferences > Desktop & Screen Saver, then set the Screen Saver to 'Start after: Never'.
  3. **Turn up brightness**: For consistency's sake, turn up your screen brightness all the way (after the AC power has been disconnected).
  4. **Quit all other Applications**: To make it a fair comparison. (I also make sure all my Macs are running identical software configurations using the [Mac Development Ansible Playbook](https://github.com/geerlingguy/mac-dev-playbook)).

### Run the test script

  1. **Download project**: Download this project to your computer (either download through GitHub or clone it with Git). _Important Note_: Don't download the project in a 'cloud' directory (e.g. inside Dropbox, Google Drive, or a folder synced via iCloud).
  2. **Open Terminal (full screen)**: Open Terminal.app and put it in full screen mode (so the actual pixels displayed is identical from laptop-to-laptop).
  3. **Run Script**: Change into this project's directory (`cd path/to/macbook-pro-battery-test`). Run `./battery-test.sh`, and then walk away for a few hours.

After your Mac forces a sleep (when the battery has run out), plug it back in, then check the most recent file in `results/` in the project directory.

## Results

Results are written to a date-and-timestamped file inside the `results` folder. This file is in CSV format, so you can open it in Excel, Numbers, Google Sheets, or any other CSV-compatible program and graph the results as needed.

The results file has the following structure (as an example):

| Counter | Time                | Battery Percentage |
| ------- | ------------------- | ------------------ |
| 0       | 2017-01-07 15:58:40 | 100%               |
| 0       | 2017-01-07 16:10:48 | 98%                |
| 0       | 2017-01-07 16:17:22 | 94%                |
| ...     | ...                 | ...                |

Results of this script's test runs have been posted to the author's blog and a public Google Sheet:

  - Raw data in Google Sheets: [2016 MacBook Pro Battery Comparisons](https://docs.google.com/spreadsheets/d/16H6TeKCOZRwzsd5bZJM2IHVqN9fU6GZhUrDiu_SK2zU/edit?usp=sharing)
  - Blog post: [Battery Life - Why I Returned my 2016 MacBook Pro with Touch Bar](http://www.jeffgeerling.com/blog/2017/i-returned-my-2016-macbook-pro-touch-bar#battery-life)

## Key Components and Files

*   **`battery-test.sh`**: The main shell script that automates the battery testing process. It downloads Drupal VM, provisions it, logs battery status, and repeats this cycle. (Described in detail above).
*   **`README.md`**: (This file) Provides an overview of the project, instructions on how to set up and run the test, and details about the script's functionality and results.
*   **`LICENSE`**: Contains the MIT License under which this project is distributed, granting liberal permissions for use and modification.
*   **`.gitignore`**: Specifies files and directories that Git should ignore. This includes the `results/` directory (which stores test output) and the `drupal-vm-master/` directory (which is created during the script's execution to house Drupal VM).
*   **`results/` (directory)**: This directory is created by `battery-test.sh` to store the output of the battery tests. Each test run generates a CSV file in this directory, named with the date and time of the test's start. These files contain the battery percentage logged at intervals throughout the test.
*   **`.github/` (directory)**: Contains GitHub-specific files, such as `FUNDING.yml` for sponsoring the project.

## Author

This script was created by [Jeff Geerling](http://www.jeffgeerling.com) to run some more formal battery tests on the 2016 Retina MacBook Pro—both with and without Touch Bar—and to see if battery life and performance between the two models (under heavier load) was much different.
