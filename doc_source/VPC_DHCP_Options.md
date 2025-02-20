# DHCP Options Sets<a name="VPC_DHCP_Options"></a>

The Dynamic Host Configuration Protocol \(DHCP\) provides a standard for passing configuration information to hosts on a TCP/IP network\. The `options` field of a DHCP message contains the configuration parameters\. Some of those parameters are the domain name, domain name server, and the netbios\-node\-type\.

You can configure DHCP options sets for your virtual private clouds \(VPC\)\.

**Topics**
+ [Overview of DHCP Options Sets](#DHCPOptionSets)
+ [Amazon DNS Server](#AmazonDNS)
+ [Changing DHCP Options](#DHCPOptions)
+ [Working with DHCP Options Sets](#DHCPOptionSet)
+ [API and Command Overview](#APIOverview)

## Overview of DHCP Options Sets<a name="DHCPOptionSets"></a>

The Amazon EC2 instances you launch into a nondefault VPC are private by default; they're not assigned a public IPv4 address unless you specifically assign one during launch, or you modify the subnet's public IPv4 address attribute\. By default, all instances in a nondefault VPC receive an unresolvable host name that AWS assigns \(for example, ip\-10\-0\-0\-202\)\. You can assign your own domain name to your instances, and use up to four of your own DNS servers\. To do that, you must specify a special set of DHCP options to use with the VPC\. 

The following table lists all the supported options for a DHCP options set\. You can specify only the options you need in your DHCP options set\. For more information about the options, see [RFC 2132](https://tools.ietf.org/html/rfc2132)\.


| DHCP Option Name | Description | 
| --- | --- | 
|  domain\-name\-servers  |  The IP addresses of up to four domain name servers, or AmazonProvidedDNS\. The default DHCP option set specifies AmazonProvidedDNS\. If specifying more than one domain name server, separate them with commas\. Although you can specify up to four domain name servers, note that some operating systems may impose lower limits\. If you want your instance to receive a custom DNS hostname as specified in `domain-name`, you must set `domain-name-servers` to a custom DNS server\.  | 
|  domain\-name  |  If you're using AmazonProvidedDNS in `us-east-1`, specify `ec2.internal`\. If you're using AmazonProvidedDNS in another region, specify *region*\.compute\.internal \(for example, `ap-northeast-1.compute.internal`\)\. Otherwise, specify a domain name \(for example, `example.com`\)\. This value is used to complete unqualified DNS hostnames\. For more information about DNS hostnames and DNS support in your VPC, see [Using DNS with Your VPC](vpc-dns.md)\.  Some Linux operating systems accept multiple domain names separated by spaces\. However, other Linux operating systems and Windows treat the value as a single domain, which results in unexpected behavior\. If your DHCP options set is associated with a VPC that has instances with multiple operating systems, specify only one domain name\.  | 
|  ntp\-servers  |  The IP addresses of up to four Network Time Protocol \(NTP\) servers\. For more information, see section 8\.3 of [RFC 2132](https://tools.ietf.org/html/rfc2132)\.   | 
|  netbios\-name\-servers  |  The IP addresses of up to four NetBIOS name servers\.  | 
|  netbios\-node\-type  |  The NetBIOS node type \(1, 2, 4, or 8\)\. We recommend that you specify 2 \(point\-to\-point, or P\-node\)\. Broadcast and multicast are not currently supported\. For more information about these node types, see section 8\.7 of [RFC 2132](https://tools.ietf.org/html/rfc2132) and section 10 of [RFC1001](https://tools.ietf.org/html/rfc1001)\.  | 

## Amazon DNS Server<a name="AmazonDNS"></a>

When you create a VPC, we automatically create a set of DHCP options and associate them with the VPC\. This set includes two options: `domain-name-servers=AmazonProvidedDNS`, and `domain-name=`*domain\-name\-for\-your\-region*\. AmazonProvidedDNS is an Amazon DNS server, and this option enables DNS for instances that need to communicate over the VPC's Internet gateway\. The string `AmazonProvidedDNS` maps to a DNS server running on a reserved IP address at the base of the VPC IPv4 network range, plus two\. For example, the DNS Server on a 10\.0\.0\.0/16 network is located at 10\.0\.0\.2\. For VPCs with multiple IPv4 CIDR blocks, the DNS server IP address is located in the primary CIDR block\.

**Note**  
The DNS server does not reside within a specific subnet or Availability Zone in a VPC. Furthermore, it is not possible to control access to it using Network ACLs or security groups.

When you launch an instance into a VPC, we provide the instance with a private DNS hostname, and a public DNS hostname if the instance receives a public IPv4 address\. If `domain-name-servers` in your DHCP options is set to AmazonProvidedDNS, the public DNS hostname takes the form `ec2-public-ipv4-address.compute-1.amazonaws.com` for the us\-east\-1 region, and `ec2-public-ipv4-address.region.compute.amazonaws.com` for other regions\. The private hostname takes the form `ip-private-ipv4-address.ec2.internal` for the us\-east\-1 region, and `ip-private-ipv4-address.region.compute.internal` for other regions\. To change these to custom DNS hostnames, you must set `domain-name-servers` to a custom DNS server\.

The Amazon DNS server in your VPC is used to resolve the DNS domain names that you specify in a private hosted zone in Route 53\. For more information about private hosted zones, see [Working with Private Hosted Zones](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html) in the *Amazon Route 53 Developer Guide*\.

Services that use the Hadoop framework, such as Amazon EMR, require instances to resolve their own fully qualified domain names \(FQDN\)\. In such cases, DNS resolution can fail if the `domain-name-servers` option is set to a custom value\. To ensure proper DNS resolution, consider adding a conditional forwarder on your DNS server to forward queries for the domain `region-name.compute.internal` to the Amazon DNS server\. For more information, see [Setting Up a VPC to Host Clusters](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-vpc-host-job-flows.html) in the *Amazon EMR Management Guide*\.

**Note**  
 You can use the Amazon DNS server IP address 169\.254\.169\.253, though some servers don't allow its use\. Windows Server 2008, for example, disallows the use of a DNS server located in the 169\.254\.x\.x network range\. 

## Changing DHCP Options<a name="DHCPOptions"></a>

After you create a set of DHCP options, you can't modify them\. If you want your VPC to use a different set of DHCP options, you must create a new set and associate them with your VPC\. You can also set up your VPC to use no DHCP options at all\.

You can have multiple sets of DHCP options, but you can associate only one set of DHCP options with a VPC at a time\. If you delete a VPC, the DHCP options set associated with the VPC is disassociated from the VPC\.

After you associate a new set of DHCP options with a VPC, any existing instances and all new instances that you launch in the VPC use these options\. You don't need to restart or relaunch the instances\. They automatically pick up the changes within a few hours, depending on how frequently the instance renews its DHCP lease\. If you want, you can explicitly renew the lease using the operating system on the instance\. 

## Working with DHCP Options Sets<a name="DHCPOptionSet"></a>

This section shows you how to work with DHCP options sets\.

**Topics**
+ [Creating a DHCP Options Set](#CreatingaDHCPOptionSet)
+ [Changing the Set of DHCP Options a VPC Uses](#ChangingDHCPOptionsofaVPC)
+ [Changing a VPC to use No DHCP Options](#DHCP_Use_No_Options)
+ [Deleting a DHCP Options Set](#DeletingaDHCPOptionSet)

### Creating a DHCP Options Set<a name="CreatingaDHCPOptionSet"></a>

You can create as many additional DHCP options sets as you want\. However, you can only associate a VPC with one set of DHCP options at a time\. After you create a set of DHCP options, you must configure your VPC to use it\. For more information, see [Changing the Set of DHCP Options a VPC Uses](#ChangingDHCPOptionsofaVPC)\.

**To create a DHCP options set**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **DHCP Options Sets**, and then choose **Create DHCP options set**\.

1. In the dialog box, enter values for the options that you want to use, and then choose **Yes, Create**\.
**Important**  
If your VPC has an Internet gateway, make sure to specify your own DNS server or Amazon's DNS server \(AmazonProvidedDNS\) for the **Domain name servers** value\. Otherwise, the instances that need to communicate with the Internet won't have access to DNS\.

   The new set of DHCP options appears in your list of DHCP options\.

1. Make a note of the ID of the new set of DHCP options \(dopt\-*xxxxxxxx*\)\. You will need it to associate the new set of options with your VPC\.

Although you've created a set of DHCP options, you must associate it with your VPC for the options to take effect\. You can create multiple sets of DHCP options, but you can associate only one set of DHCP options with your VPC at a time\.

### Changing the Set of DHCP Options a VPC Uses<a name="ChangingDHCPOptionsofaVPC"></a>

You can change which set of DHCP options your VPC uses\. If you want the VPC to use no DHCP options, see [Changing a VPC to use No DHCP Options](#DHCP_Use_No_Options)\.

**Note**  
The following procedure assumes that you've already created the DHCP options set you want to change to\. If you haven't, create the options set now\. For more information, see [Creating a DHCP Options Set](#CreatingaDHCPOptionSet)\.

**To change the DHCP options set associated with a VPC**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Your VPCs**\.

1. Select the VPC, and select **Edit DHCP Options Set** from the **Actions** list\.

1. In the **DHCP Options Set** list, select a set of options from the list, and then choose **Save**\.

After you associate a new set of DHCP options with the VPC, any existing instances and all new instances that you launch in that VPC use the options\. You don't need to restart or relaunch the instances\. They automatically pick up the changes within a few hours, depending on how frequently the instance renews its DHCP lease\. If you want, you can explicitly renew the lease using the operating system on the instance\.

### Changing a VPC to use No DHCP Options<a name="DHCP_Use_No_Options"></a>

You can set up your VPC to use no set of DHCP options\. 

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Your VPCs**\.

1. Select the VPC, and select **Edit DHCP Options Set** from the **Actions** list\.

1. In the **DHCP Options Set** list, select **No DHCP Options Set** from the list, and then choose **Save**\.

 You don't need to restart or relaunch the instances\. They automatically pick up the changes within a few hours, depending on how frequently the instance renews its DHCP lease\. If you want, you can explicitly renew the lease using the operating system on the instance\. 

### Deleting a DHCP Options Set<a name="DeletingaDHCPOptionSet"></a>

When you no longer need a DHCP options set, use the following procedure to delete it\. The VPC must not be using the set of options\.

**To delete a DHCP options set**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **DHCP Options Sets**\.

1. Select the set of DHCP options to delete, and then choose **Delete**\.

1. In the confirmation dialog box, choose **Yes, Delete**\.

## API and Command Overview<a name="APIOverview"></a>

You can perform the tasks described on this page using the command line or an API\. For more information about the command line interfaces and a list of available APIs, see [Accessing Amazon VPC](what-is-amazon-vpc.md#VPCInterfaces)\.

**Create a set of DHCP options for your VPC**
+ [create\-dhcp\-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-dhcp-options.html) \(AWS CLI\)
+ [New\-EC2DhcpOption](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2DhcpOption.html) \(AWS Tools for Windows PowerShell\)

**Associate a set of DHCP options with the specified VPC, or no DHCP options**
+ [associate\-dhcp\-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/associate-dhcp-options.html) \(AWS CLI\)
+ [Register\-EC2DhcpOption](https://docs.aws.amazon.com/powershell/latest/reference/items/Register-EC2DhcpOption.html) \(AWS Tools for Windows PowerShell\)

**Describes one or more sets of DHCP options**
+ [describe\-dhcp\-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-dhcp-options.html) \(AWS CLI\)
+ [Get\-EC2DhcpOption](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2DhcpOption.html) \(AWS Tools for Windows PowerShell\)

**Deletes a set of DHCP options**
+ [delete\-dhcp\-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-dhcp-options.html) \(AWS CLI\)
+ [Remove\-EC2DhcpOption](https://docs.aws.amazon.com/powershell/latest/reference/items/Remove-EC2DhcpOption.html) \(AWS Tools for Windows PowerShell\)
