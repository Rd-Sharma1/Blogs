# INTRODUCTION
The Domain Name System (DNS) is the backbone of the internet that helps translate the easy to remember domain names into the IP addresses where the machine is actually hosted. A common phone-book analogy is used to define DNS, but is it as simple as that in reality? Lets find out what actually is happening under the hood from the moment we type the domain name in the browser and hit enter till the website is opened; looks like a very small time frame right? What if i tell you that the DNS lookup takes only 50% of that time, rest is the communication with the server and even this ~50% is the worst case scenario.. so lets dive into what is happening in around 20 - 200 milliseconds

# DIG AS A DIAGNOSTIC TOOL
DIG (Domain Information Groper) is a powerful tool in Linux that we can use to understand the DNS lookup (the process of getting the IP address from domain name) process with the help of real public information about different servers and their DNS records. To use the tool side by side you can download it: For macOS: brew install bind For Debian/Ubuntu sudo apt install dnsutils For Arch sudo pacman -S bind-tools For Windows follow this link: https://github.com/trinib/AdGuard-WireGuard-Unbound-DNScrypt/wiki/Install-Dig-&-WSL OR look for dig installation using dig Check using dig -v

# WHY DNS EXISTS?
DNS helps us find the IP address of the server but how does it do this in the background so fast that we do not even realize it. To start off from the moment you type the url for the website you want to visit in your browser, the browser first searches through the cache for the IP address; wait if browser knows the IP addresses itself then why are we bothering with DNS?

#C aching and Why DNS is Still Needed?
Well because what the browser initially searches for is the cache where the previously and commonly visited IPs are stored to avoid repeating same DNS lookup each time you enter www.google.com.Now, if the browser finds the IP then there is no need for DNS lookup but when the cache misses the real deal starts..

# WHY DNS IS HIERARCHICAL?
the DNS resolver receives the IP address and starts what is called as a recursive cycle of finding the IP address. Before we understand how that cycle works we need to tackle one big question first

**Why is DNS not storing a one big database for names and IPs?**
(and yes we are asking this question means it doesn't.) Well the answer is simple if you give it some thought, but just to give some reference there are around ~1.3 billion websites and mapping them in a simple key-value pair would make the whole thing run at the speed of continental drift!

At this point we know it is not a simple dictionary as the phonebook analogy might hint but thenHow is it stored?

The DNS stores this information in a hierarchy, there are 3 levels to this: ->root level dns servers, ->top level dns servers, ->authoritative dns servers...

we will go over how each of them might look like when we will go through an example with the help of the dig command but what we need to know right now is how this hierarchy leads us to the desired IP address.

# ROOT SERVERS
We have established that we do not have a simple key-value mapping but something more profound and complex in place and now lets look at how the DNS resolver goes about getting us the IP address for the domain name it received from the browser. The resolver first queries the root server and the nearest root server responds with an IP address (oh simple we are done then? ah.. not exactly), this IP address is not the one we are looking for, see what the root server sends us is the IP of the TLDS (or Top Level Domain Server) which we should request. What happens here is that the root server maps our top level domain and gives us the IP of server which will know more about the domains under that particular TLD..

The idea is that no DNS server knows everything!
## Understanding root server using dig . NS command

We know the structure of dig command already, now we will experience first-hand what does this actually do and how it works. The command dig . NS is the actual default values of the command dig. This returns a list of all the root servers. There are only 13 root servers in the world that every DNS lookup goes through.. I should also add that 13 servers does not mean there are 13 machines these servers are distributed and they return the IP of TLD server as well as the NS records.

# TLD SERVERS
All that is fine but what exactly is a top level domain that we are talking about? This is something that you see daily in your like but just seems to just don't care about. Have you ever notices how some websites end with .com while others with .ai or .me or here's another popular one .org or .net (there are ccTLDs or country-code TLDs also for e.g., .uk for the UK, .in for India, .jp for Japan). All these are top level domains have something called a TLD server or TLDS (not to be confused with TLDs; multiple)

## Understanding TLD server using 'dig com NS' command

Now we can see a pattern that since dig . NS returned the list of root servers the dig com NS would return a list of tld servers among other things, these are the servers handling the .com domains around the world

At this point the DNS resolver gets the IP address of the TLD Server where the server will return an IP address (and is this the IP we are looking for? Not yet). This is the IP of the Authoritative Name Servers

# AUTHORITATIVE NAME SERVERS
Name Records
During a DNS lookup, the recursive resolver uses NS records to determine where to query next after obtaining the TLD server information, ensuring the request reaches the authoritative name servers for the target domain

Authoritative name servers holds all the name records for that particular domain and these are also the servers that provides DNS resolver the desired IP address of the domain. The Name Server Records specify the authoritative DNS servers responsible for a domain, guiding the DNS lookup to the correct servers that hold the domain's actual DNS records

## Understanding Authoritative Name Servers using dig google.com NS command

dig google.com NS provides detailed information about the IP of the domain in question (google.com in this case) with its name servers. Similar commands with different NS record names can also be used to get information about it (for e.g., dig google.com MX for email protocols or dig google.com A for A record information)

# CONCLUSION
We can now see how actually the DNS lookup is happening behind the scenes and why it could cause a lot of issues if not managed properly.. We have realized that the steps involved in DNS lookup are something like: Browser -> DNS Resolver -> Root Servers -> TLDS -> Authoritative Servers -> DNS Resolver -> Browser. DNS lookup needs to be fast so it uses UDP but at the same time to ensure security(there are other prominent reasons) TCP is also utilized. You can now try different dig commands and explore this domain, try dig -h to understand the command structure.
