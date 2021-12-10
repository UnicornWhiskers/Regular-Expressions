# :heart: Regular-Expressions :heart:

This is repo is mainly to keep track of examples of using Regex for various tasks.


## Using SED in place of cut

Had a list of MITRE Attack techniques used accross a few attacks that were documented in (thedfirreport)[https://thedfirreport.com/] and I wanted to just list out hte Unique techniques. So immediatly I think that I should go with the every handey "cat file | sort | uniq" but potentialy due to the way I had entered the data it wasn't working for me. So I decided to try SED which I've used in the past but I always seem to struggle with much more then when using other implementatinos of regex. This is almost certainly a user issue.

Data looks like so:
```
$ cat dfirreports.txt
OS Credential Dumping – T1003
SMB/Windows Admin Shares – T1021.002
System Owner/User Discovery – T1033
Network Service Scanning – T1046
Windows Management Instrumentation – T1047
Scheduled Task/Job – T1053
Process Injection – T1055
PowerShell – T1059.001
Domain Groups – T1069.002
File and Directory Discovery – T1083
Access Token Manipulation – T1134
Network Share Discovery – T1135
Domain Trust Discovery – T1482
```
There are some basic MITRE Att&ck techniqes as well as sub techniques.   I would like to extract the main technique and subtechnique if it exists in order sort.

I like to use [https://regex101.com/] with some TLP white sample data to test out my regex statement.

but you can also simplify your SED testing with a simple string echo'd to standard out
```
echo "this is my test string - T1000.001"
```
To capture the last line I'll try to define its structure as a regular expression.  

I have a capital T followed by 4 digits between 0 and 9 and then followed optionally by a period and 3 digits betwen 0 and 9 and that should immediatly be followed by a hidden end of line or line-feed character.

So for the parent technique:  

T[0-9]{4}

For the subtechnique

\.[0-9]{3}

But to make this optional I need to put this into a capture group (in brackets) and followed by the question mark (?) which indicates teh preceeding group shoudl match zero or more times.

So that should look like:

T[0-9]{4}(\.[0-9]{3})?

And then I want to put the end of line anchor "$".  Note that you might have to fix your line feeds if you created yoru list in Windows which was my experience.  You can use sed for this as well:

```
$ file dfirreport.txt
dfirreport.txt: ASCII text, with CRLF line terminators
$ sed -i 's/\r$//' dfirreports.txt
$ file dfirreport1.txt
dfirreport1.txt: ASCII text
```

T[0-9]{4}(\.[0-9]{3})?$

Testing this againts my test string in Regex101.com I get the following:
```
match 1: T1000.001
group 1: .001
```
Using SED I get the following:
$ echo "this is my test string - T1000.001" | sed -r 's/T[0-9]{4}(\.[0-9]{3})?$//'
this is my test string -

Which looks good.  now testing with another string without the sub technique
```
$ echo "this is my second test string - T1000" | sed -r 's/T[0-9]{4}(\.[0-9]{3})?$//'
this is my second test string -
```
This also appears to be working good.  I just need to extract it instead of removeing it.  I'll do this by putting the who match in a capture group.

(T[0-9]{4}(\.[0-9]{3})?)$
```
$echo "this is my test string - T1000.001" | sed -r 's/(T[0-9]{4}(\.[0-9]{3})?)$/\1/'
this is my test string - T1000.001
```
So that output the entire string.  Just to confirm that it is matching, I'll try outputing it in brackets
```
$ echo "this is my test string - T1000.001" | sed -r 's/(T[0-9]{4}(\.[0-9]{3})?$)/(\1)/'
this is my test string - (T1000.001)
```
So it is working, but SED prefers to outout the rest of the string along with substitution, which is probably the ideal functionality in other circumstances.  I guess I'll capture the rest of the string as another capture group and then choose which one I want to output.

(.*)(T[0-9]{4}(\.[0-9]{3})?)$

```
echo "this is my test string - T1000.001" | sed -r 's/(.*)(T[0-9]{4}(\.[0-9]{3})?$)/\2/'
T1000.001
```

This is great, so now I can test it out on the main file:
```
cat dfirreports.txt | sed -r 's/(.*)(T[0-9]{4}(\.[0-9]{3})?$)/\2/'
T1566.001
T1078.002
T1059
T1204
T1059.001
T1059.003
T1204.002
T1136
T1078
T1087.001

$ cat dfirreports.txt | sed -r 's/(.*)(T[0-9]{4}(\.[0-9]{3})?$)/\2/' | wc -l
93

$ cat dfirreports.txt | sed -r 's/(.*)(T[0-9]{4}(\.[0-9]{3})?$)/\2/' | sort | uniq | wc -l
55

```




