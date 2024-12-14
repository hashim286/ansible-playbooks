

# Purpose

The point of this playbook package is to permit us to utilize CaC (Configuration as Code) to configure switchports on the various switches in the lab.

  

# Usage

- (recommended) create a python virtual environment, this was created with python 3.12.3 so use that at least if possible

	`python -m venv ./.venv`
	`source .venv/bin/activate`

- install requirements with pip 

	`pip install -r requirements.txt`

- the general flow for using this package is to first create a yaml file called vars.yaml. In this yaml file we use the format in the [YAML Format](#YAML%20Format) section
- after the vars.yaml file is defined, run the make-playbook.yaml file to generate the ansible playbook that will run the configuration 
- define a credentials file in the credentials path entered into the YAML. The name for the file should be credentials_(os).yaml. In this case, os must mirror the name entered into the YAML
	- credentials_junos.yaml
	- credentials_nxos.yaml
	- the files can use the format below 
	~~~
	username: (username)
	password: (password)
	~~~
- ensure the files are encrypted by running below command

	`ansible-vault encrypt (file_path)`
	
- look over the playbook to make sure it has the right parameters
- run playbook, enter vault pass you used during encryption

	`ansible-playbook playbooks/playbook.yaml --ask-vault-pass`
	
## YAML Format
- if the below format has messed up spacing, refer to vars.yaml for a tab corrected example

~~~
credentials_file_path: path to credentials folder

switches:
   - hostname: 101.101.62.156
     os: junos
     els: false
     interfaces: 
      - interface: ge-0/0/45
        mode: trunk
        vlan: 62-65
   - hostname: 101.101.62.153
     os: nxos
     interfaces: 
      - interface: Ethernet1/1
         mode: trunk
        vlan: 1,62-65
  - hostname: 101.101.62.156
    os: junos
    els: False
    interfaces: 
     - interface: ge-0/0/43
       mode: access
       vlan: 50

  - hostname: 101.101.62.154
    os: nxos
    interfaces: 
     - interface: Ethernet1/51
        mode: access
        vlan: 1
  - hostname: 101.101.62.155
    os: junos
    els: True
    interfaces: 
     - interface: ge-0/0/43
       mode: access
       vlan: 62
     - interface: ge-0/0/45
       mode: access
       vlan: 62

~~~

- credentials_file_path: this is the directory where you have stored the login credentials to the switch
- switches: stores info for all hosts we are configuring
- hostname: the IP or DNS address of the specific host we are applying configuration to 
- os: the os the device runs on 
	- for cisco it will be nxos, ios, etc. 
	- for juniper it will always be junos
- els: for junos there are two separate types of OS's, ELS and non-ELS. If the device is a cisco or non-junos entry the ELS field is not mandatory. For any junos device this field must be filled
	- non-ELS: when configuring, if you are setting the trunk/access port by below command, it is a non-ELS junos device and els: false should be used, if it's set under "interface-mode" then it is an ELS junos and els: true should be used
	
`set interface ge-0/0/1 unit 0 family ethernet-switching port-mode (acess | trunk)`

- interfaces: stores the info for all interface configurations within the host
	- interface: the name of the interface we are configuring
	- mode: either trunk or access
	- vlan: vlan ID or set of VLANs we are running on the link 
		- if trunk, these can be multiple values entered as vlan1,vlan2,vlan5-7
			- this adds vlan1, 2, 5, 6, and 7 on the link
		- if access, this must be 1 vlan or left blank to do vlan 1 (entering vlan 1 manually also works)

