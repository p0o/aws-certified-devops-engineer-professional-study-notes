
### Network ACL

- default NACL allows all inbound and outbound traffic
- Rules are evaluated starting with the lowest numbered rule. As soon as a rule matches traffic, it's applied regardless of any higher-numbered rule that might contradict it
	- It means specific port rules should be placed before rules for larger port range (e.g ephemeral ports)
- Reachability Analyzer is a static configuration analysis tool. Use Reachability Analyzer to analyze and debug network reachability between two resources in your VPC

Every subnet should be associated to a route table. You can explicitly associate it otherwise subent is implicitly associated with main route table.

You can associate multiple subnets to one route table but one subnet can only be associated with one route table.


### Availability Zones

- AZs are within 100 KM (60 miles) of each other
- 