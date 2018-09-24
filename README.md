# RHEL Deployment to Azure

This playbooks deploys a RHEL server (using parameters) to azure, waits for IP and creates default user clusteradmin.
The seccond playbook in playbook.yml can be extended with new roles to install/configure extra software.

## TDO's to run this playbook from Tower.

### Create survey with the following variables
<ol>
<li><strong>azure_machine_sku</strong>: RHEL machines: 7-RAW, 7-LVM, 7.4, 7.3</li>
<li><strong>azure_virtualMachineSize</strong>: Azure machine size: Standard_B2ms, Standard_B4ms, Standard_D2s_v3, Standard_D4s_v3, Standard_E2s_v3, Standard_E46s_v3</li>
<li><strong>azure_virtualMachineName</strong>: Machine name: must be unique and useable in a FQN. xxxxxx.westeurop.cloudapp.azure.com</li>
<li><strong>azure_wait</strong>: Time to wait: Secconds to wait until script will retrieve dynamic IP from Azure. (120 sec seems to work fine.)</li>
<li><strong>extraSoftware</strong>: Extra software: For now cloudforms, gitlab, tower and satellite can be deployed by this script.</li>
</ol>

### Create some credentials
<ol>
<li>Credential for vault. (vault pass in repo)</li>
<li>Machine credential for clusteradmin: This user is created on machine and will be used after IP is retrieved from Azure. public and private key in repo (dynamic host)</li>
</ol>


2 playbooks are in the playbook.yml.<br>
## 1) Deploy a RHEL server to azure:

This playbook has different roles.
<ul>
<li>createTemplate: creates dynamic parameters.json file for deployment to azure.
  <ul>Parameters are in group_vars/all.yml
     <li>please change azure parameters to match the azure resource group etc you use</li>
  </ul>
  <ul>Secret parameters are in vars/secret.yml
     <li>please change using password from /vault_pass.txt</li>
  </ul>
</li>
<li>loginAzure: logs in to azure</li>
<li>deployRHEL: deploys a RHEL server with parameters set in createTemplate</li>
<li>getIPFromAzure:
<ul>
   <li>waits 120 second to get ip from machine you just deployed</li>
   <li>creates host variable (dynamic) with retrieved ip for use in next playbook</li>
   <li>creates host file with dynamic retrieved ip</li>
</li>
</ul>

## 2) Creates standard user to new RHEL machine

It creates the user clusteradmin and adds public key.

The public and private keys are in /resources



# ssh

user: clusteradmin

public: resources/id_rsa.pub (uploaded to machine)

private: resources/id_rsa (can be used directly to configure machine via 'ssh clusteradmin@"{{ ansible_host }}" -i id_rsa' from /resources map)
