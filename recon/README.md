# Scripts

- [massdnsParser.py](#massdnsparserpy) - Convert MassDNS results into JSON format, separating it into different fields.
- [headersRecon.py](#headersreconpy) - Collect all response headers from the list of websites provided to it.
- [crtRecon.go](#crtRecongo) - Search for domains using certificates (Using domain or organization name)
- [cnMap.py](#cnMappy) - Resolve CNAME records recursively from a list of domains.

## massdnsParser.py

This script takes as an argument a file resulting from MassDNS and converts it into JSON format, separating it into the fields 'Public A Records,' 'Private A Records,' and 'CNAME Records.'

First, you must manually perform domain resolution using MassDNS:
```sh
massdns -r resolvers-trusted.txt -t A -w massdns_results.txt -o S domains.txt
```
You can obtain a list of trusted resolvers (resolvers-trusted.txt) at the following link:
- [resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

### Usage

```sh
massdnsParser.py [-h] [-o OUTPUT] input_file
massdnsParser.py: error: the following arguments are required: input_file
```

#### Usage Example

```
python3 massdnsParser.py massdns_results.txt -o massdns.json
```

### Output

```json
{
  "Public A Records": {
    "ip1": ["domain1", "domain2"],
    "ip2": ["domain3"]
  },
{
  "Private A Records": {
    "ip3": ["domain4", "domain5"],
    "ip4": ["domain6"]
  },
  "CNAME Records": {
    "domain7": "alias1",
    "domain8": "alias2"
  }
}

```
Ways to parse this output:
- List all IPs in 'Public A Records':
```sh
jq '.["Public A Records"] | keys' massdns.json
```
- List all domains associated with a specific public IP in 'Public A Records':
```sh
jq '.["Public A Records"]["specific_public_ip"]' massdns.json
```
- List all domains in 'Public A Records':
```sh
jq '.["Public A Records"] | .[]' massdns.json
```
- List all domains and their corresponding public IPs in 'Public A Records':
```sh
jq '.["Public A Records"] | to_entries | .[] | "\(.key): \(.value)"' massdns.json
```
- List all IPs in 'Private A Records':
```sh
jq '.["Private A Records"] | keys' massdns.json
```
- List all domains associated with a specific private IP in 'Private A Records':
```sh
jq '.["Private A Records"]["specific_private_ip"]' massdns.json
```
- List all domains in 'Private A Records':
```sh
jq '.["Private A Records"] | .[]' massdns.json
```
- List all domains and their corresponding private IPs in 'Private A Records':
```sh
jq '.["Private A Records"] | to_entries | .[] | "\(.key): \(.value)"' massdns.json
```
- List all domains in 'CNAME Records':
```sh
jq '.["CNAME Records"] | keys' massdns.json
```
- Get the CNAME of a specific domain in 'CNAME Records':
```sh
jq '.["CNAME Records"]["specific_domain"]' massdns.json
```
- List all domains and their CNAMEs in 'CNAME Records':
```sh
jq '.["CNAME Records"] | to_entries | .[] | "\(.key) -> \(.value)"' massdns.json
```

<br/>

## headersRecon.py

This script will collect all response headers from the list of websites provided to it. Very useful for conducting a large-scale analysis of an organization. Among other things, it can be used to detect custom headers or software versions.

### Usage
```sh
python3 headersRecon.py
usage: headersRecon.py [-h] -l LIST [-o OUTPUT]
headersRecon.py: error: the following arguments are required: -l/--list

```
#### Usage Example

```
python3 headersRecon.py -l webs.txt -o headers.json
```

Input File example:
```text
https://example1.com/
http://example2.com/
https://example3.com/
http://example4.com/
https://example5.com/
https://example6.com/
```

### Output
```json

{
    "http://example.com": {
        "date": "Tue, 01 Jan 2023 00:00:00 GMT",
        "content-type": "text/plain; charset=utf-8",
        "content-length": "1234",
        "x-request-id": "abcdef12-3456-7890-abcd-ef1234567890",
        "cache-control": "no-cache, must-revalidate",
        "accept-ch": "en-US",
        "critical-ch": "en-US",
        "vary": "User-Agent",
        "x-adblock-key": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "set-cookie": "session=abcdef12-3456-7890-abcd-ef1234567890; expires=Tue, 01 Jan 2023 00:00:00 GMT; path=/"
    },
    "https://example2.com": {
        "Date": "Tue, 01 Jan 2023 00:00:00 GMT",
        "Content-Type": "application/json; charset=utf-8",
        "Content-Length": "5678",
        "X-Request-Id": "ghijkl34-5678-9012-ghij-kl3456789012",
        "Cache-Control": "private, max-age=3600",
        "Accept-Ch": "es-ES",
        "Critical-Ch": "es-ES",
        "Vary": "Accept-Encoding",
        "X-Adblock-Key": "ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ"
    }
}
```
How to parse output:
- Get all websites
```sh
cat webs_headers.txt | jq 'keys[]'

"https://example1.com"
"https://example2.com"
"https://example3.com"
"https://example4.com"
"https://example5.com"
"https://example6.com"
"https://example7.com"
"https://example8.com"
"https://example9.com"
"https://example10.com"
"https://example11.com"
"https://example12.com"
"https://example13.com"
"https://example14.com"
```

- Get all headers
```sh
cat webs_headers.txt | jq '.[] | keys[]' | sort -u

"Accept-Ch"
"Accept-Ranges"
"Age"
"Alt-Svc"
"Bugly-Version"
"Cache-Control"
"Cache-Status"
"Connection"
"Content-Encoding"
"Content-Language"
"Content-Length"
"Content-Security-Policy"
"Content-Security-Policy-Report-Only"
"Content-Type"
"Critical-Ch"
"Cross-Origin-Opener-Policy"
"Date"
"ETag"
"Etag"
"Expires"
"Last-Modified"
"Origin-Trial"
```
- Get headers values
```sh
cat webs_headers.txt | jq '.[] | values[]'

"Fri, 15 Dec 2023 19:38:34 GMT"
"text/html; charset=utf-8"
"1037"
"95d6009b-d8bf-46d7-b2c2-2b29badfffec"
"no-store, max-age=0"
"close"
"text/html"
"chunked"
"keep-alive"
"Thu, 09 Nov 2023 18:01:23 GMT"
```

- Get specific header values
```sh
cat webs_headers.txt | jq '.[].Server' | sort -u

"AmazonS3"
"ESF"
"Netlify"
"Server"
"nginx"
"nginx/1.12.2"
"nginx/1.20.0"
"nginx/1.22.0"
"nginx/1.8.0"
"openresty"
"stgw"
```
## crtRecon.go

Script to perform a search for domains using certificates (crt.sh) either vertically or horizontally. It allows enumeration through a domain or organization name.

### Installation

```sh
wget https://raw.githubusercontent.com/draco-0x6ba/ethical-hacking/main/recon/crtRecon.go
go install ./crtRecon.go
```

### Usage

```
Usage of the program:
  -d string
        Domain name for the HTTP request
  -o string
        Output file to save the results (optional)
  -org string
        Organization name for the HTTP request
```

#### Usage Example

Enumeration using a domain:
```sh
crtRecon -d "hackerone.com"

mta-sts.forwarding.hackerone.com
support.hackerone.com
*.hackerone.com
go.hackerone.com
www.hackerone.com
mta-sts.managed.hackerone.com
events.hackerone.com
info.hackerone.com
api.hackerone.com
mta-sts.hackerone.com
docs.hackerone.com
gslink.hackerone.com
links.hackerone.com
hackerone.com
design.hackerone.com
```
Enumeration using an organization name:
```sh
crtRecon -d "HackerOne Inc."

phabricator.inverselink.com
hacker.one
proteus.inverselink.com
withinsecurity.com
ci.inverselink.com
hackerone-ext-content.com
bd1.inverselink.com
info.hacker.one
events.hackerone.com
go.inverselink.com
*.testserver.inverselink.com
ma.hacker.one
bd3.inverselink.com
www.enorekcah.com
attjira.inverselink.com
go.hacker.one
payments-production.inverselink.com
enorekcah.com
...
```

### Tip

You can quickly obtain the organization names of domains using [tlsx](https://github.com/projectdiscovery/tlsx) tool from [ProjectDiscovery](https://github.com/projectdiscovery). Example:

```sh
crtRecon -d "hackerone.com" | tlsx -so -silent

mta-sts.managed.hackerone.com:443
mta-sts.forwarding.hackerone.com:443
mta-sts.hackerone.com:443
gslink.hackerone.com:443
docs.hackerone.com:443 [HackerOne Inc]
api.hackerone.com:443 [HackerOne Inc.]
hackerone.com:443 [HackerOne Inc.]
www.hackerone.com:443 [HackerOne Inc.]
support.hackerone.com:443
```

It's possible to discover new organization names associated with domains you already have, which you can then use for further searches.

## cnMap.py

Resolve CNAME records recursively (default depth 5) from a list of domains.

### Usage

```sh
usage: cnMap.py [-h] [-l FILE_PATH] [-o OUTPUT] [--csv CSV] [--resolvers RESOLVERS] [-d DEPTH]

Resolve CNAME records recursively from a list of domains.

options:
  -h, --help            show this help message and exit
  -l FILE_PATH, --list FILE_PATH
                        Path to the file containing domains
  -o OUTPUT, --output OUTPUT
                        Path to the output file (optional)
  --csv CSV             Path to the CSV output file (optional)
  --resolvers RESOLVERS
                        Path to the file containing DNS resolvers (optional)
  -d DEPTH, --depth DEPTH
                        Depth of recursive resolution (default: 5)
```

#### Usage Example

```sh
python3 cnMap.py -l domains.txt --resolvers resolvers-trusted.txt -o domains_cname.json --csv domains_cname.csv
```

You can obtain a list of trusted resolvers (resolvers-trusted.txt) at the following link:
- [resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

```sh
python3 cnMap.py -l <(crtRecon -d hackerone.com) --resolvers resolvers-trusted.txt -o domains_cname.json --csv domains_cname.csv

{
    "support.hackerone.com": [
        "2fe254e58a0ea8096400b2fda121ee35.freshdesk.com.",
        "fwfd-use1-lb208.freshdesk.com." <-- CNAME Record from 2fe254e58a0ea8096400b2fda121ee35.freshdesk.com
    ],
    "gslink.hackerone.com": [
        "d3rxkn2g2bbsjp.cloudfront.net."
    ],
    "mta-sts.hackerone.com": [
        "hacker0x01.github.io."
    ],
    "mta-sts.managed.hackerone.com": [
        "hacker0x01.github.io."
    ],
    "mta-sts.forwarding.hackerone.com": [
        "hacker0x01.github.io."
    ]
}
```
