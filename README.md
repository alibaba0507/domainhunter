# Domain Hunter

Authors Joe Vest (@joevest) & Andrew Chiles (@andrewchiles)

Domain name selection is an important aspect of preparation for penetration tests and especially Red Team engagements. Commonly, domains that were used previously for benign purposes and were properly categorized can be purchased for only a few dollars. Such domains can allow a team to bypass reputation based web filters and network egress restrictions for phishing and C2 related tasks. 

This Python based tool was written to quickly query the Expireddomains.net search engine for expired/available domains with a previous history of use. It then optionally queries for domain reputation against services like Symantec WebPulse (BlueCoat), IBM X-Force, and Cisco Talos. The primary tool output is a timestamped HTML table style report.

## Changes

- 13 August 2019
   + Added authentication support for ExpiredDomains.net thanks to acole76!
   
- 5 October 2018
   + Fixed logic for filtering domains with desirable categorizations. Previously, some error conditions weren't filtered and would result in domains without a valid categorization making it into the final list.

- 4 October 2018
   + Tweaked parsing logic
   + Fixed changes parsed columns indexes

- 17 September 2018
    + Fixed Symantec WebPulse Site Review parsing errors caused by service updates

- 18 May 2018
    + Add --alexa switch to control Alexa ranked site filtering

- 16 May 2018
    + Update queries to increase probability of quickly finding a domain available for instant purchase. Previously, many reported domains had an "In Auction" or "Make an Offer" status. New criteria: .com|.net|.org + Alexa Ranked + Available for Purchase
    + Improved logic to filter out uncategorized and some potentially undesirable domain categorizations in the final text table and HTML output
    + Removed unnecessary columns from HTML report

- 6 May 2018
    + Fixed expired domains parsing when performing a keyword search
    + Minor HTML and text table output updates
    + Filtered reputation checks to only execute for .COM, .ORG, and .NET domains and removed check for Archive.org records when performing a default or keyword search. Credit to @christruncer for the original PR and idea.

- 11 April 2018
    + Added OCR support for CAPTCHA solving with tesseract. Thanks to t94j0 for the idea in [AIRMASTER](https://github.com/t94j0/AIRMASTER)  
    + Added support for input file list of potential domains (-f/--filename)
    + Changed -q/--query switch to -k/--keyword to better match its purpose
    + Added additional error checking for ExpiredDomains.net parsing

- 9 April 2018
    + Added -t switch for timing control. -t <1-5>
    + Added Google SafeBrowsing and PhishTank reputation checks
    + Fixed bug in IBMXForce response parsing

- 7 April 2018
    + Fixed support for Symantec WebPulse Site Review (formerly Blue Coat WebFilter)
    + Added Cisco Talos Domain Reputation check
    + Added feature to perform a reputation check against a single non-expired domain. This is useful when monitoring reputation for domains used in ongoing campaigns and engagements.

- 6 June 2017
    + Added python 3 support
    + Code cleanup and bug fixes
    + Added Status column (Available, Make Offer, Price, Backorder, etc)

## Features

- Retrieve specified number of recently expired and deleted domains (.com, .net, .org) from ExpiredDomains.net
- Retrieve available domains based on keyword search from ExpiredDomains.net
- Perform reputation checks against the Symantec WebPulse Site Review (BlueCoat), IBM x-Force, Cisco Talos, Google SafeBrowsing, and PhishTank services
- Sort results by domain age (if known) and filter for reputation
- Text-based table and HTML report output with links to reputation sources and Archive.org entry

## Installation

Install Python requirements

    pip3 install -r requirements.txt
    
Optional - Install additional OCR support dependencies

- Debian/Ubuntu: `apt-get install tesseract-ocr python3-imaging`

- MAC OSX: `brew install tesseract`
- Windows: 
I see steps are scattered in different answers. Based on my recent experience with this pytesseract error on Windows, writing different steps in sequence to make it easier to resolve the error:

1. Install tesseract using windows installer available at: https://github.com/UB-Mannheim/tesseract/wiki

2. Note the tesseract path from the installation.Default installation path at the time the time of this edit was: C:\Users\USER\AppData\Local\Tesseract-OCR. It may change so please check the installation path.

3. pip install pytesseract

4. set the tesseract path in the script before calling image_to_string:

pytesseract.pytesseract.tesseract_cmd = r'C:\\Users\\USER\\AppData\\Local\Tesseract-OCR\\tesseract.exe' ^ this path can be 
 different depent on install directory of  https://github.com/UB-Mannheim/tesseract/wiki
check https://stackoverflow.com/questions/50951955/pytesseract-tesseractnotfound-error-tesseract-is-not-installed-or-its-not-i


## Usage

    usage: domainhunter.py [-h] [-a] [-k KEYWORD] [-c] [-f FILENAME] [--ocr]
                        [-r MAXRESULTS] [-s SINGLE] [-t {0,1,2,3,4,5}]
                        [-w MAXWIDTH] [-V]

    Finds expired domains, domain categorization, and Archive.org history to determine good candidates for C2 and phishing domains

    optional arguments:
    -h, --help            show this help message and exit
    -a, --alexa           Filter results to Alexa listings
    -k KEYWORD, --keyword KEYWORD
                            Keyword used to refine search results
    -c, --check           Perform domain reputation checks
    -f FILENAME, --filename FILENAME
                            Specify input file of line delimited domain names to
                            check
    --ocr                 Perform OCR on CAPTCHAs when challenged
    -r MAXRESULTS, --maxresults MAXRESULTS
                            Number of results to return when querying latest
                            expired/deleted domains
    -s SINGLE, --single SINGLE
                            Performs detailed reputation checks against a single
                            domain name/IP.
    -t {0,1,2,3,4,5}, --timing {0,1,2,3,4,5}
                            Modifies request timing to avoid CAPTCHAs. Slowest(0)
                            = 90-120 seconds, Default(3) = 10-20 seconds,
                            Fastest(5) = no delay
    -w MAXWIDTH, --maxwidth MAXWIDTH
                            Width of text table
    -V, --version         show program's version number and exit
  
    -P,  --proxy ,        required=False, default=None, help="proxy. ex https://127.0.0.1:8080")
    -u,  --username,      required=False, default=None, type=str, help="username for expireddomains.net")
	-p,  --password,      required=False, default=None, type=str, help="password for expireddomains.net")
    -o,  --output,        required=False, default=None, type=str, help="output file path")
    -ks, --keyword-start, help='Keyword starts with used to refine search results', required=False, default="", type=str, dest='keyword_start')
    -ke, --keyword-end, help='Keyword ends with used to refine search results', required=False, default="", type=str, dest='keyword_end')
	
    Examples:
    ./domainhunter.py -k apples -c --ocr -t5
    ./domainhunter.py --check --ocr -t3
    ./domainhunter.py --single mydomain.com
    ./domainhunter.py --keyword tech --check --ocr --timing 5 --alexa
    ./domaihunter.py --filename inputlist.txt --ocr --timing 5

