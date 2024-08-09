
# hashtag-fuzz
### &nbsp;&nbsp;hashtag-fuzz, The wrapper of ffuf ###

[![Python](https://img.shields.io/static/v1?label=&labelColor=lightblue&message=Python&color=blue&style=flat&logo=python&logoColor=black)]()
&nbsp;[![ffuf](https://img.shields.io/static/v1?label=&labelColor=lightblue&message=ffuf&color=blue&style=flat&logo=go&logoColor=black)]()&nbsp;[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)



## Overview

***hashtag-fuzz*** is a fuzzing tool designed to test and bypass WAFs and CDNs. By leveraging advanced features such as random User-Agent and header value, random delays, handle multi-threading, selective chunking of wordlists and Round Robin proxy rotation for each chunked, it offers a robust solution for security professionals aiming to identify vulnerabilities in web applications. This tool stands out as the almost first and best in its class for WAF testing about rate and limits.

## Installation

### First need to install amazing fuzzing tool: <a href="https://github.com/ffuf/ffuf#installation">ffuf</a>
allow app to global access in OS, if don't want global put it same folder with this script, then:

```bash
git clone https://github.com/Hashtag-AMIN/hashtag-fuzz.git
cd ./hashtag-fuzz
chmod +x ./hashtag-fuzz
./hashtag-fuzz
```
All library used is built-in python library, No need pip ;)

#### Global access:

```bash
cp ./hashtag-fuzz /usr/local/sbin/
```


## Features

- ***Control Threads for Fuzzing:*** The tool allows users to control the number of threads for fuzzing. By adjusting the thread count, you can manage the tool's performance and ensure that your fuzzing activities do not overwhelm the target server.
- ***Random Delay Control:*** Implementing random delays between requests is crucial for bypassing WAFs and mimic human behavior and avoid detection. The tool reads and applies random delays from the configuration, adding an extra layer of sophistication to your fuzzing activities.
- ***Random User-Agent and Headers:*** The tool supports randomizing User-Agent and other headers, which is essential for evading WAF detection. By sending requests with varying headers, the tool helps you bypass WAFs more effectively.
- ***Proxy and TOR Support:*** To further enhance the tool's stealth capabilities, you can route requests through proxies or Tor. This feature ensures that your requests appear to come from different IP addresses, making it harder for WAFs to detect and block your fuzzing activities.
- ***Selective Chunking of Wordlists:*** The tool allows you to split your wordlist into chunks, sending each chunk with random headers, User-Agent, and IPs. This feature is especially useful when dealing with large wordlists, as it optimizes the fuzzing process.
  - ***Round Robin Proxy Usage:*** Rotate proxies in a round-robin manner for each chunk.
  - ***TOR IP Cycling:*** Each chunk sent through TOR will use a different source IP, improving chances of bypassing WAF/CDN filters.

## WAF Modes

The tool supports four WAF Modes, each with specific configurations:

| WAF Modes   | Description |
|-------------|-------------|
| ***entry***   | Basic WAF profile with minimal protection mechanisms. Suitable for initial testing. |
| ***common***  | Standard WAF profile with common protection mechanisms. Suitable for general-purpose testing. |
| ***pro***     | Advanced WAF profile with more sophisticated protection mechanisms. Suitable for testing against professional-grade WAFs. |
| ***prime***   | Premium WAF profile with the highest level of protection. Suitable for testing against enterprise-grade WAFs. |

<hr>

## Now let see more details: 

Hashtag-Fuzz supports four WAF modes, each offering unique features and configurations:

| Mode   | Control Threads for Fuzzing | Control Random Delay | Random User-Agent & Headers | Cunck size of splitted wordliost|
|--------|-----------------------------|----------------------|-----------------------------|-----------------------------|
| ***entry*** | Basic thread control, limited 20 Threads | Minimal delay, random delays between 0.1-0.2s | Random User-Agent and Randomization 3 top headers for simulate internal network | splitted worlist to 250 chunks |
| ***common*** | Improved thread management, limited 10 Threads | Introduces random delays between 0.2-0.5s | Random User-Agent,  Randomization 6 top headers with random local/Private IP | splitted worlist to 200 chunks |
| ***pro*** | Advanced thread control, limited 5 Threads | random delays between requests 0.5-1s | Random User-Agent and Randomization efficient headers with local/Private range | splitted worlist to 100 chunks |
| ***prime*** | Maximum thread control, limited 1 Threads | Max delay, random delays between 1-2s | Most useful headers randomization and Random User-Agent | splitted worlist to 50 chunks |

#### Remember, All this value is selective with argument and you can add custom value ;)

## Usage 

```
└─# ./hashtag-fuzz -h

 __                       __      __                             ___
/\ \                     /\ \    /\ \__                        /'___\
\ \ \___      __      ___\ \ \___\ \ ,_\    __       __       /\ \__/ __  __  ____   ____
 \ \  _ `\  /'__`\   /',__\ \  _ `\ \ \/  /'__`\   /'_ `\_____\ \ ,__/\ \/\ \/\_ ,`\/\_ ,`\
  \ \ \ \ \/\ \L\.\_/\__,/`\ \ \ \ \ \ \_/\ \L\.\_/\ \L\ \_____\ \ \_\ \ \_\ \/_/  /\/_/  /_
   \ \_\ \_\ \__/.\_\/\____/\ \_\ \_\ \__\ \__/.\_\ \____ \____/\ \_\ \ \____/ /\_/__\/_/___\
    \/_/\/_/\/__/\/_/\/___/  \/_/\/_/\/__/\/__/\/_/\/___L\ \     \/_/  \/___/  \/____/\/____/
                                                     /\____/
                                                     \_/__/
                                                            The wrapper of ffuf
                                                        The nightmare of WAFs & CDNs
                                                https://github.com/Hashtag-AMIN/hashtag-fuzz

usage: hashtag-fuzz [-h] [-u URL] [-U URLS] [-request REQUEST_RAW] 
                    [-H HEADER] [-b COOKIE] [-d DATA] [-X METHOD] -w WORDLIST 
                    [-waf {entry,common,pro,prime}]
                    [-cs chunk_SIZE] [-t THREAD] [-p RANDOM_DELAY] 
                    [-x [PROXY]] [-xf PROXY_FILE] [-tor [TOR]] 
                    [-o OUTPUT] [-of {txt,csv}]
                    [-a ADDITIONAL_CMD] [-v]

hashtag-fuzz wrapper for FFUF.

options:
  -h --help             show this help message and exit
  -u --url URL          Target URL
  -U --urls URLS        File include List of Target URLs
  -request --request-raw REQUEST_RAW
                        File Path to raw request [-request-proto is default: https, 
                        if need to change with -a command: -a="-request-proto http"]
  -H --header HEADER    Custom headers to add to requests
  -b --cookie COOKIE    Cookies to add to requests
  -d --data DATA        Data to send with the request for POST/PUT/PATCH methods
  -X --method METHOD    HTTP method to use (e.g., GET, POST, PUT)
  -w --wordlist WORDLIST
                        Path to wordlist
  -waf --waf-mode       {ENTRY,COMMON,PRO,PRIME}
                        WAF or CDN behavior mode: entry, common, pro, or prime
  -cs --chunk-size chunk_SIZE
                        Split each wordlist with Chunk size, (default=300)
  -t --thread THREAD    threads use in ffuf (default=[select in waf mode])
  -p --random-delay RANDOM_DELAY
                        random delay range use in ffuf (default=[select in waf mode])
  -x --proxy PROXY      Proxy server|servers to use (c)
  -xf --proxy-file PROXY_FILE
                        Proxy servers file to use
  -tor --tor TOR        use Tor proxy with unique(dynamic) IP address for each chunk of wordlist 
                        (default=socks5://127.0.0.1:9050)
  -o --output OUTPUT    Output file name of FFUF result
  -of --output-format   {TXT,CSV}
                        Output of FFUF brief and useful mode: txt, csv
  -a --additional-cmd ADDITIONAL_CMD
                        Additional FFUF commands
  -v --verbose          verbose mode default: False

```

### Single URL Fuzzing

```bash
./hashtag-fuzz -u http://site.tld/FUZZ -w ./wordlist.txt -waf entry
./hashtag-fuzz -u http://site.tld/FUZZ -w ./wordlist.txt -waf pro -cs 10
```

### raw request Fuzzing

```bash
./hashtag-fuzz -request ./raw-req.txt -w ./wordlist.txt -waf common
```

### Multi-URL Fuzzing

```bash
./hashtag-fuzz -U ./urls.txt -w ./wordlist.txt -waf entry -cs 50 -H "X-header: header-val"
./hashtag-fuzz -U ./urls.txt -w ./wordlist.txt -waf pro -H 'Authorization: value_token'
```

### Pipe Input Fuzzing

```bash
echo 'http://site.tld' | ./hashtag-fuzz -w ./wordlist.txt -waf common -cs 15 -d "var1=FUZZ&var2=val2" -X "PUT"
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 60 "X-header: FUZZ" -p '2-2.5'
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf pro -b 'cookie=value_session'
```

#### Don't forget urls or values inculde FUZZ keyword when Use --urls or urls come from stdin

## Proxy Mode

### Proxy and Proxy file

default = http://127.0.0.1:8080 with use -x (for Burp), also multi value and proxy file with valid urls. 
```bash
echo 'http://site.tld/FUZZ' | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 15 -t 50 -x 
echo 'http://site.tld/FUZZ' | ./hashtag-fuzz -w ./wordlist.txt -waf entry -x 'http://proxy1.com'
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf common -cs 15 "header: header-val" -x 'http://proxy1.com' -x 'http://proxy2.com'
echo 'http://site.tld/FUZZ' | ./hashtag-fuzz -w ./wordlist.txt -waf pro -cs 100 -x ./proxy.txt
```

### TOR Proxy 

Every time each chunk sent to target, Tor service is restarted and new IP in taken, So need add Tor as service in OS

default = socks5://127.0.0.1:9050 with use -tor (for default local address tor)

```bash
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 15 "header: header-val" -tor
echo 'http://test.it/FUZZ' | ./hashtag-fuzz -w ./wordlist.txt -waf common -cs 80 -tor socks5://proxy1.tor:8000
```
<a href="https://docs.start9.com/0.3.5.x/device-guides/index">Help and Docs for Add Tor as service in any OS</a>

## additional commad

If you need add some command in ffuf, you can write it as string in -a/--additional-cmd argument

```bash
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf common -d "data-var=FUZZ" -X "PUT" -a="-mr '.*test'"
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 15 -tor -a='-ac'
```
#### Point: Use = {equal} insted {space} for avoid error in argparser library just for -a/--additional-cmd flag, becuase maybe value include special characters.

## Best usage

If you want fuzz but waf or cdn block in high rate in request, use these techniques together

```bash
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 20 -tor
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 30 -xf ./proxy.txt
```

default random delay in prime mode between 1-2, but sometime need to customize:

```bash
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf prime -cs 10 -tor -p '2-2.5'
```

By default handle threads, but Also can contorl it:

```bash
cat ./urls.txt | ./hashtag-fuzz -w ./wordlist.txt -waf pro -cs 5 -tor -p '2.5-3' -t 1
```

One useful follow for use this feature, use wirh <a href="https://github.com/devanshbatham/ParamSpider">ParamSpider</a> or <a href="https://github.com/0xKayala/ParamSpider">ParamSpider</a>

```bash
"urls_come_form_ParamSpider_with_FUZZ_keyword" | ./hashtag-fuzz -w ./wordlist.txt -waf pro -cs 50 -tor 
```
<hr>

### Happy Hunting, Happy learning ;)