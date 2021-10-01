## Running CLI Scripts and finding and deleting Unused NIC
Connecting to azure and running azure cli scripts in cloudshell bash.

**az account list --output table**
```
siju_manalikatil@Azure:~$ az account list --output table
A few accounts are skipped as they don't have 'Enabled' state. Use '--all' to display them.
Name                                                      CloudName    SubscriptionId                        State    IsDefault
--------------------------------------------------------  -----------  ------------------------------------  -------  -----------
Visual Studio Enterprise – MPN                            AzureCloud   93cc2a24-ba9e-405e-af11-48affddf6342  Enabled  True
CA-13661198 New Technology Transformation-GTO Architects  AzureCloud   6965eb53-1efc-427c-90a5-b7f30f902d11  Enabled  False
```

**az account set --subscription 93cc2a24-ba9e-405e-af11-48affddf6342**

```
siju_manalikatil@Azure:~$ az account show --output table
EnvironmentName    HomeTenantId                          IsDefault    Name                            State    TenantId
-----------------  ------------------------------------  -----------  ------------------------------  -------  ------------------------------------
AzureCloud         b9fec68c-c92d-461e-9a97-3d03a0f18b82  True         Visual Studio Enterprise – MPN  Enabled  b9fec68c-c92d-461e-9a97-3d03a0f18b82
```

While trying to list the Nics kept getting below error.
```
siju_manalikatil@Azure:~/scripts/maintenance$ az network vnet list
(ResourceGroupNotFound) Resource group 'vnetTest' could not be found.
```
ran same command with debug option
az network vnet list --debug
found below entry
cli.knack.commands: Configured default 'vnetTest' for arg resource_group_name

Looked may be some default configuration, checked az config 

```
siju_manalikatil@Azure:~/scripts/maintenance$ az config get
Command group 'config' is experimental and under development. Reference and support levels: https://aka.ms/CLI_refstatus
{
  "cloud": [
    {
      "name": "name",
      "source": "/home/siju_manalikatil/.azure/config",
      "value": "AzureCloud"
    }
  ],
  "defaults": [
    {
      "name": "group",
      "source": "/home/siju_manalikatil/.azure/config",
      "value": "vnetTest"
    }
  ]
}
siju_manalikatil@Azure:~/scripts/maintenance$

siju_manalikatil@Azure:~/scripts/maintenance$ az config unset defaults.group=vnetTest --local

this command did not work, so had manually remove the entry from file
/home/siju_manalikatil/.azure/config

```

Now created the script file
find-unused-nic.sh

```
# Set deleteUnattachedNics=1 if you want to delete unattached NICs
# Set deleteUnattachedNics=0 if you want to see the Id(s) of the unattached NICs
deleteUnattachedNics=0

unattachedNicsIds=$(az network nic list --query '[?virtualMachine==`null`].[id]' -o tsv)
for id in ${unattachedNicsIds[@]}
do
   if (( $deleteUnattachedNics == 1 ))
   then

       echo "Deleting unattached NIC with Id: "$id
       az network nic delete --ids $id
       echo "Deleted unattached NIC with Id: "$id
   else
       echo $id
   fi
done


siju_manalikatil@Azure:~/scripts/maintenance$ . find-unused-nic.sh
/subscriptions/93cc2a24-ba9e-405e-af11-48affddf6342/resourceGroups/Kubernetes-RG/providers/Microsoft.Network/networkInterfaces/kube-worker-1367
/subscriptions/93cc2a24-ba9e-405e-af11-48affddf6342/resourceGroups/Kubernetes-RG/providers/Microsoft.Network/networkInterfaces/myVMNic
/subscriptions/93cc2a24-ba9e-405e-af11-48affddf6342/resourceGroups/Kubernetes-RG/providers/Microsoft.Network/networkInterfaces/testvm563

after changing value to 1 

siju_manalikatil@Azure:~/scripts/maintenance$ . find-unused-nic.sh
Deleting unattached NIC with Id: /subscriptions/93cc2a24-ba9e-405e-af11-48affddf6342/resourceGroups/Kubernetes-RG/providers/Microsoft.Network/networkInterfaces/kube-worker-1367
Deleted unattached NIC with Id: /subscriptions/93cc2a24-ba9e-405e-af11-48affddf6342/resourceGroups/Kubernetes-RG/providers/Microsoft.Network/networkInterfaces/kube-worker-1367
Deleting unattached NIC with Id: /subscriptions/93cc2a24-ba9e-405e-af11-48affddf6342/resourceGroups/Kubernetes-RG/providers/Microsoft.Network/networkInterfaces/myVMNic
 - Running ..
```