Use defaults to check for most recent 100 domains and check reputation
    
    python3 ./domainhunter.py

Search for 1000 most recently expired/deleted domains, but don't check reputation

    python3 ./domainhunter.py -r 1000

Perform all reputation checks for a single domain

    python3 ./domainhunter.py -s mydomain.com

    [*] Downloading malware domain list from http://mirror1.malwaredomains.com/files/justdomains

    [*] Fetching domain reputation for: mydomain.com
    [*] Google SafeBrowsing and PhishTank: mydomain.com
    [+] mydomain.com: No issues found
    [*] BlueCoat: mydomain.com
    [+] mydomain.com: Technology/Internet
    [*] IBM xForce: mydomain.com
    [+] mydomain.com: Communication Services, Software as a Service, Cloud, (Score: 1)
    [*] Cisco Talos: mydomain.com
    [+] mydomain.com: Web Hosting (Score: Neutral)

Perform all reputation checks for a list of domains at max speed with OCR of CAPTCHAs

    python3 ./domainhunter.py -f <domainslist.txt> -t 5 --ocr

Search for available domains with keyword term of "dog", max results of 25, and check reputation
    
    python3 ./domainhunter.py -k dog -r 25 -c

     ____   ___  __  __    _    ___ _   _   _   _ _   _ _   _ _____ _____ ____
    |  _ \ / _ \|  \/  |  / \  |_ _| \ | | | | | | | | | \ | |_   _| ____|  _ \
    | | | | | | | |\/| | / _ \  | ||  \| | | |_| | | | |  \| | | | |  _| | |_) |
    | |_| | |_| | |  | |/ ___ \ | || |\  | |  _  | |_| | |\  | | | | |___|  _ <
    |____/ \___/|_|  |_/_/   \_\___|_| \_| |_| |_|\___/|_| \_| |_| |_____|_| \_\

    Expired Domains Reputation Checker
    Authors: @joevest and @andrewchiles

    DISCLAIMER: This is for educational purposes only!
    It is designed to promote education and the improvement of computer/cyber security.
    The authors or employers are not liable for any illegal act or misuse performed by any user of this tool.
    If you plan to use this content for illegal purpose, don't.  Have a nice day :)

    [*] Downloading malware domain list from http://mirror1.malwaredomains.com/files/justdomains

    [*] Fetching expired or deleted domains containing "dog"
    [*]  https://www.expireddomains.net/domain-name-search/?q=dog

    [*] Performing domain reputation checks for 8 domains.
    [*] BlueCoat: doginmysuitcase.com
    [+] doginmysuitcase.com: Travel
    [*] IBM xForce: doginmysuitcase.com
    [+] doginmysuitcase.com: Not found.
    [*] Cisco Talos: doginmysuitcase.com
    [+] doginmysuitcase.com: Uncategorized
