#!/usr/bin/env python3
import subprocess, argparse, random, sys, os, time, itertools, pathlib, json, re, string
from datetime import datetime, timedelta
from urllib.parse import urlparse, urlunparse

banner = r"""
 __                       __      __                             ___
/\ \                     /\ \    /\ \__                        /'___\
\ \ \___      __      ___\ \ \___\ \ ,_\    __       __       /\ \__/ __  __  ____   ____
 \ \  _ `\  /'__`\   /',__\ \  _ `\ \ \/  /'__`\   /'_ `\_____\ \ ,__/\ \/\ \/\_ ,`\/\_ ,`\
  \ \ \ \ \/\ \L\.\_/\__,/`\ \ \ \ \ \ \_/\ \L\.\_/\ \L\ \_____\ \ \_\ \ \_\ \/_/  /\/_/  /_
   \ \_\ \_\ \__/.\_\/\____/\ \_\ \_\ \__\ \__/.\_\ \____ \____/\ \_\ \ \____/ /\_/__\/_/___\
    \/_/\/_/\/__/\/_/\/___/  \/_/\/_/\/__/\/__/\/_/\/___L\ \     \/_/  \/___/  \/____/\/____/
                                                     /\____/
                                                     \_/__/
                                                            The nightmare of WAFs & CDNs
                                                    https://github.com/Hashtag-AMIN/hashtag-fuzz
"""

USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Edge/91.0.864.48",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:91.0) Gecko/20100101 Firefox/91.0",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Opera/77.0.4054.277",
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/37.0.2062.94 Chrome/37.0.2062.94",
    "Mozilla/5.0 (iPhone; CPU iPhone OS 8_4_1 like Mac OS X) AppleWebKit/600.1.4 (KHTML, like Gecko) GSA/7.0.55539 Mobile/12H321 Safari",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729;",
    "Mozilla/5.0 (Linux; Android 5.1.1; Nexus 7 Build/LMY47V) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.84 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/600.8.9 (KHTML, like Gecko) Version/8.0.8 Safari/600.8.9",
    "Mozilla/5.0 (iPad; CPU OS 8_4_1 like Mac OS X) AppleWebKit/600.1.4 (KHTML, like Gecko) Version/8.0 Mobile/12H321 Safari/600.1.4",
    "Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:40.0) Gecko/20100101 Firefox/40.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.85 Safari/537.36",
    "Mozilla/5.0 (Android; Tablet; rv:40.0) Gecko/40.0 Firefox/40.0",
    "Mozilla/5.0 (iPhone; CPU iPhone OS 8_2 like Mac OS X) AppleWebKit/600.1.4 (KHTML, like Gecko) Version/8.0 Mobile/12D508",
    "Mozilla/5.0 (MSIE 9.0; Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko",
    "Mozilla/5.0 (Linux; Android 4.4.2; SM-T320 Build/KOT49H) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.84",
    "Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko",
    "Mozilla/5.0 (X11; CrOS x86_64 7077.95.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.90 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/600.7.12 (KHTML, like Gecko) Version/8.0.7 Safari/600.7.12",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:40.0) Gecko/20100101 Firefox/40.0",
    "Mozilla/5.0 (Linux; Android 5.0.2; SM-T530NU Build/LRX22G) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.84",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:40.0) Gecko/20100101 Firefox/40.0",
    "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)",
    "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.85 Safari/537.36",
    "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36",
    "Mozilla/5.0 (X11; CrOS x86_64 7077.134.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.156 Safari/537.36",
    "Mozilla/5.0 (Windows NT 6.2; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0",
    "Mozilla/5.0 (Linux; U; Android 4.4.3; en-us; KFASWI Build/KTU84M) AppleWebKit/537.36 (KHTML, like Gecko) Silk/3.68",
    "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; MDDCJS; rv:11.0) like Gecko",
    "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/7.0)"
]

