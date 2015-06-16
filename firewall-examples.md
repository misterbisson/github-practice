#Title: Cloud Firewall Examples
##Stability: 4
Text:

These examples are meant to outline some common use-cases for the firewall. Note that all instances need to have their firewall property set for rules to apply to them:

```bash
$ sdc-enablemachinefirewall <machine-id>
```

Recall that for instances that have Cloud Firewall feature enabled, the default policy is always:

- allow incoming ping requests
- block all other incoming traffic
- allow all outgoing traffic

The examples here are not mutually exclusive. You can combine the rules in the examples as you see fit.

- [Allow SSH Traffic](CloudFirewallExamples-AllowSSHTraffic)
- [Allow HTTP Traffic](CloudFirewallExamples-AllowHTTPTraffic)
- [Multiple Web and Database Server Setup](CloudFirewallExamples-MultipleWebandDatabaseServerSetup)
- [Bastion Host Setup](CloudFirewallExamples-BastionHostSetup)


### Allow SSH Traffic {#allowSSHtraffic}

To allow SSH access from any IP address to all instances in a datacenter, create the following rule:


```bash
$ sdc-createfirewallrule --enabled \
                         --rule "FROM any TO all vms ALLOW tcp PORT 22"
{
  "id": "589f1458-d42b-4bad-9613-d738ce074225",
  "rule": "FROM any TO all vms ALLOW tcp PORT 22",
  "enabled": true
}
```

To allow SSH to one instance with ID ba2c95e9-1cdf-4295-8253-3fee371374d9, create this rule. Note that without the `—enabled` option, this rule is disabled when it is created.

```bash
sdc-createfirewallrule --rule "FROM any TO vm ba2c95e9-1cdf-4295-8253-3fee371374d9 ALLOW tcp PORT 22"
{
  "id": "0b3adeaf-cfd9-4cbc-a566-148f569c050c",
  "rule": "FROM any TO vm ba2c95e9-1cdf-4295-8253-3fee371374d9 ALLOW tcp PORT 22",
  "enabled": false
}
```

To enable the rule, use this command:

```bash
$ sdc-enablefirewallrule 0b3adeaf-cfd9-4cbc-a566-148f569c050c
{
  "id": "0b3adeaf-cfd9-4cbc-a566-148f569c050c",
  "rule": "FROM any TO vm ba2c95e9-1cdf-4295-8253-3fee371374d9 ALLOW tcp PORT 22",
  "enabled": true
}
```

You can see that both of these rules allow SSH traffic from the Internet. If there is more than one rule that affects incoming traffic, the least restrictive one applies. In this case, the rule that allows SSH traffic to all instances in the datacenter is applied.

If you were to disable that rule, however, only the second rule would apply.


### Allow HTTP Traffic {#allowHTTPtraffic}

To allow HTTP connections from any IP addresss to all instances in a datacenter, create the following rule:


```bash
$ sdc-createfirewallrule --enabled \
                         --rule "FROM any TO all vms ALLOW tcp PORT 80"
{
  "id": "d4f01808-dd52-4fb8-b0e8-951d888e1aaa",
  "rule": "FROM any TO all vms ALLOW tcp PORT 80",
  "enabled": true
}
```


To allow both HTTP and HTTPS connections to all instances in a datacenter, update the rule to include port 443:

```bash
$ sdc-updatefirewallrule --rule "FROM any TO all vms ALLOW tcp (PORT 80 AND PORT 443)" d4f01808-dd52-4fb8-b0e8-951d888e1aaa
{
  "id": "d4f01808-dd52-4fb8-b0e8-951d888e1aaa",
  "rule": "FROM any TO all vms ALLOW tcp (PORT 80 AND PORT 443)",
  "enabled": true
}
```


### Mutliple Web and Database Server Setup {#WebandDatabaseServerSetup}

Suppose that you run a website in which two web servers talk to two database servers. You can use [Working with Instance Tags](Working with Instance Tags.html) to identify each kind of instance.

