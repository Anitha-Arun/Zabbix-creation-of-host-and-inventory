Before using this code please create the host group , the hostgroup must be created in zabbix
Zabbix Host and Inventory Management Script
This Python script connects to the Zabbix API to automate the creation and updating of hosts and their inventory details. It reads data from a CSV file containing information about devices (hosts) such as their hostname, group, device name, type, operating system, serial number, tag, and MAC address. The script will then create new hosts in Zabbix if they don't already exist, associating them with the specified group and adding the inventory details.

Key Features:
Zabbix API Integration: The script uses the pyzabbix library to connect to a Zabbix server via its API, enabling interaction with Zabbix resources like hosts and host groups.

CSV Input: The script reads a CSV file that contains device information (hostname, group, name, type, OS, serial number, tag, and MAC address) to create or update the corresponding hosts in Zabbix.

Host Group Management: It looks up the group ID for a given group name and associates the host with that group during creation.

Host Creation: If the hostname from the CSV file does not already exist in Zabbix, the script will create the host with the given inventory details (device name, type, OS, serial number, MAC, and tag).

Inventory Information: It associates detailed inventory fields, such as the device name, type, serial number, OS, MAC address, and tag, with the host during creation.

Error Handling: The script includes error handling to capture and report any issues while interacting with the Zabbix API, such as failing to create a host or failing to find a host group.

How It Works:
Zabbix API Connection: The script connects to the Zabbix API using the provided Zabbix URL, username, and password.

Fetching Host Group ID: For each entry in the CSV, the script fetches the group ID based on the group name provided in the CSV file.

Checking for Existing Host: The script checks if the host (based on its hostname) already exists in Zabbix. If it does, it skips creating the host and proceeds to the next entry.

Creating a New Host: If the host does not exist, it creates a new host in Zabbix and adds the corresponding inventory details (device name, type, OS, serial number, MAC address, and tag).

Error Logging: Any errors encountered (e.g., issues with Zabbix API requests, missing CSV data) are logged and displayed.

CSV Format:
The script expects the CSV file to have the following columns (case-sensitive):

Host: The hostname of the device.
Group: The group name where the host should be added.
Name: The device name to be stored in the host inventory.
Type: The device type to be stored in the host inventory.
OS: The operating system to be stored in the host inventory.
Serial No: The device's serial number to be stored in the host inventory.
Tag: A tag that can be used for categorization.
MAC: The MAC address to be stored in the host inventory.
Steps to Run:
Install Dependencies: Ensure you have pyzabbix installed:


pip install pyzabbix

Configure Zabbix API Credentials: Set the following variables with your Zabbix details:

ZABBIX_URL: Your Zabbix server URL (e.g., http://your-zabbix-server/zabbix).
ZABBIX_USER: Your Zabbix username.
ZABBIX_PASSWORD: Your Zabbix password.
Prepare the CSV File: Prepare a CSV file (input.csv) with the following columns: Host, Group, Name, Type, OS, Serial No, Tag, MAC.

Run the Script: Execute the script:

bash

python create_hosts.py

Verify in Zabbix: After running the script, log into your Zabbix dashboard and verify that the hosts were created or updated with the correct inventory information.

Script Example:
import csv
from pyzabbix import ZabbixAPI

# Zabbix API connection details
ZABBIX_URL = 'http://your-zabbix-url/zabbix' 
ZABBIX_USER = 'your-zabbix-user' 
ZABBIX_PASSWORD = 'your-zabbix-password' 

# Connect to Zabbix API
zapi = ZabbixAPI(ZABBIX_URL)
zapi.login(ZABBIX_USER, ZABBIX_PASSWORD)

# Function to get group ID by name
def get_group_id(group_name):
    groups = zapi.hostgroup.get(filter={"name": group_name})
    if groups:
        return groups[0]['groupid']
    return None

# Function to get host ID by hostname (case insensitive)
def get_host_id(hostname):
    hosts = zapi.host.get(filter={"host": {"like": hostname.strip()}})
    if hosts:
        return hosts[0]['hostid']
    return None

# Function to create a new host with inventory details
def create_host_with_inventory(hostname, group_id, device_name, device_type, os, serial_no, tag, mac):
    result = zapi.host.create(
        host=hostname,
        groups=[{"groupid": group_id}],
        interfaces=[],  # No interfaces specified
        inventory={
            "name": device_name,
            "type": device_type,
            "os": os,
            "serialno_a": serial_no,
            "macaddress_a": mac,
            "tag": tag
        }
    )
    return result['hostids'][0] if 'hostids' in result else None

# Open CSV file
with open('input.csv', mode='r', encoding='utf-8-sig') as file:
    reader = csv.DictReader(file)
    
    for row in reader:
        hostname = row.get('Host', '').strip()
        group_name = row.get('Group', '').strip()
        device_name = row.get('Name', '').strip()
        device_type = row.get('Type', '').strip()
        os = row.get('OS', '').strip()
        serial_no = row.get('Serial No', '').strip()
        tag = row.get('Tag', '').strip()
        mac = row.get('MAC', '').strip()

        try:
            # Get group ID
            group_id = get_group_id(group_name) if group_name else None

            if group_id:
                # Check if the host exists
                host_id = get_host_id(hostname)

                if not host_id:
                    # Create the host with inventory details if it doesn't exist
                    host_id = create_host_with_inventory(
                        hostname, group_id, device_name, device_type, os, serial_no, tag, mac
                    )
                    if host_id:
                        print(f"Created host {hostname} with ID {host_id}")
                    else:
                        print(f"Failed to create host {hostname}")
                else:
                    print(f"Host {hostname} already exists with ID {host_id}")

            else:
                print(f"Group '{group_name}' not found")

        except Exception as e:
            print(f"Error updating host with hostname {hostname}: {e}")

print("Host creation complete.")

This description provides detailed insights into the purpose and operation of the script, making it easier for others to understand and use it effectively.
Author__
This is a Python script written and maintained by Anitha.Damarla.
for Doubts please contant :Anithadamarla0313@gmail.com


