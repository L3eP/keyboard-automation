Dibawah ini merupakan Bash script untuk menonaktifkan internal keyboard ketika external bluetooth keyboard terdeteksi. kode ini akan berjalan secara kontinu dan bereaksi pada aktifitas Bluetooth ketika external keyboards terdeteksi dan menonaktifkan internal keyboard hingga external keyboard terputus dan akan mengaktifkan internal keyboard kembali.

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
        echo "$(date): Disabled internal keyboard." >> /var/log/keebs.log
    else
        echo "$(date): Internal keyboard not found." >> /var/log/keebs.log
    fi
}

# Function to enable the internal keyboard
enable_internal_keyboard() {
    local internal_id=$(get_device_id "$INTERNAL_KEYBOARD")
    if [ -n "$internal_id" ]; then
        xinput enable "$internal_id"
        echo "$(date): Enabled internal keyboard." >> /var/log/keebs.log
    else
        echo "$(date): Internal keyboard not found." >> /var/log/keebs.log
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
### Langkah setup untuk script ini:

1. **Menyimpan Script**: Copy the script into a file named `toggle_internal_keyboard.sh`.

2. **Mengizinkan eksekusi pada script**: chmod +x toggle_internal_keyboard.sh

3. **Menjalankan Script di latarbelakang**: ./toggle_internal_keyboard.sh &
==================================================================================================================

### Aktifasi script berdasarkan Bluetooth Activity

Untuk mengaktifkan script saat ada aktifitas Bluetooth, kita bisa kombinasikan dengan `bluetoothctl` dengan script sebagai berikut:

1. **Modifikasi script untuk Bluetooth Activity**:
   Mengubah looping untuk memastikan Bluetooth activity menggunakan `bluetoothctl`:

==================================================================================================================
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

==================================================================================================================
2. **Install `bluetoothctl`**: 
   pastikan `bluetoothctl` terinstall pada sistem. Biasanya sudah terinstall secara bawaan pada perangkat yang mendukung koneksi bluetooth.

### Penyesuaian izin untuk melakukan logging

pastikan sistem mengizinkan modifikasi pada file `/var/log/keebs.log`. kita perlu membuat file dan menentukan izin akses file tersebut:

sudo touch /var/log/keebs.log
sudo chmod 666 /var/log/keebs.log

==================================================================================================================
### Best Practices

- **Continuous Monitoring**: script akan berjalan,dan memastikan setiap 5 detik aktifitas bluetooth. kamu bisa ubah menyesuaikan frekuensi aktifitasmu.
- **Logging**: script akan melakukan logging hanya untuk tujuan monitoring.
- **Resource Usage**: pastikan script tidak memakan sumberdaya terlalu banyak saat dijalankan.

 Feel free to initialized the device names and paths according to your specific environment!
