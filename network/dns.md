# Domain Name System (DNS) Notes

DNS (Domain Name System) translates human-readable domain names (like `www.google.com`) into computer-readable IP addresses (like `142.250.190.46`). It functions as the phonebook of the internet.

---

## 1. How DNS Resolution Works

When you visit a website, your computer goes through several steps to resolve the IP address:

1. **DNS Recursor (Resolver)**: Usually operated by your ISP or a public provider (like Google `8.8.8.8` or Cloudflare `1.1.1.1`). It acts as a librarian, querying other servers to locate the address.
2. **Root Name Server**: The first stop for resolving names. It reads the top-level domain (TLD) from right to left (e.g., `.com`) and directs the recursor to the TLD server.
3. **TLD Name Server**: Manages DNS information for specific extensions (like `.com`, `.net`, `.org`). It points the recursor to the authoritative name server for the target domain.
4. **Authoritative Name Server**: The final stop. This server holds the actual DNS records for the domain. It returns the correct IP address to the recursor, which forwards it to your browser.

---

## 2. Common DNS Record Types

DNS configurations use different types of records depending on the destination:

| Record Type | Name | Purpose | Example |
| :--- | :--- | :--- | :--- |
| **`A`** | Address Record | Maps a domain/subdomain to an **IPv4** address. | `google.com` $\rightarrow$ `142.250.190.46` |
| **`AAAA`** | IPv6 Address Record | Maps a domain/subdomain to an **IPv6** address. | `google.com` $\rightarrow$ `2607:f8b0:4005:805::200e` |
| **`CNAME`** | Canonical Name | Maps an alias/subdomain to another domain name (not an IP). | `blog.example.com` $\rightarrow$ `example.github.io` |
| **`MX`** | Mail Exchanger | Directs email to the domain's mail servers. | `mail.example.com` (Priority 10) |
| **`TXT`** | Text Record | Stores arbitrary text notes, often used for domain verification, SPF, and DKIM security policies. | `v=spf1 include:_spf.google.com ~all` |
| **`NS`** | Name Server | Specifies the authoritative name servers for the domain. | `ns1.hover.com`, `ns2.hover.com` |

---

## 3. TTL (Time to Live)

**TTL** is a setting on DNS records that tells caching servers (like resolvers or web browsers) how long (in seconds) they should store the record before requesting a fresh copy from the authoritative name server.
- **Low TTL (e.g., 300 seconds)**: Quick updates during DNS changes, but increases load on authoritative servers.
- **High TTL (e.g., 86400 seconds / 24 hours)**: Reduces query overhead and speeds up loading times, but DNS updates will take longer to propagate globally.

---

## 4. Diagnostic Commands (Examples)

Use these command-line tools to lookup and troubleshoot DNS configurations:

### `nslookup` (Cross-platform)
Query a domain's A record:
```bash
nslookup google.com
```

### `dig` (Linux/Mac tool for verbose details)
Lookup A records:
```bash
dig google.com
```

Lookup MX records (mail servers):
```bash
dig google.com MX
```

Query a specific DNS server (e.g., using Cloudflare's `1.1.1.1` resolver):
```bash
dig @1.1.1.1 google.com
```
