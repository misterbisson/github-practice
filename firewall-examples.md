Title: Cloud Firewall Examples
----
Stability: 4
----
Text:

				    <div class='panelMacro'><table class='infoMacro'><colgroup><col width='24'><col></colgroup><tr><td valign='top'><img src="/assets/images/icons/emoticons/information.gif" width="16" height="16" align="absmiddle" alt="" border="0"></td><td><b>Beta</b><br />Cloud Firewall is in beta at this time.</td></tr></table></div>

<p>These examples are meant to outline some common use-cases for the firewall. Note that all instances need to have their firewall property set for rules to apply to them:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-enablemachinefirewall &lt;machine-id&gt;
</pre>
</div></div>


<p>Recall that for instances that have Cloud Firewall feature enabled, the default policy is always:</p>

<ul>
	<li>allow incoming ping requests</li>
	<li>block all other incoming traffic</li>
	<li>allow all outgoing traffic</li>
</ul>


<p>The examples here are not mutually exclusive. You can combine the rules in the examples as you see fit.</p>

<div>
<ul>
    <li><a href='#CloudFirewallExamples-AllowSSHTraffic'>Allow SSH Traffic</a></li>
    <li><a href='#CloudFirewallExamples-AllowHTTPTraffic'>Allow HTTP Traffic</a></li>
    <li><a href='#CloudFirewallExamples-MultipleWebandDatabaseServerSetup'>Multiple Web and Database Server Setup</a></li>
    <li><a href='#CloudFirewallExamples-BastionHostSetup'>Bastion Host Setup</a></li>
</ul></div>

<h1><a name="CloudFirewallExamples-AllowSSHTraffic"></a>Allow SSH Traffic</h1>

<p>To allow SSH access from any IP address to all instances in a datacenter, create the following rule:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-createfirewallrule --enabled \
                         --rule "FROM any TO all vms ALLOW tcp PORT 22"
{
  "id": "589f1458-d42b-4bad-9613-d738ce074225",
  "rule": "FROM any TO all vms ALLOW tcp PORT 22",
  "enabled": true
}
</pre>
</div></div>


<p>To allow SSH to one instance with ID ba2c95e9-1cdf-4295-8253-3fee371374d9, create this rule. Note that without the <tt>--enabled</tt> option, this rule is disabled when it is created.</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
sdc-createfirewallrule --rule "FROM any TO vm ba2c95e9-1cdf-4295-8253-3fee371374d9 ALLOW tcp PORT 22"
{
  "id": "0b3adeaf-cfd9-4cbc-a566-148f569c050c",
  "rule": "FROM any TO vm ba2c95e9-1cdf-4295-8253-3fee371374d9 ALLOW tcp PORT 22",
  "enabled": false
}
</pre>
</div></div>

<p>To enable the rule, use this command:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-enablefirewallrule 0b3adeaf-cfd9-4cbc-a566-148f569c050c
{
  "id": "0b3adeaf-cfd9-4cbc-a566-148f569c050c",
  "rule": "FROM any TO vm ba2c95e9-1cdf-4295-8253-3fee371374d9 ALLOW tcp PORT 22",
  "enabled": true
}
</pre>
</div></div>

<p>You can see that both of these rules allow SSH traffic from the Internet. If there is more than one rule that affects incoming traffic, the least restrictive one applies. In this case, the rule that allows SSH traffic to all instances in the datacenter is applied.</p>

<p>If you were to disable that rule, however, only the second rule would apply.</p>



<h1><a name="CloudFirewallExamples-AllowHTTPTraffic"></a>Allow HTTP Traffic</h1>

<p>To allow HTTP connections from any IP addresss to all instances in a datacenter, create the following rule:</p>


<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-createfirewallrule --enabled \
                         --rule "FROM any TO all vms ALLOW tcp PORT 80"
{
  "id": "d4f01808-dd52-4fb8-b0e8-951d888e1aaa",
  "rule": "FROM any TO all vms ALLOW tcp PORT 80",
  "enabled": true
}
</pre>
</div></div>


<p>To allow both HTTP and HTTPS connections to all instances in a datacenter, update the rule to include port 443:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-updatefirewallrule --rule "FROM any TO all vms ALLOW tcp (PORT 80 AND PORT 443)" d4f01808-dd52-4fb8-b0e8-951d888e1aaa
{
  "id": "d4f01808-dd52-4fb8-b0e8-951d888e1aaa",
  "rule": "FROM any TO all vms ALLOW tcp (PORT 80 AND PORT 443)",
  "enabled": true
}
</pre>
</div></div>


<h1><a name="CloudFirewallExamples-MultipleWebandDatabaseServerSetup"></a>Multiple Web and Database Server Setup</h1>

<p>Suppose that you run a website in which two web servers talk to two database servers. You can use <a href="Working with Instance Tags.html" title="Working with Instance Tags">tags</a> to identify each kind of instance.</p>