Give each of the web server a `role` tag with the value `www`:

```bash
$ sdc-addmachinetags --tag "role=www" d06bb2bd-e18d-63c0-acce-b125ee36b9e0
{
  "role": "www"
}
```

And give the database servers a `role` tag with the value `db`.

```bash
$ sdc-addmachinetags --tag "role=db" 2171717d-a15c-6a6d-83ae-f1610f552c13
{
  "role": "db"
}
```

We now need to create firewall rules to control access to these instances.  Recall that by default, instances with Cloud Firewall enabled block all incoming TCP and UDP traffic. We now need to open up the necessary ports for each instance role.

First, we want to allow communication between the web servers and the database servers. We do so by creating this rule:

```bash
$ sdc-createfirewallrule --enabled \
                         --rule "FROM tag role = www TO tag role = db ALLOW tcp PORT 5432"
{
  "id": "82cb2ab3-ff43-4b05-955a-66fcde84c5f8",
  "rule": "FROM tag role = www TO tag role = db ALLOW tcp PORT 5432",
  "enabled": true
}
```

This rule allows **_only_** the web servers to connect to the database servers on the standard PostgreSQL port (5432). All other inbound traffic to the database servers is blocked.

Next, we want to allow HTTP and HTTPS traffic to the web servers from anywhere on the Internet. We do so by creating this rule:

```bash
$ sdc-createfirewallrule --enabled --rule "FROM any TO tag role = www ALLOW tcp (PORT 80 AND PORT 443)"
{
  "id": "36548d03-0436-44d5-bf7b-383e693fd46e",
  "rule": "FROM any TO tag role = www ALLOW tcp (PORT 80 AND PORT 443)",
  "enabled": true
}
```

After you have created both of these rules, instances with the tag `role` set to `db` will have the following behavior:

- Allow incoming TCP traffic on port 5432 from instances with tag `role=www`
- Allow all outgoing traffic
- Allow incoming ping requests
- Block all other incoming traffic


And instances with the tag `role` set to `www` will have the following behavior:

- Allow incoming TCP traffic on ports 80 and 443 from any IP address
- Allow outgoing TCP traffic on port 5432 to instances with  `tag role=www`
- Allow all outgoing traffic
- Allow incoming ping requests
- Block all other incoming traffic

Creating additional instances with the role tags listed above will automatically apply these rules. For example, to apply the web server rules to a new server, just give it tag `role=www`.


### Bastion Host Setup {#BastionHostSetup}

In this setup, we have the following requirements:

1. Instances are allowed access from the bastion host on all ports
2. Instances block all other connections
3. The bastion host accepts SSH connections from only certain IP addresses and no others.


Recall that the default policy is to block all incoming connections, so requirement 2 is taken care of. We then need two rules to handle the other requirements.

The bastion host has the id 99a640b6-476f-ee0b-e2b0-b5146d6beb9f. To allow all traffic from the bastion host to all of the instances, you would create this rule:

```bash
$ sdc-createfirewallrule --enable \
                         --rule "FROM vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f TO all vms ALLOW tcp PORT all"
{
  "id": "23831805-bae6-41f0-8f04-bef1191443d4",
  "rule": "FROM vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f TO all vms ALLOW tcp PORT all",
  "enabled": true
}
```

The second requirement is that the bastion host should accept SSH connections only from certain IP addresses. To do that you use this rule:

```bash
$ sdc-createfirewallrule --enabled \
                         --rule "FROM (ip 172.1.1.110 OR ip 172.1.1.111) to vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f ALLOW tcp PORT 22"
{
  "id": "5494849f-bbaa-41a9-b92e-21d1d61bf656",
  "rule": "FROM (ip 172.1.1.110 OR ip 172.1.1.111) TO vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f ALLOW tcp PORT 22",
  "enabled": true
}
```

When you create new instances, they will have access from the bastion host.