url_regex = re.compile(
            r'^(http|https|socks4|socks5|FUZZ)://'             # scheme
            r'((([A-Za-z0-9$._%+-]+):([A-Za-z0-9$._%+-]+)@)?'  # optional username:password@
            r'((\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})|'          # IP address
            r'([A-Za-z0-9.-]+\.[A-Za-z]{2,6}))'                # hostname
            r'(:([0-9]{1,5}|FUZZ))?)'                          # optional port or FUZZ
            r'(/.*)?$'                                         # optional path
        )

letters = string.ascii_letters + string.digits

whitespace_char = [
    
        "%00",
        "%08",
        "%09",
        "%0a",
        "%0b",
        "%0C",
        "%0d",
        "%20",
        "%7F",
        "%81",
        "%8F",
        "%9D",
        "%A0",
        "%AD",
        "%0d%0a",
        "%C2%A0",
        "%C2%AD"
    ]

top_proxy_headers = [
    
        "X-Forwarded-For",
        "X-Forwarded-Host",
        "X-Remote-IP",
        "X-Originating-IP",
        "X-Remote-Addr",
        "X-Client-IP",
        "X-Host",
    ]

class SingleMetavarFormatter(argparse.HelpFormatter):

    def _format_action_invocation(self, action):

        if not action.option_strings:
            metavar, = self._metavar_formatter(action, action.dest)(1)
            return metavar.upper()
        else:
            parts = []
            parts.extend(action.option_strings)
            if action.nargs != 0:
                metavar = self._metavar_formatter(action, action.dest)(1)
            return ' '.join(parts)

def make_ffuf_url(url, wordlist, headers, threads, additional_cmd, delay, cookies, method, data, proxy, output_format, output_final, verbose, sleep, redirect):
    
    print(f"""
  [+] Start FUZZing on {url} url ...
________________________________________________
""")
    command = [
        'ffuf',
        '-u', f"{url}",
        '-t', str(threads),
        '-mc', 'all',
        '-fc', '404',
        '-of', 'json',
        '-c',
        '-noninteractive'
    ]
    command, tmp_wordlist, tmp_output = add_headers(command, headers, additional_cmd, delay, cookies, method, data, proxy, wordlist, redirect)
    exec_ffuf(command, tmp_wordlist, tmp_output, output_format, output_final, verbose, sleep)
    
def make_ffuf_raw(raw_req, wordlist, headers, threads, additional_cmd, delay, cookies, method, data, proxy, output_format, output_final, verbose, sleep, redirect):

    print(f"""
  [+] Start FUZZing on {raw_req} file ...
 ________________________________________________
 """)
    command = [
        'ffuf',
        '-request', f"{raw_req}",
        '-t', str(threads),
        '-mc', 'all',
        '-fc', '404',
        '-of', 'json',
        '-c',
        '-noninteractive'
    ]
    command, tmp_wordlist, tmp_output = add_headers(command, headers, additional_cmd, delay, cookies, method, data, proxy, wordlist, redirect)
    exec_ffuf(command, tmp_wordlist, tmp_output, output_format, output_final, verbose, sleep)
    
def exec_ffuf(command, tmp_wordlist, tmp_output, output_format, output_final, verbose, sleep):

    try:
        if verbose:
            print("___________________________")
            print("FFUF command run in behind:", "\n")
            str_command = ' '.join(command)
            print(str_command)
            print("___________________________")

        subprocess.run(command, check=True)
        os.remove(tmp_wordlist)
        if os.path.exists(tmp_output):
            write_final_output(tmp_output, output_format, output_final)

    except KeyError:
        print("Need Keyword FUZZ")
        os.remove(tmp_output)

    except KeyboardInterrupt:
        print("KeyboardInterrupt Error..!")
        print("Stop Script Manually!!")
    
    if sleep:
        time.sleep(sleep)
    else:
        time.sleep(1)
        
