Namecheap to AWS Route53 DNS Migration story
============================================

.. author:: default
.. categories:: none
.. tags:: dns, namecheap, route53, aws, sysadmin
.. comments::


System administration is not my job but then sometimes I need to wear that hat to help the team. Here is how I managed DNS migration from Namecheap to AWS Route53.

Get the Zone file
-----------------

Namecheap support was kind enough to send me zone file when I requested for one.

Format the zone file
--------------------

- Remove whitespeces in the beginning of all lines in zone file
- Add below text as first line of zone file
- Make sure *quote* all TXT record values

:: 
    
    @ORIGIN example.com

Install cli53

::

    pip install cli53

Zone export 
---------------

::

    cli53 create example.com
    cli53 import example.com --file example.com.zone

    

Verify
-------

Find out your name server. Logon to AWS Route 53 console and find out NS entries. Pick one. In my case it was *ns-1261.awsdns-29.org*

::

    pip install \
        git+http://github.com/joemiller/dns_compare.git#egg=dns_compare
    dns_compare -z example.com --file example.com.zone \
        --server 8.8.8.8 -t false
    dns_compare -z example.com --file example.com.zone \
        --server ns-1261.awsdns-29.org # use your aws ns
    # -- OR --- use -t to ignore ttl differences
    dns_compare -z example.com --file example.com.zone \
        --server ns-1261.awsdns-29.org -t false

Finally
-------

.. note:: **Do understand steps below will cause some downtime (number of hours).**

- Go to namecheap's "Transfer DNS to Webhost" page for your domain. Add new name servers. Save.

::

    watch dig -t ANY @8.8.8.8 example.com
