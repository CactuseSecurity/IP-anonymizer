# IP anonymizer

cactus-anonymizer.pl - replace IP addresses with anonymized IPs as well as text with anonymized text in plain text files

SYNOPSIS
  ./cactus-anonymizer.pl [-txt-subst-file=/var/tmp/strings.txt] [-net="192.168.0.0/16"] config-file1 config-file2 ...

DESCRIPTION

This is a script for 

a) replacing IP addresses in plain text with anonymized equivalents from the network range supplied.

b) replacing strings in a file with anonymized strings

Input is a number of ASCII files (all parameters not starting with -)
IP addresses as well as strings are replaced  one-for-one throughout 
all text files, so once an IP address has an anonymized equivalent, 
it stays that way. 

This is useful if you need to use production configuration data for testing.
E.g. from firewalls but do not want to expose the production data on a
test system. This way you can protect an organization's 
identity at the same time.

DEPENDENCIES

Needs netaddr-ip and cgi modules for perl. E.g. on Ubuntu 22.04:

     sudo apt install libcgi-pm-perl
     sudo apt install libnetaddr-ip-perl

CAVEATS 

- currently only implemented for IPv4

- beware of anonymizing common strings; e.g. "INT" when handling database dumps is part of keyword CONSTRAINT
  use slightly longer strings like "INT_" instead

PARAMETERS
- The network range used for replacement, is set to "10.0.0.0/8" if omitted.

- For each file <infile> supplied an anonymized file called 
  <infile>.anonymized is created.

The second argument is a network address, which should be given in
CIDR notation, and really represents a range of IP addresses from
which we can draw from while doing the IP address substitutions (Note
that the use of NetAddr::IP means that we will never overflow this
range - but it will wrap around if we increment it enough). Using an
RFC1918 private address range is a good idea.

Note that the script tries to handle network addresses so that 
network address and netmask (both given in 255.255.255.x notation
as well as a.b.c.d/xy notation) will match by simply setting 
all netmasks to /32. 

EXAMPLES

 a) ./cactus-anonymizer.pl -net=172.20.0.0/21 -txt-subst-file=/var/tmp/strings.txt /var/tmp/fw.cfg /var/tmp/router9.cfg

 b) cactus-anonymizer.pl -txt-subst-file=strings.txt /tmp/netscreen1.cfg
 
    Output:
    no net specified, using default net 10.0.0.0/8
    anonymizing: /var/tmp/netscreen1.cfg ... result file = /var/tmp/netscreen1.cfg.anonymized
    Anonymized 20197 ip addresses and 150 strings in 31.1 seconds (0.46 Mbytes/second).

 
  c) Anonymizing a whole (ASCII) Postgresql database:

     creating an ASCII dump of the database:
       pg_dump -U dbadmin -d cactusdb -W >/var/tmp/cactus_db.dump.sql

     or as postgres user:
       pg_dump -d cactusdb >/var/tmp/cactus_db.dump.sql

     turn binary .Fc dump into ascii (only necessary if you do not already have an ascii dump):
       pg_restore /var/tmp/cactus_db.dump.Fc >/var/tmp/cactus_db.dump.sql

     anonymizing:
       cactus-anonymizer.pl -txt-subst-file=/var/tmp/strings.txt /var/tmp/cactus_db.dump.sql

     restoring anonymized database:
       psql --set ON_ERROR_STOP=on targetdb </var/tmp/cactus_db.dump.sql

TODO

- reliably replace network address by networks with consistent netmasks
  (currently all networks are reduced to a /32 netmask)

AUTHOR

Tim Purschke tmp@cactus.de, Cactus eSecurity GmbH, Germany
