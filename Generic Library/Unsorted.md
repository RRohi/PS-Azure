# Various cmdlets related to Azure resources.

## Get RG object.
```powershell
$RG = Get-AzResourceGroup -ResourceGroupName rgname
```

## Get NSG object.
```powershell
$NSG = Get-AzNetworkSecurityGroup -ResourceGroupName $RG.ResourceGroupName -Name NSGname
```

## Add inbound RDP security rule to NSG.
```powershell
$NSG = Get-AzNetworkSecurityGroup -ResourceGroupName $RG.ResourceGroupName -Name NSGname
$NSG | Add-AzNetworkSecurityRuleConfig -Name InboundRDP -SourceAddressPrefix * -SourcePortRange * -DesinationAddressPrefix * -DestinationPortRange 3389 -Access Allow -Priority 300 -Protocol TCP -Direction Inbound
$NSG | Set-AzNetworkSecurityGroup
```

## Associate NSG to a NIC.
```powershell
$NIC = Get-AzNetworkInterface -Name nic0
$NIC.NetworkSecurityGroup = $NSG
$NIC | Set-AzNetworkInterface
```

## Create new private DNS zone.
```powershell
New-AzPrivateDnsZone -ResourceGroupName $RG.ResourceGroupName -Name domain.name
```

## Add virtual network link to private DNS zone.
```powershell
New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $RG.ResourceGroupName -ZoneName domain.name -Name linkname -VirtualNetworkId $NIC.Id -EnableRegistration
```

## List Private DNS records.
```powershell
Get-AzPrivateDnsRecordSet -ResourceGroupName $RG.ResourceGroupName -ZoneName domain.name
```

## Create DNS for external resolution.
```powershell
New-AzDnsZone -ResourceGroupName $RG.ResourceGroupName -Name external.dns.name
```

## Create an A record in DNS.
```powershell
Get-AzPublicIpAddress -ResourceGroupName $RG.ResourceGroupName -Name pipname -OutVariable PIP | Out-Null
$Records = @()
$Records += New-AzDnsRecordConfig $PIP.IpAddress
New-AzDnsRecordSet -ResourceGroupName $RG.ResourceGroupName -ZoneName domain.name -RecordType A -Name vmname -Ttl 3600 -DnsRecords $Records
```

## Modify a record in DNS.
```powershell
$RS = Get-AzDnsRecordSet -ResourceGroupName $RG.ResourceGroupName -ZoneName domain.name -Name recordname -RecordType A
$RS.Ttl = 36000
Set-AzDnsRecordSet -RecordSet $RS
```