def add_headers(command, headers, additional_cmd, delay, cookies, method, data, proxy, wordlist, redirect):

    if headers:
        for header in headers:
            command.extend(['-H',f'{header}'])

    if method:
        command.extend(['-X',f'{method}'])

    if data:
        command.extend(['-d',f'{data}'])

    if cookies:
        command.extend(['-b',f'{cookies}'])

    if delay:
        command.extend(['-p',f'{delay}'])

    if redirect:
        command.extend(['-r'])
    
    if proxy:
        command.extend(['-x',f'{proxy}'])

    if additional_cmd:
        command.extend(additional_cmd.split())

    tmp_wordlist = make_wordlist_file(wordlist)
    command.extend(['-w', f"{tmp_wordlist}"])
    
    tmp_output = f"tmp-output-{time.time_ns()}.json"
    command.extend(['-o', f"{tmp_output}"])
    
    return command , tmp_wordlist, tmp_output

def generate_random_ip():
    range_type = random.choice(["127", "10", "172", "192"])

    if range_type == "127":
        return f"127.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
    elif range_type == "10":
        return f"10.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
    elif range_type == "172":
        return f"172.{random.randint(16, 31)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
    elif range_type == "192":
        return f"192.168.{random.randint(0, 255)}.{random.randint(0, 255)}"

def get_waf_headers(mode):

    random_IP = generate_random_ip()
    headers = {
        'entry': [
            f"X-Forwarded-For: {random_IP}",
            f"X-Forwarded-Host: {random_IP}",
            f"X-Remote-IP: {random_IP}",
            f"X-Originating-IP: {random_IP}",
            f"X-Remote-Addr: {random_IP}",
            f"X-Client-IP: {random_IP}",
            f"X-Host: {random_IP}",
            f"True-Client-IP: {random_IP}",
            f"Client-IP: {random_IP}",
        ],
        'common': [
            f"X-Forwarded-For: {random_IP}",
            f"X-Forwarded-Host: {random_IP}",
            f"X-Real-IP: {random_IP}",
            f"X-Remote-Addr: {random_IP}",
            f"X-Forwarded-By: {random_IP}",
            f"X-Remote-IP: {random_IP}",
            f"X-Originating-IP: {random_IP}",
            f"X-Client-IP: {random_IP}",
            f"X-Host: {random_IP}",
            f"True-Client-IP: {random_IP}",
            f"Client-IP: {random_IP}",
        ],
        'pro': [
            f"X-Forwarded-For: {random_IP}",
            f"X-Real-IP: {random_IP}",
            f"X-Remote-IP: {random_IP}",
            f"X-Client-IP: {random_IP}",
            f"X-Forwarded-Host: {random_IP}",
            f"X-Remote-Addr: {random_IP}",
            f"X-Forwarded-By: {random_IP}",
            f"X-Original-For: {random_IP}",
            f"X-Originating-IP: {random_IP}",
            f"X-Forwarded: {random_IP}",
            f"X-Host: {random_IP}",
            f"True-Client-IP: {random_IP}",
            f"Client-IP: {random_IP}",
        ],
        'prime': [
            f"X-Forwarded-For: {random_IP}",
            f"X-Remote-Addr: {random_IP}",
            f"X-Remote-IP: {random_IP}",
            f"X-Forwarded-Host: {random_IP}",
            f"X-Forwarded-Server: {random_IP}",
            f"X-Forwarded-By: {random_IP}",
            f"X-Client-IP: {random_IP}",
            f"X-Real-IP: {random_IP}",
            f"X-Forward-For: {random_IP}",
            f"X-Original-For: {random_IP}",
            f"X-Forwarded: {random_IP}",
            f"X-Originating-IP: {random_IP}",
            f"X-Host: {random_IP}",
            f"True-Client-IP: {random_IP}",
            f"Client-IP: {random_IP}",
            f"Forwarded: for={random_IP}; proto=http; by={random_IP}"
        ]
    }
    return headers.get(mode, [])

def get_threads_waf(mode):
    threads = {
        'entry': 20,
        'common': 10,
        'pro': 5,
        'prime': 1
    }
    return threads.get(mode, 30)

def get_delay_waf(mode):
    delays = {
        'entry': '0.1-0.3',
        'common': '0.2-0.5',
        'pro': '0.5-1',  
        'prime': '1-2'
    }
    return delays.get(mode, None)

