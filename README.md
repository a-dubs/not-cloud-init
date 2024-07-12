# not-cloud-init

![Static Badge](https://img.shields.io/badge/version-v0.9.5-blue) ![Static Badge](https://img.shields.io/badge/Python-%3E=3.8-green)

A tool for gathering information about a currently running machine and generating a basic cloud-config that can be used as a good starting point for re-creating the original machine via cloud-init.  
Since this tool only generates a cloud-config file for cloud-init to use, this tool is inherently limited by cloud-init capabilities.

## Current State: Beta (v0.9.5)
The tool is currently in a beta state and has all the main features implemented.  

A CLI has now been implemented to allow for customizing the functionality of this tool.  

Further testing across a variety of systems is needed to ensure the tool is working as expected.

## Features (What can it do?)

This CLI tool gathers information from your system and generates a cloud-config file that can be used with cloud-init to create a simple image similar to the original system.

- Gather and export Apt sources from sources.list and sources.list.d/
- Gather and export Snaps installed on the system
- Gather and export manually installed Apt packages
- Gather and export detailed operating system info
- Rename the current user to the default "ubuntu" user for VM or Cloud use
- Gather and export SSHD config info (root login, password auth, etc.)
- Gather and export SSH keys for the current user (public keys, authorized_keys)
- Gather and export User info such as what shell they use and if they have sudo rights or not

## Installation

### Manual Installation

1. Clone this repo  

  * Using SSH  

      ```bash
      git clone git@github.com:a-dubs/not-cloud-init.git
      ```
  * Using HTTPS

      ```bash
      git clone https://github.com/a-dubs/not-cloud-init.git
      ```
2. Create and activate a virtual environment (optional but recommended)
   * Install `pyvenv` (if not already installed)
   ```bash
   sudo apt-get install python3-venv
   ```
   * Create a virtual environment
   ```bash
   python3 -m venv venv
   ```
   * Activate the virtual environment
   ```bash
   source venv/bin/activate
   ```
3. Install the dependencies
   ```bash
   pip install .
   ```
4. Check if the CLI is working
   ```bash
   python -m not_cloud_init.cli --help
   ```

### Auto install (Recommended for fresh containers/VMs only):

```bash 
curl -s https://raw.githubusercontent.com/a-dubs/not-cloud-init/main/install.sh -o install.sh && source install.sh
```

#### Contents of the install.sh script:
```bash
#!/bin/bash

# Clone the repository using HTTPS
echo "Cloning the repository..."
git clone https://github.com/a-dubs/not-cloud-init.git
cd not-cloud-init

# Install python3-venv if not already installed
echo "Installing python3-venv if not already installed..."
sudo apt-get install -y python3-venv

# Create and activate a virtual environment
echo "Creating and activating virtual environment..."
python3 -m venv venv
source venv/bin/activate

# Install dependencies
echo "Installing dependencies..."
pip install .

# Check if the CLI is working
echo "Checking if CLI is working..."
python -m not_cloud_init.cli --help

echo "Installation complete."
```

## How to use

### Base Level CLI
The CLI entrypoint is `cli.py` in the `src` directory.

```bash
Usage: python -m not_cloud_init.cli [OPTIONS] COMMAND [ARGS]...

  CLI tool for system package management.

Options:
  --log-level TEXT  Set the logging level (DEBUG, INFO, WARNING, ERROR,
                    CRITICAL).
  --help            Show this message and exit.

Commands:
  generate
```

### Generating a cloud-config file

To generate a cloud-config file, use the `generate` command.

The `generate` command has the following CLI usage:

```bash
Usage: python -m not_cloud_init.cli generate [OPTIONS]

  Generate a cloud-init configuration file for the current machine.

Options:
  -v, --verbose            Enable verbose output.
  -o, --output-path TEXT   Path to output file.
  -f, --force              Write over output file if it already exists.
  --gather-hostname        Enable gathering the hostname of the machine. This
                           is will cause issues unless this exact machine is
                           being redeployed using the generated cloud-init
                           config.
  --gather-public-keys     Enable gathering of all public key files in the
                           ~/.ssh directory. This will allow you to use the
                           same public keys on the new machine as the current
                           machine.
  --password TEXT          Set the password for the user. WARNING: This is
                           incredibly insecure and is stored in plaintext in
                           the cloud-init config.
  --disable-apt            Disable the gathering and generation of apt config.
  --disable-snap           Disable the gathering and generation of snap
                           config.
  --disable-ssh            Disable the gathering and generation of ssh config.
  --disable-user           Disable the gathering and generation of user
                           config.
  --rename-to-ubuntu-user  Keep the current user but rename it to the default
                           'ubuntu' user.
  --help                   Show this message and exit.
```

### Example Calls:

#### Minimum Call
```bash
python -m not_cloud_init.cli generate
```
This will output a cloud-config file named cloud-config.yaml with the default settings.

#### Minimum Call with Verbose Output
```bash
python -m not_cloud_init.cli generate -v
```
This will output a cloud-config file named cloud-config.yaml with the default settings
and will output more verbose information about the process to console. 

#### Custom Output Path Example 
```bash
python -m not_cloud_init.cli generate -o cc.yaml
```
This will output a cloud-config file named cc.yaml with the default settings.  
If cc.yaml already exists, the tool will not overwrite it because the `-f` flag is not passed.

#### Overwrite Custom Output Path Example
```bash
python -m not_cloud_init.cli generate -o cc.yaml -f 
```
This will output a cloud-config file named cc.yaml with the default settings.  
But now, the tool will overwrite cc.yaml if it already exists.

#### Disable Apt Example
```bash
python -m not_cloud_init.cli --log-level DEBUG generate -f --disable-apt
```
This will gather all other configs except for the apt sources and packages installed on the system.
And will overwrite the output file if it already exists.

#### Rename User Example
```bash
python -m not_cloud_init.cli --log-level DEBUG generate -o cc.yaml --rename-to-ubuntu-user
```
This will gather the config for the user the tool is run as, but rename the user to "ubuntu" in the cloud-config file
so that when the cloud-config is used to create a new machine, the user will be named "ubuntu" instead of the original.

#### Complex Example
```bash
python -m not_cloud_init.cli \
    --log-level DEBUG \
    generate  \
    --password letmein \
    --disable-apt \
    --disable-snap \
    -f \
    --gather-hostname \
    --gather-public-keys
```
This will generate a cloud-config file with the password "letmein" for the user, disable the gathering of apt and snap
configs, force overwrite the output file if it already exists, capture the hostname of the machine, gather the public
keys for recreation on the new machine, and sets the log level to DEBUG for the log file. 


## Gathering feedback

If you have a few minutes to kill, please consider running this tool locally on your system or machine(s) of your
choosing.   
Currently this project should be in a decently stable state, so I'm looking for feedback on any bugs or unexpected
behavior involving the newly created CLI.

1. Follow the instructions above in the **Installation** section to install not-cloud-init on your machine(s) of choosing.

2. Run the not-cloud-init CLI:  
   * Follow the instructions above for suggested usage and mess around with the CLI options to see what the tool can do.  
   * At the very least, invoke the tool trying out both standard and verbose output levels so that you can provide feedback on the cli output as well:

      Standard Output:
      ```bash
      python -m not_cloud_init.cli generate -f
      ```
      Verbose Output:
      ```bash
      python -m not_cloud_init.cli generate -f -v
      ```

3. Look over the content of the generated cloud-config file and then fill out this google form with your feedback
and optionally include the generated cloud-config file in the form as well if you are comfortable with doing so.  
https://forms.gle/aE8476VK9miAbZLc8  

If you end up testing on multiple machines, you can just fill out the form multiple times and just skip to the second
page of the form to submit the cloud-config files. 


## Submitting Issues Via GitHub
In addition to the feedback form, feel free to open an issue against this repo for any bugs or features you'd like to see addressed. 

Thanks! 

