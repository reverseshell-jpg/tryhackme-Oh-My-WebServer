# TryHackMe: Oh-My-WebServer

## 1. Recon

<img width="660" height="154" alt="image" src="https://github.com/user-attachments/assets/6ece80a6-3364-4363-bf88-556960486625" />

The server runs SSH and HTTP.

After fuzzing directories with various tools, I couldn't find any valuable API or endpoint.

Next, I used the **Wappalyzer** browser extension (Technology profiler) to identify the versions and services the website was using.

<img width="499" height="545" alt="image" src="https://github.com/user-attachments/assets/6251be4a-ec1d-45af-8120-14f6d483de69" />

It was easy to see the server was running **Apache 2.4.49**.

After searching online, I found **CVE-2021-41773** ([reference](https://github.com/mr-exo/CVE-2021-41773)), which could be used for this box.

## 2. Exploit

A quick overview of CVE-2021-41773:
- While parsing the URL string, if Apache detects the sequence `%2e` (URL-encoded for `.`), it decodes it to `.`.
- **Logic flaw:** the validation function only checks for the standard `../` pattern, but decodes each character without recursively re-checking the entire string after decoding.

**Getting a shell:**

<img width="937" height="51" alt="image" src="https://github.com/user-attachments/assets/bebf0a28-9129-4938-a6b5-4f4a75ca3a02" />

*(Note: when the request is pointed at `/bin/sh`, Apache treats `/bin/sh` as a CGI script. Per the CGI spec, any output (stdout) from the child process returned to the web server must be formatted as a valid HTTP response header.)*

<img width="610" height="111" alt="image" src="https://github.com/user-attachments/assets/47cf0a26-c253-4996-b9f6-36a19268d6d6" />

## 3. Getting user.txt

However, our current shell is running inside a Docker container.

Using **linpeas** to scan, I found an interesting detail:

<img width="329" height="52" alt="image" src="https://github.com/user-attachments/assets/58e9afbd-a130-4d62-b2db-54f0754e9c54" />

This means Python 3.7 can allow a process to change its own UID (using the `setuid()` function).

<img width="350" height="36" alt="image" src="https://github.com/user-attachments/assets/882b37bd-253d-4b39-a000-003feb60722a" />

**Reading user.txt:**

<img width="302" height="36" alt="image" src="https://github.com/user-attachments/assets/be623cbc-08fb-4b8e-a7dc-b309c2cc426c" />

## 4. Getting root.txt

Wrote a simple bash script to scan for open ports in the Docker network:

```bash
for port in $(seq 20 5999); do
  timeout 1 bash -c "echo > /dev/tcp/172.17.0.1/$port" 2>/dev/null && echo "Port $port OPEN" || echo "Port $port CLOSED"
done
```

After the script finished running, port **5986** was found open.

After researching port 5986, I found a HackTricks article:

<img width="443" height="91" alt="image" src="https://github.com/user-attachments/assets/da69f67a-04bc-45f6-a27f-af1f2c87b86f" />

However, this article had been removed. Based on the title, I searched further and found **CVE-2021-38647**.

<img width="648" height="40" alt="image" src="https://github.com/user-attachments/assets/802990f8-fa5a-4b4a-83d2-8f4ffcbb4bae" />

**Reading root.txt:**

<img width="768" height="42" alt="image" src="https://github.com/user-attachments/assets/d4fa503e-af35-4a35-bab2-77e24c85bd67" />

**Done**