def get_chunk_size(mode):
    chunk_size = {
        'entry': 250,
        'common': 200,
        'pro': 100,
        'prime': 50
    }
    return chunk_size.get(mode, 300)

def split_wordlist(wordlist, chunk_size):
    with open(wordlist, 'r', encoding='utf-8', errors='ignore') as file:
        lines = file.readlines()
    return [lines[i:i + chunk_size] for i in range(0, len(lines), chunk_size)]

def make_wordlist_file(wordlist_chunked):
    file_name = f"./tmp-wordlist-{time.time_ns()}.txt"
    with open(file_name, 'w', encoding='utf-8', errors='ignore') as file:
        file.writelines(wordlist_chunked)
    return file_name

def write_final_output(tmp_output, format_output, output_final):
    with open(tmp_output, "r", encoding='utf-8', errors='ignore') as file:
        data = json.load(file)

    sorted_results = sorted(data['results'], key=lambda x: x['status'])

    if format_output.lower() == "txt":
        txt_output = "\n".join(f"[{result['status']}] {result['url']} [{result['length']}] [{result['input']['FUZZ']}]" for result in sorted_results)

        if txt_output:
            with open(output_final, 'a', encoding='utf-8', errors='ignore') as file:
                file.write(txt_output  + "\n")

    elif format_output.lower() == "csv":
        csv_output = "\n".join(f"{result['status']},{result['url']},{result['length']},{result['input']['FUZZ']}" for result in sorted_results)

        if csv_output:
            with open(output_final, 'a', encoding='utf-8', errors='ignore') as file:
                file.write(csv_output + "\n")

    elif format_output.lower() == "json":
        json_output = [{'status': result['status'], 'url': result['url'], 'length': result['length'], 'FUZZ': result['input']['FUZZ']} for result in sorted_results]

        try:
            with open(output_final, 'r', encoding='utf-8', errors='ignore') as file:
                existing_data = json.load(file)
        except:
                existing_data = []
        existing_data.extend(json_output)

        if json_output:
            with open(output_final, 'w', encoding='utf-8', errors='ignore') as file:
                file.write(json.dumps(existing_data))
    
    os.remove(tmp_output)

def sort_by_status(output_file):
    with open(output_file, 'r', encoding='utf-8', errors='ignore') as file:
        lines = file.readlines()

    lines_with_status = [(line.split()[0], line) for line in lines]
    sorted_lines = sorted(lines_with_status, key=lambda x: x[0])
    sorted_lines = [line[1] for line in sorted_lines]

    with open(output_file, 'w', encoding='utf-8', errors='ignore') as file:
        file.writelines(sorted_lines)

def restart_tor():
    try:
        if 'linux' in sys.platform:
            subprocess.run(['systemctl', 'restart', 'tor'], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, check=True)

        elif 'win32'in sys.platform:
            subprocess.run(['sc', 'stop', 'tor'], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, check=True)
            time.sleep(0.5)
            subprocess.run(['sc', 'start', 'tor'], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, check=True)

        elif 'darwin' in sys.platform:
            subprocess.run(['brew', 'services', 'restart', 'tor'], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, check=True)

        else:
            print("-tor/--tor Option not avalible in your OS")

        time.sleep(0.5)
        
    except subprocess.CalledProcessError as e:
        print(f"Error restarting Tor: {e}")

def validate_proxies(proxies):
    for proxy in proxies:
        if not url_regex.match(proxy):
            print(f"Invalid proxy URL: {proxy}")
            sys.exit(0)

def validate_urls(urls):
    for url in urls:
        if not url_regex.match(url):
            print(f"Invalid URL: {url}")
            sys.exit(0)

