## Anchors

*Anchors* are used for sourcing rules into the main ruleset, either on-demand with `pfctl`, or from an external text file. They can be any combination of rule snippets, tables, and even other anchors. For example we can create a table that includes random IP addresses that we want to ban from the system. These are completely arbitrary addresses for demonstration purposes. You can use any combination of addresses as long as they are logical. For example you cannot use 111.222.333.444

Create a file named `/etc/random`:
```command
sudo vim /etc/random
```

Add the following:
```
ext_if = "em0"

table <random> { 157.247.226.123 157.247.226.124 157.247.226.125 }

block in quick on $ext_if from <random> 
block return out quick on egress to <random>
```

Add the anchor above the `block all` rule in your `pf.conf`:
```
*--snip--*
anchor random_anchor
load anchor random_anchor from "/etc/random"
block all
*--snip--*
```

Load the ruleset:
```command
sudo pfctl -f /etc/pf.conf
```

Verify the anchor is running:
```command
sudo pfctl -s Anchors
```

Show the anchor rules:
```command
sudo pfctl -a banned_anchor -s rules
```

View the anchor in the ruleset:
```command
sudo pfctl -s rules
```

*Anchors* can also be used to insert rules on-demand without having to reload the main ruleset. This can be useful for testing, quick-fixes, emergencies, etc. Let's create an anchor that blocks internal hosts. We'll name it `rogue_hosts`. Once the anchor is in place, we can use it anytime. 

Add the anchor `rogue_hosts` above the `block all` rule in `pf.conf`:
```
*--snip--*
anchor rogue_hosts
block all
*--snip--*
```

Reload the ruleset:
```command
sudo pfctl -f /etc/pf.conf
```

Verify the anchor is running:
```command
sudo pfctl -s Anchors
```

View the anchor in the ruleset:
```command
sudo pfctl -s rules
```

Now we add a rule that blocks the host:
```command
sudo sh -c 'echo "block return out quick on egress from XXX.XXX.XX.XX" | pfctl -a rogue_hosts -f -'
```

Show the anchor rules:
```command
sudo pfctl -a rogue_hosts -s rules
```

You should see this:
```
block return out quick on egress inet from XXX.XXX.XX.XX to any
```

There are many pros and cons to using *anchors*, all of which are dependent on your needs. They can help reduce clutter from the main ruleset, but managing multiple files could be burdensome. They can be useful for adding rules on-demand, but someone must be present to add those rules. In other scenarios they are necessary for external applications that are designed to integrate with [PF](https://www.freebsd.org/cgi/man.cgi?query=pf&apropos=0&sektion=0&manpath=FreeBSD+12.0-RELEASE+and+Ports&arch=default&format=html).



