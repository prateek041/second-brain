DNS stands for Domain name system. It is like a phone book, that maps a name to a number, where the name is the website name and the number is the IP address.

## DNS Lookup lifecycle
- you search for a web address like www.example.com in browser.
- **Browser Cache**: browser checks the local browser cache to find a match.
- **Operating System Cache**: If not found on browser cache, local operating system cache is looked up.
- **Recursive DNS Servers**: If not found even in OS DNS cache, the query moves onto the recursive DNS server configured by ISP (Internet Service Provider).
- **Root Nameserver**: If the recursive server doesn't know the answer, the request is forwarded to the Root nameserver. Which itself doesn't know the answer but directs the query to a TLD (Top Level Domain) server depending on the extension(.com, .net).
- **TLD Nameserver**: This server manages domain's information. It doesn't have the IP either but tells the recursive server which nameserver (authoritative nameserver) controls the data for`example.com`.
- **Authoritative name-server**: has the actual IP address which it sends back to the recursive DNS server.
- **Caching**: The recursive server caches the IP address and send the response to your device. This cached response has a TTL (Time to Live), after which it is deleted.
- **Browser Connects**: Finally the browser has the IP address and it successfully connects to it.