<p>Give each of the web server a <tt>role</tt> tag with the value <tt>www</tt>:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-addmachinetags --tag "role=www" d06bb2bd-e18d-63c0-acce-b125ee36b9e0
{
  "role": "www"
}
</pre>
</div></div>

<p>And give the database servers a <tt>role</tt> tag with the value <tt>db</tt>.</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-addmachinetags --tag "role=db" 2171717d-a15c-6a6d-83ae-f1610f552c13
{
  "role": "db"
}
</pre>
</div></div>

<p>We now need to create firewall rules to control access to these instances.  Recall that by default, instances with Cloud Firewall enabled block all incoming TCP and UDP traffic. We now need to open up the necessary ports for each instance role.</p>

<p>First, we want to allow communication between the web servers and the database<br/>
servers. We do so by creating this rule:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-createfirewallrule --enabled \
                         --rule "FROM tag role = www TO tag role = db ALLOW tcp PORT 5432"
{
  "id": "82cb2ab3-ff43-4b05-955a-66fcde84c5f8",
  "rule": "FROM tag role = www TO tag role = db ALLOW tcp PORT 5432",
  "enabled": true
}
</pre>
</div></div>

<p>This rule allows *<b>only</b>* the web servers to connect to the database servers on the standard PostgreSQL port (5432). All other inbound traffic to the database servers is blocked.</p>

<p>Next, we want to allow HTTP and HTTPS traffic to the web servers from anywhere<br/>
on the Internet. We do so by creating this rule:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-createfirewallrule --enabled --rule "FROM any TO tag role = www ALLOW tcp (PORT 80 AND PORT 443)"
{
  "id": "36548d03-0436-44d5-bf7b-383e693fd46e",
  "rule": "FROM any TO tag role = www ALLOW tcp (PORT 80 AND PORT 443)",
  "enabled": true
}
</pre>
</div></div>

<p>After you have created both of these rules, instances with the tag <tt>role</tt> set to <tt>db</tt> will have the following behavior:</p>

<ul>
	<li>Allow incoming TCP traffic on port 5432 from instances with tag <tt>role=www</tt></li>
	<li>Allow all outgoing traffic</li>
	<li>Allow incoming ping requests</li>
	<li>Block all other incoming traffic</li>
</ul>


<p>And instances with the tag <tt>role</tt> set to <tt>www</tt> will have the following behavior:</p>

<ul>
	<li>Allow incoming TCP traffic on ports 80 and 443 from any IP address</li>
	<li>Allow outgoing TCP traffic on port 5432 to instances with  <tt>tag role=www</tt></li>
	<li>Allow all outgoing traffic</li>
	<li>Allow incoming ping requests</li>
	<li>Block all other incoming traffic</li>
</ul>


<p>Creating additional instances with the role tags listed above will automatically<br/>
apply these rules. For example, to apply the web server rules to a new server,<br/>
just give it tag <tt>role=www</tt>.</p>


<h1><a name="CloudFirewallExamples-BastionHostSetup"></a>Bastion Host Setup</h1>

<p>In this setup, we have the following requirements:</p>

<ol>
	<li>Instances are allowed access from the bastion host on all ports</li>
	<li>Instances block all other connections</li>
	<li>The bastion host accepts SSH connections from only certain IP addresses and no others.</li>
</ol>


<p>Recall that the default policy is to block all incoming connecions, so<br/>
requirement 2 is taken care of. We then need two rules to handle the other<br/>
requirements.</p>

<p>The bastion host has the id 99a640b6-476f-ee0b-e2b0-b5146d6beb9f. To allow all traffic from the bastion host to all of the instances, you would create this rule:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-createfirewallrule --enable \
                         --rule "FROM vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f TO all vms ALLOW tcp PORT all"
{
  "id": "23831805-bae6-41f0-8f04-bef1191443d4",
  "rule": "FROM vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f TO all vms ALLOW tcp PORT all",
  "enabled": true
}
</pre>
</div></div>

<p>The second requirement is that the bastion host should accept SSH connections only from certain IP addresses. To do that you use this rule:</p>

<div class="code panel" style="border-width: 1px;"><div class="codeContent panelContent">
<pre class="theme: Default; brush: plain; gutter: false" style="font-size:12px; font-family: ConfluenceInstalledFont,monospace;">
$ sdc-createfirewallrule --enabled \
                         --rule "FROM (ip 172.1.1.110 OR ip 172.1.1.111) to vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f ALLOW tcp PORT 22"
{
  "id": "5494849f-bbaa-41a9-b92e-21d1d61bf656",
  "rule": "FROM (ip 172.1.1.110 OR ip 172.1.1.111) TO vm 99a640b6-476f-ee0b-e2b0-b5146d6beb9f ALLOW tcp PORT 22",
  "enabled": true
}
</pre>
</div></div>

<p>When you create new instances, they will have access from the bastion host.</p>
