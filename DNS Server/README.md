# Domain Name System (DNS) Server

# Building Your Own DNS Server: A Step-by-Step Guide

Every time you type a website address into your browser, you interact with the Domain Name System (DNS). This system is crucial because it maps human-readable domain names to the IP addresses that computers use to identify each other on the network. In this guide, we will delve into what a DNS server is, how it works, and how you can build your own DNS server using Node.js.

## What is a DNS Server?

A DNS server is essentially a computer server that maintains a database of public IP addresses and their associated hostnames. The primary function of DNS servers is to translate domain names, which are easy for people to remember, into IP addresses, which computers use to establish connections. For example, when you enter <www.example.com> into your browser, a DNS server translates that into an IP address like 192.0.2.1.

## How DNS Works

The process begins when you type a domain name into a browser. The browser sends a request to a DNS server to translate the domain into an IP address. Once the IP address is received, the browser can initiate a connection to the server hosting the website content. This entire process is known as a DNS lookup.

### Understanding DNS Resolution

#### DNS Hierarchy and Root Servers

The DNS system is structured hierarchically. At the top level are the root servers; there are 13 sets of these worldwide. These root servers are pivotal as they manage the database for top-level domains (TLDs) like .com, .net, .org, and newer ones like .app and .dev.

#### Why are there only 13 DNS root server addresses?

A common misconception is that there are only 13 root servers in the world. In reality, there are many more, but still only 13 IP addresses used to query the different root server networks. Limitations in the original architecture of DNS require there to be a maximum of 13 server addresses in the root zone. In the early days of the Internet, there was only one server for each of the 13 IP addresses, most of which were located in the United States.

Today each of the 13 IP addresses has several servers, which use Anycast routing to distribute requests based on load and proximity. Right now there are over 600 different DNS root servers distributed across every populated continent on earth.

#### Example of DNS Resolution

Let's follow a DNS query step-by-step:

1. **Query Initiation**: A user enters example.com into their browser.
2. **Root Server Query**: The query first reaches a DNS root server. The root server doesn't know the exact IP address but knows where to direct the query for .com domains.
3. **TLD Server Query**: The root server directs the query to a TLD server for .com. The TLD server finds the nameserver handling the specific domain example.com.
4. **Domain Nameserver Query**: The query is then sent to the domain's nameserver, which could be managed by a service like Google DNS or Cloudflare.
5. **IP Resolution**: The nameserver returns the IP address for example.com, which is then used by the browser to connect to the web server and load the website.

## Key Players in DNS

### Major DNS Providers

Services like Google DNS (8.8.8.8) and Cloudflare (1.1.1.1) are major DNS providers. They offer fast DNS resolution services that are often more reliable and quicker than those provided by local ISPs. These providers also enhance privacy and security features for DNS queries.

### DNS Record Types

#### Common DNS Records

- **A Record**: Directs a domain to a specific IPv4 address.
- **CNAME Record**: Points a domain or subdomain to another domain name, allowing for easier management of multiple domain names.
- **NS Record**: Delegates a domain or subdomain to use particular nameservers, essentially directing where the DNS queries should be sent.

## Practical Examples

Using saquib.dev:

- A record for saquib.dev could point to 76.76.1.2.
- CNAME record for app.spotify.com might point to saquib.dev, indicating that app.spotify.com is an alias for saquib.dev.
- NS record for app.saquib.dev could delegate DNS responsibilities to ns.myserver.com, indicating that this server now has authority over app.saquib.dev.

## Building a DNS Server in Node.js

### Setting Up a Basic Server

Here's a simple script to create a UDP server that listens on port 53, which is the standard port for DNS services:

```javascript
const dgram = require("node:dgram");
const server = dgram.createSocket("udp4");

server.on("message", (msg, rinfo) => {
  console.log({ msg: msg.toString(), rinfo });
});

server.bind(53, () => console.log("DNS Server is running on port 53"));
```

### Handling DNS Queries with dns-packet

To decode DNS queries and send responses, we use the dns-packet library:

```javascript
const dnsPacket = require("dns-packet");
const server = dgram.createSocket("udp4");

const db = {
  "ksaquib.surge.sh": {
    type: 'A', 
    data: '1.2.3.4'
  },
  "blog.ksaquib.dev": {
    type: 'CNAME', 
    data: 'saquib.com'
  },
};

server.on("message", (msg, rinfo) => {
  const incomingReq = dnsPacket.decode(msg);
  const ipFromDb = db[incomingReq.questions[0].name];

  const ans = dnsPacket.encode({
    type: "response",
    id: incomingReq.id,
    flags: dnsPacket.AUTHORITATIVE_ANSWER,
    questions: incomingReq.questions,
    answers: [
      {
        type: ipFromDb.type,
        class: "IN",
        name: incomingReq.questions[0].name,
        data: ipFromDb.data,
      },
    ],
  });
  server.send(ans, rinfo.port, rinfo.address);
});

server.bind(53, () => console.log("DNS Server is running on port 53"));
```

### Testing Your DNS Server

#### Using dig Command

You can test your DNS server with the dig command as follows:

```bash
dig @localhost ksaquib.surge.sh
```

This command should return the IP address you defined in your database, verifying that your server can handle DNS queries properly.