def add_browser_header(url):
    
    parsed_url = urlparse(url)
    browser_headers = [
        
        f"User-Agent: {random.choice(USER_AGENTS)}",
        f'Origin: {parsed_url.scheme}://{parsed_url.netloc}',
        f'Referer: {parsed_url.scheme}://{parsed_url.netloc}/',
        f"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        f"Accept-Encoding: gzip, deflate, br",
        f"Accept-Language: en-US,en;q=0.9",
        f"Upgrade-Insecure-Requests: 1",
        f"Sec-Fetch-Dest: empty",
        f"Sec-Fetch-Mode: cors",
        f"Sec-Fetch-Site: same-origin",
        f"Sec-Fetch-User: ?1",
        f'Sec-Ch-Ua-Platform: "Windows"',
        f"Sec-Ch-Ua-Mobile: ?0",
        f'Sec-Ch-Ua: "Chromium";v="135", "Not-A.Brand";v="8"'
        f"Cache-Control: max-age=0",
        f"If-Modified-Since: {(datetime.now() - timedelta(days=7)).strftime('%a, %d %b %Y %H:%M:%S GMT')}",
        f"If-None-Match: *",
        f"X-Requested-With: XMLHttpRequest",
        f"Priority: u=1",
        f"Te: trailers",
        f"DNT: 1",
        
    ]
    return browser_headers

def generate_random_string(length):
        return ''.join(random.choice(letters) for _ in range(length))
    
def add_random_query_str(url):
    
    if "?" in url:
        if url[-1] in ["?",'&']:
            url = url + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7))
            
        elif url[-1] == "=" or url[-1] in string.printable:
            url = url + "&" + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7))
        
    else:
        if url.count("/") >= 3:
            url = url + "?" + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7))
            
        elif url.count("/") == 2:
            url = url + "/?" + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7))
            
    return url

def add_whitespace_char(url):
    
    if "?" in url:
        if url[-1] in ["?",'&']:
            url = url + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7)) + random.choice(whitespace_char)
            
        elif url[-1] == "=" or url[-1] in string.printable:
            url = url + "&" + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7)) + random.choice(whitespace_char)
        
    else:
        if url.count("/") >= 3:
            url = url + "?" + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7)) + random.choice(whitespace_char)
            
        elif url.count("/") == 2:
            url = url + "/?" + generate_random_string(random.randint(4,7)) + "=" + generate_random_string(random.randint(4,7)) + random.choice(whitespace_char)
            
    return url

def extract_url_from_raw(raw_request):
    with open(raw_request, 'r', encoding='utf-8', errors='ignore') as file:
        lines = [line.strip() for line in file.readlines()]
        request_line = lines[0]
        path = request_line.split()[1]
    
        host = None
        for line in lines:
            if line.lower().startswith("host:"):
                host = line.split(":", 1)[1].strip()
                break
        if not host:
            raise ValueError("Host header not found.")
        
        url = f"https://{host}{path}"
    return url

def update_raw_path(url, raw_request):

    parsed_url = urlparse(url)
    path = parsed_url.path
    if parsed_url.query:
        path = f"{path}?{parsed_url.query}"

    with open(raw_request, 'r+', encoding='utf-8', errors='ignore') as file:
        request_lines = [line.strip() for line in file.readlines()]
        if not request_lines:
            return None
        first_line = request_lines[0]
        method, _, version = first_line.split()
        new_first_line = f"{method} {path} {version}"
        request_lines[0] = new_first_line

        for i, line in enumerate(request_lines[1:]):
            if line.lower().startswith('host:'):
                request_lines[i + 1] = f"Host: {parsed_url.netloc}"
                break
        updated_request = "\n".join(request_lines).strip() + "\n\n"
        
        file.seek(0)
        file.truncate()
        file.write(updated_request)

    return raw_request

def uppercase_random_char(url):
    parsed_url = urlparse(url)
    host = parsed_url.netloc
    random_index = random.randint(0, len(host) - 1)
    host = host[:random_index] + host[random_index].upper() + host[random_index + 1:]
    modified_url = parsed_url._replace(netloc=host)
    
    return urlunparse(modified_url)

