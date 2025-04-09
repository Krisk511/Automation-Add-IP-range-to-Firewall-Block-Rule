# Automation-Add-IP-range-to-Firewall-Block-Rule
Automation of IP range firewall block rules from a threat feed

## Objective
- Access a list of IP addresses from a threat intelligence feed which have been identified as malicious actors.
- Write a script that adds this list to your Firewall Block rules
- Automate script to run every day

## 

curl 127.0.0.1/evil_IP.feed
The lab environment does not have internet access. Therefore, this threat intelligence feed is being simulated for this exercise. This simulated feed contains IP address ranges that were known to be the source of malicious actors in early 2023. These IP address ranges are from the FireHOL service at iplists.firehol.org.

This command retrieves and displays the contents of the IP threat feed to the screen.

Enter the following to add a new iptables rule to block communications from the first IP address range from the feed:

iptables -A INPUT -s 5.134.128.0/19 -j DROP
Enter iptables -S to view the current iptables rules.

You realize that manually creating rules for each IP address range from a threat feed will be quite tedious and a task you may need to perform daily. So, you decide to automate the process.

Enter vim ip_block.sh to create a new script file.

Type i to enter insert mode. The message -- INSERT -- should be present at the bottom of the screen.

Type the following into the empty document area of vim:

#!/bin/bash

# Retrieve IP addresses with CIDR notation from an HTML file using regex
IPS=($(curl -s 127.0.0.1/evil_IP.feed | grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}(/[0-9]{1,2})?"))

# Add an IP block to iptables for each IP address
for IP in "${IPS[@]}"
do
  iptables -A INPUT -s $IP -j DROP
  echo "IP address range drop rule for $IP added to iptables"
done
This script will retrieve the IP address threat feed, extract the IP address and CIDR notations, then, for each IP address range, it will add a drop rule to iptables.

You may skip the 2nd and 3rd comment lines that start with the octothorpe (i.e., #). But they are presented here for clarity. Comment lines begin with an octothorpe and end with a carriage return line feed (i.e., an Enter).

Be careful to pay attention to the curly brackets (i.e., '{' & '}'), the square brackets (i.e., '[' & ']'), and the parenthesis (i.e., '(' & ')'). These each serve a different and important purpose and cannot be interchanged. The code needs to be typed EXACTLY as presented, including capitalization.

When you start the first line of your file with an octothorpe (i.e., #), vim will likely automatically prepend each subsequent line with one as well. When you press Enter after typing the first line of "#!/bin/bash", you will need to delete the octothorpe that automatically appears on the second line. This will prevent every other line from also being prepended with an octothorpe.

If you also wanted to add rules to iptables to drop outbound packets, which of the following lines could be added to the script?

iptables -A EGRESS -s $IP -j DROP
iptables -A OUTPUT -s $IP -j DROP
iptables -A FORWARD -s $IP -j DROP
iptables -A EXTERNAL -s $IP -j DROP
Congratulations, you have answered the question correctly.

Once finished, press ESC to exit insert mode.

Pressing ESC may cause your browser to exit full-screen mode. If that occurs, press ESC a second time to exit VIM's insert mode. Then you can re-enable full-screen mode from the Display lab interface menu.

Enter :wq to save and quit VIM.

Select the Score button to validate this task:

Enter cat ip_block.sh to view the contents of the script file you have created.

Enter chmod +x ip_block.sh to set the execute permissions on the file.

Enter ./ip_block.sh to execute the script.

You should see several lines of output indicating that IP address range drop rules have been added to iptables.

Enter iptables -S to view the current iptables rules.

You should notice that there is at least one (1) duplicate rule. This will be addressed later in this exercise.

You now have a script that will update the iptables filter rules with IP address ranges from a threat intelligence feed. However, to complete the configuration of security automation, you need to schedule this script to run regularly.

Automate the execution of the script, so it runs daily at 1 AM by entering the following:

echo "0 1 * * * /bin/bash /root/ip_block.sh" | crontab -
This command inserts the scheduled task details into the cron table without having to open the file to edit it in a more manual fashion.

The scheduled task configuration in the quotations of this command has the following elements:

0 : The minute when the job should run.
1 : The hour when the job should run (1 AM).
* : The day of the month when the job should run (every day).
* : The month when the job should run (every month).
* : The day of the week when the job should run (every day of the week).
/bin/bash : specifies that the job should be run using the Bash shell
/root/ip_block.sh : the path to the script that should be executed.
Select the Score button to validate this task:

The string '0 1 * * * /bin/bash /root/ip_block.sh' is present in the file
Task complete
Enter crontab -l to view the scheduled tasks.

The automation of your IP blocking from a threat feed is nearly complete, but it needs testing and refinement. While you could wait until 1 AM tomorrow for the feed to update, you will simulate this with a pre-written script.

Enter ./update_feed.sh to update the threat feed with additional IP addresses.

This command is used to simulate the occurrence of the threat intelligence provider adding new IP ranges to the feed.

Enter ./ip_block.sh to pull the updated threat feed and add iptables block rules.

This command is used to simulate the auto-triggering of the scheduled task.

Now that a day has passed (wink wink), check the iptables rules list by entering iptables -S.

You should see a longer presentation of filter rules than before, including several for the IP address ranges that were recently added to the threat feed. However, you notice that there are several duplicate rules for the same IP address. While the filtering may still function with duplicate rules, this will become an overly large and cumbersome list quickly without addressing the duplicate rule problem.

You need to add an additional command to the scheduled task script to address the duplicate rule issue. Enter vim ip_block.sh to edit the script.

Use the arrow keys on your keyboard to move the cursor to the final line of the existing script.

Type i to enter insert mode. The message -- INSERT -- should be present at the bottom of the screen.

If your cursor is not on an empty line already (because there is no empty line after the "done" statement), move the cursor to the end of the line, then press Enter on your keyboard.

Press Enter on your keyboard again to ensure there is a blank line after the "done" statement and a 2nd empty line where you will type the next statement.

Type the following into the 2nd empty line you have created:

iptables-save | awk '!seen[$0]++' | iptables-restore
The result should look like the following (although you may have left out the comments):

#!/bin/bash

# Retrieve IP addresses with CIDR notation from an HTML file using regex
IPS=($(curl -s 127.0.0.1/evil_IP.feed | grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}(/[0-9]{1,2})?"))

# Add an IP block to iptables for each IP address
for IP in "${IPS[@]}"
do
  iptables -A INPUT -s $IP -j DROP
  echo "IP block $IP added to iptables"
done

iptables-save | awk '!seen[$0]++' | iptables-restore
This additional command statement will export the iptables rule set through the awk command, which will filter out duplicates (by suppressing already-seen lines). Then it will re-import the de-duplicated rule set back into iptables.

Once finished, press ESC to exit insert mode.

Pressing ESC may cause your browser to exit full-screen mode. If that occurs, press ESC a second time to exit VIM's insert mode. Then you can re-enable full-screen mode from the Display lab interface menu.

Enter :wq to save and quit VIM.

Select the Score button to validate this task:

The string 'iptables-save' is present in the file
Task complete
Enter cat ip_block.sh to view the contents of the script file you have edited.

Enter ./ip_block.sh to execute the script you just updated.

This execution of the same IP blocking script will retrieve the threat feed again and add the IP range block rules to iptables which will result in even more duplicates. However, the final line of the script will then remove the duplicates.

Enter iptables -S to view the rules list.



Source: CompTIA CySA+ Assisted Lab
