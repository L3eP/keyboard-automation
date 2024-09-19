Below is a Bash script that automates the disabling of the internal keyboard when the external keyboard Bluetooth keyboard is detected. The script runs continuously and will react to Bluetooth activity by disabling the internal keyboard when external keyboards is connected and enabling it when external keyboard is disconnected.

### Bash Script: `keebs.sh`

==================================================================================================================
#!/bin/bash

# Internal keyboard device name
INTERNAL_KEYBOARD="AT Translated Set 2 keyboard"

# External keyboard device name
EXTERNAL_KEYBOARD="Keychron K3"

# Function to get the device ID of a given device name
get_device_id() {
    xinput list | grep "$1" | awk '{print $6}' | sed 's/id=//'
}

# Function to disable the internal keyboard
disable_internal_keyboard() {
    local internal_id=$(get_device_id "$INTERNAL_KEYBOARD")
    if [ -n "$internal_id" ]; then
        xinput disable "$internal_id"
        echo "$(date): Disabled internal keyboard." >> /var/log/toggle_keyboard.log
    else
        echo "$(date): Internal keyboard not found." >> /var/log/toggle_keyboard.log
    fi
}

# Function to enable the internal keyboard
enable_internal_keyboard() {
    local internal_id=$(get_device_id "$INTERNAL_KEYBOARD")
    if [ -n "$internal_id" ]; then
        xinput enable "$internal_id"
        echo "$(date): Enabled internal keyboard." >> /var/log/toggle_keyboard.log
    else
        echo "$(date): Internal keyboard not found." >> /var/log/toggle_keyboard.log
    fi
}

# Main monitoring loop
while true; do
    # Check if the external keyboard is connected
    if xinput list | grep -q "$EXTERNAL_KEYBOARD"; then
        disable_internal_keyboard
    else
        enable_internal_keyboard
    fi
    # Sleep for a short duration before checking again
    sleep 5
done

==================================================================================================================
### Instructions to Set Up the Script:

1. **Save the Script**: Copy the script into a file named `keebs.sh`.

2. **Make it Executable**: chmod +x keebs.sh

3. **Run the Script**:
   You can run the script in the background: ./keebs.sh &

### Triggering the Script on Bluetooth Activity

To trigger the script on Bluetooth activity, you can use a combination of `bluetoothctl` and the script as follows:

1. **Modify the Script for Bluetooth Activity**:
   Change the main loop to check for Bluetooth activity using `bluetoothctl`:

   # Main monitoring loop
   while true; do
       # Check if Keychron K3 is connected using bluetoothctl
       if bluetoothctl info | grep -q "$EXTERNAL_KEYBOARD"; then
           disable_internal_keyboard
       else
           enable_internal_keyboard
       fi
       # Sleep for a short duration before checking again
       sleep 5
   done

2. **Install `bluetoothctl`**: 
   Make sure you have `bluetoothctl` installed on your system. It's usually available by default on most Linux distributions with Bluetooth support.

### Setup Logging Permissions

Ensure that the script has permission to write logs to `/var/log/toggle_keyboard.log`. You may need to create the log file and set appropriate permissions:

sudo touch /var/log/toggle_keyboard.log
sudo chmod 666 /var/log/toggle_keyboard.log

### Best Practices

- **Continuous Monitoring**: The script runs indefinitely, checking every 5 seconds. You can adjust the sleep duration if you want a more frequent check.
- **Logging**: The script logs actions to a specified log file for monitoring purposes.
- **Resource Usage**: Monitor resource usage to ensure that the script is not consuming excessive CPU or memory.

Feel free to customize the device names and paths according to your specific environment!