def add_tamper_header(wafHeaderSet):
    if wafHeaderSet:
        tampe_header = [f"{random.choice(top_proxy_headers)}:  "]
    else:
        header = random.choice(top_proxy_headers)
        tampe_header = [
            f"{header}: {generate_random_ip()}",
            f"{header}:  "
        ]
    return tampe_header

def main():

    parser = argparse.ArgumentParser(formatter_class=SingleMetavarFormatter)
    parser.add_argument('-u', '--url', help='Target URL', type=str)
    parser.add_argument('-U', '--urls', help='File include List of Target URLs', type=pathlib.Path)
    parser.add_argument('-request','--req-raw', help='File containing the raw http request [-request-proto is default: https, change with -a command: -a="-request-proto http"]', type=pathlib.Path)
    parser.add_argument('-H', '--header', action='append', help='Custom headers to add to requests,  Multiple -H flags are accepted.', type=str)
    parser.add_argument('-b', '--cookie', help='Cookie data `"NAME1=VALUE1; NAME2=VALUE2"` for copy as curl[ffuf] functionality.', type=str)
    parser.add_argument('-d', '--data', help='Body to send with the request', type=str)
    parser.add_argument('-X', '--method', help='HTTP method to use (e.g.,GET,POST,PUT)', type=str)
    parser.add_argument('-r', '--redirect', help='Follow redirects (default: false)', action='store_true')
    parser.add_argument('-w', '--wordlist', required=True, action='append', help='Path to wordlist,  Multiple -w/--wordlist flags are accepted and fuzz with all wordlist.', type=str)
    parser.add_argument('-t', '--thread', help='Threads use in ffuf (default: [select with waf mode])', type=int)
    parser.add_argument('-p', '--delay', help='Random delay range use in ffuf (default: [select with waf mode]) For example "0.5-2.0" or "0.3"')
    parser.add_argument('-waf', '--waf-mode', choices=['entry', 'common', 'pro', 'prime'], help='WAF behavior mode: entry, common, pro, prime')
    parser.add_argument('-cs', '--chunk-size', help='Split each wordlist with Chunk size, (default: 300)', type=int)
    parser.add_argument('-rq', '--random-query', help='Append a random query string to url', action='store_true')
    parser.add_argument('-rs', '--random-space', help='Append a random query with whitespace characters end of url', action='store_true')
    parser.add_argument('-ct', '--case-tamper', help='Uppercase random word of host in each junk', action='store_true')
    parser.add_argument('-ht', '--header-tamper', help='Add & repeat proxy/CDN headers with Null value', action='store_true')
    parser.add_argument('-sl', '--sleep', help='Sleep beetween fuzzing each junk of wordlist', type=int)
    parser.add_argument('-x', '--proxy', nargs='?', const='http://127.0.0.1:8080', action='append', help='Proxy server(s) to use (default: http://127.0.0.1:8080 for Burp) Multiple -x/--proxy flags are accepted')
    parser.add_argument('-xf', '--proxy-file', help='Proxy servers file to use', type=pathlib.Path)
    parser.add_argument('-tor', '--tor', const='socks5://127.0.0.1:9050', nargs='?', help='Use Tor with unique(dynamic) IP for each chunk of wordlist (default: socks5://127.0.0.1:9050)')
    parser.add_argument('-o', '--output', help='Output file name of FFUF result', default='output', type=str)
    parser.add_argument('-of', '--output-format', help='Output of FFUF brief and useful mode: txt, csv, json (default: txt)', default='txt', type=str, choices=['txt','csv','json'])
    parser.add_argument('-a', '--additional-cmd', help='Additional commands for FFUF', type=str)
    parser.add_argument('-v', '--verbose', help='verbose mode (default: false)', action='store_true')

    args = parser.parse_args()

    if len([arg for arg in sys.argv if arg in ['-u', '-U', '-request', '--url', '--urls', '--req-raw']]) > 1:
        parser.error("[-] Argument -u/--url|-U/--urls|-request/--req-raw: not allowed with more than one use !!")
    
    if sum(bool(arg) for arg in [args.url, args.urls, args.req_raw, not sys.stdin.isatty()]) != 1:
        parser.error("[-] You must provide exactly one of -u/--url, -U/--urls, -request/--req-raw, or input from stdin !!")

    for wordlist in args.wordlist:
        if not os.path.exists(wordlist):
            parser.error(f"[-] The file {wordlist} value of -w/--wordlist flag does not exist !!")
    
    if args.req_raw:
        if not os.path.exists(args.req_raw):
            parser.error(f"[-] The file {args.req_raw} value of -request/--req-raw flag does not exist !!")
    
    if sum(bool(arg) for arg in [args.proxy, args.proxy_file, args.tor]) > 1:
        parser.error("[-] You must provide exactly one of --proxy/--proxy-file/--tor !!")

    if len([arg for arg in sys.argv if arg in ['-tor', '--tor', '-xf', '--proxy-file']]) > 1:
        parser.error("[-] Argument -tor/--tor|-xf/--proxy-file: not allowed with more than one use !!")
    
    proxy = args.proxy
    proxy_file = args.proxy_file
    tor = args.tor
    output_format = args.output_format
    output = args.output
    verbose = args.verbose

    if args.thread:
        threads = args.thread
    else:
        threads = get_threads_waf(args.waf_mode)
    
    if args.delay:
        delay = args.delay
    else:
        delay = get_delay_waf(args.waf_mode)

    if args.chunk_size:
        chunnk_size = args.chunk_size
    else:
        chunnk_size = get_chunk_size(args.waf_mode)

    output_final = f"{output}__{datetime.now().strftime('%Y-%m-%d-%H-%M-%S-%f')}.{output_format.lower()}"

    if proxy_file:
        with open(proxy_file, "r", encoding='utf-8', errors='ignore') as file:
            lines = file.readlines()
            proxies = [line.strip() for line in lines]
        validate_proxies(proxies)
    elif proxy:
        validate_proxies(proxy)
        proxies = proxy
    else:
        proxies = [None]

    proxy_cycle = itertools.cycle(proxies)
    def get_next_proxy():
        return next(proxy_cycle)

    try:
        if args.url:
            url = args.url
            validate_urls([url])
            for wordlist in args.wordlist:
                wordlist_chunks = split_wordlist(wordlist, chunnk_size)
                for wordlist_chunk in wordlist_chunks:
                    headers = args.header.copy() if args.header else []
                    headers.extend(add_browser_header(url))
                    if args.waf_mode:
                        headers.extend(get_waf_headers(args.waf_mode))
                    if args.header_tamper:
                        headers.extend(add_tamper_header(args.waf_mode))
                    if tor:
                        restart_tor()
                        proxy = tor
                    else:
                        proxy = get_next_proxy()
                    if args.random_query and args.random_space:
                        url = add_random_query_str(url)
                        url = add_whitespace_char(url)
                    elif args.random_space:
                        url = add_whitespace_char(url)
                    elif args.random_query:
                        url = add_random_query_str(url)
                    else:
                        url = args.url
                    if args.case_tamper:
                        url = uppercase_random_char(url)
                    
                    make_ffuf_url(url, wordlist_chunk, headers, threads, args.additional_cmd, delay, args.cookie, args.method, args.data, proxy, output_format, output_final, verbose, args.sleep, args.redirect)
                    headers = []
                    url = args.url
                    
            if os.path.exists(output_final):
                sort_by_status(output_final)

        elif args.urls:
            with open(args.urls, 'r', encoding='utf-8', errors='ignore') as f:
                urls = f.read().splitlines()
                validate_urls(urls)
                for url in urls:
                    main_url = url
                    for wordlist in args.wordlist:
                        wordlist_chunks = split_wordlist(wordlist, chunnk_size)
                        for wordlist_chunk in wordlist_chunks:
                            headers = args.header.copy() if args.header else []
                            headers.extend(add_browser_header(url))
                            if args.waf_mode:
                                headers.extend(get_waf_headers(args.waf_mode))
                            if args.header_tamper:
                                headers.extend(add_tamper_header(args.waf_mode))
                            if tor:
                                restart_tor()
                                proxy = tor
                            else:
                                proxy = get_next_proxy()
                            if args.random_query and args.random_space:
                                url = add_random_query_str(url)
                                url = add_whitespace_char(url)
                            elif args.random_space:
                                url = add_whitespace_char(url)
                            elif args.random_query:
                                url = add_random_query_str(url)
                            if args.case_tamper:
                                url = uppercase_random_char(url)
                            
                            make_ffuf_url(url, wordlist_chunk, headers, threads, args.additional_cmd, delay, args.cookie, args.method, args.data, proxy, output_format, output_final, verbose, args.sleep, args.redirect)
                            headers = []
                            url = main_url
            
            if os.path.exists(output_final):
                sort_by_status(output_final)

        elif args.req_raw:
            main_url = extract_url_from_raw(args.req_raw)
            request_raw = args.req_raw
            for wordlist in args.wordlist:
                wordlist_chunks = split_wordlist(wordlist, chunnk_size)
                for wordlist_chunk in wordlist_chunks:
                    headers = args.header.copy() if args.header else []
                    url = extract_url_from_raw(request_raw)
                    headers.extend(add_browser_header(url))
                    if args.waf_mode:
                        headers.extend(get_waf_headers(args.waf_mode))
                    if args.header_tamper:
                        headers.extend(add_tamper_header(args.waf_mode))
                    if tor:
                        restart_tor()
                        proxy = tor
                    else:
                        proxy = get_next_proxy()
                    if args.random_query and args.random_space:
                            url = add_random_query_str(url)
                            url = add_whitespace_char(url)
                    elif args.random_space:
                            url = add_whitespace_char(url)
                    elif args.random_query:
                            url = add_random_query_str(url)
                    if args.case_tamper:
                            url = uppercase_random_char(url)  
                    
                    update_raw_path(url, request_raw)
                    
                    make_ffuf_raw(request_raw, wordlist_chunk, headers, threads, args.additional_cmd, delay, args.cookie, args.method, args.data, proxy, output_format, output_final, verbose, args.sleep, args.redirect)
                    headers = []
                    update_raw_path(main_url, request_raw)
                    
            if os.path.exists(output_final):
                sort_by_status(output_final)

        else:
            urls = [line.strip() for line in sys.stdin]
            validate_urls(urls)
            for url in urls:
                main_url = url
                for wordlist in args.wordlist:
                    wordlist_chunks = split_wordlist(wordlist, chunnk_size)
                    for wordlist_chunk in wordlist_chunks:
                        headers = args.header.copy() if args.header else []
                        headers.extend(add_browser_header(url))
                        if args.waf_mode:
                            headers.extend(get_waf_headers(args.waf_mode))
                        if args.header_tamper:
                            headers.extend(add_tamper_header(args.waf_mode))
                        if tor:
                            restart_tor()
                            proxy = tor
                        else:
                            proxy = get_next_proxy()
                        if args.random_query and args.random_space:
                            url = add_random_query_str(url)
                            url = add_whitespace_char(url)
                        elif args.random_space:
                            url = add_whitespace_char(url)
                        elif args.random_query:
                            url = add_random_query_str(url)
                        if args.case_tamper:
                            url = uppercase_random_char(url)
                        
                        make_ffuf_url(url, wordlist_chunk, headers, threads, args.additional_cmd, delay, args.cookie, args.method, args.data, proxy, output_format, output_final, verbose, args.sleep, args.redirect)
                        headers = []
                        url = main_url

        if os.path.exists(output_final):
            sort_by_status(output_final)
    
    except subprocess.CalledProcessError as e:
        print(f"An error occurred: {e}")

    except KeyError as e :
        print("Need Keyword FUZZ")

    except FileNotFoundError:
        print("FFUF is not installed or not found in the system path.")

    except KeyboardInterrupt :
        print("KeyboardInterrupt Error..!")
        print("Stop Script Manually!!")
          
if __name__ == "__main__":

    print(banner)
    main()
    print()
