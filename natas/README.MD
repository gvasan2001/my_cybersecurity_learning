# Natas – Server-Side Web Security

**Natas** is a web security challenge that teaches the fundamentals of server-side web security. 

Each level focuses on a specific vulnerability or security weakness. The objective is to identify and exploit these vulnerabilities to obtain the password required to access the next level.

---

## Default Credentials (Level 0)
* **Username:** `natas0`
* **Password:** `natas0`

---

## Natas Level 0 → Level 1

### Description
The password for Level 1 is exposed directly within the webpage's client-side source code.

### Steps to Solve
1. Open the **Natas Level 0** webpage.
2. Right-click on the page and select **Inspect** or **View Page Source**.
3. Examine the HTML source code.
4. Locate the commented-out or exposed password.
5. Use the discovered password to log in as `natas1`.

### Key Learning
Sensitive information, credentials, or keys should **never** be exposed in client-side HTML/JavaScript source code, as anyone can view it.

###  Level 1-Credentials
* **Username:** `natas1`
* **Password:** `scfWG6qNEIdzqVyfRwEGXyNUfFZkZeQ7`
---

## Natas Level 1 → Level 2

### Description
The password for Level 2 is exposed directly within the webpage's client-side source code. However, right-clicking is disabled on this website..

### Steps to Solve
1. Open the **Natas Level 1** webpage.
2. Since right-clicking is restricted, press the F12 key on your keyboard (or the specific shortcut depending on your laptop) to open the Developer Tools.
3. Examine the HTML source code.
4. Locate the commented-out or exposed password.
5. Use the discovered password to log in as `natas2`.

### Key Learning
Sensitive information, credentials, or encryption keys should **never** be exposed in client-side HTML or JavaScript source code. Even if a website restricts right-clicking, attackers have multiple other ways to access and view the source code.

 

### Level 2-Credentials
* **Username:** `natas2`
* **Password:** `vsDOxoXyq3wckCP1ZmTZ71ngIA606odB`


## Natas Level 2 → Level 3

### Description
The password for level 3 is exposed in the `/files/users.txt` file. Directory 
indexing is enabled on the server, so we can browse the directory listing and 
see all the files — one of which, `users.txt`, exposes sensitive data like 
the password.

### Steps to Solve
1. Open the **Natas Level 2** webpage.
2. Right-click on the page and select **Inspect** or **View Page Source**.
3. Examine the HTML source code — there's an image tag `<img src="files/pixel.png">` 
   that reveals the directory name.
4. The `files` folder does not have an index file, so the server lists all 
   files in it — one of which is `users.txt`.
5. The `users.txt` file reveals the password to log in as `natas3`.

### Key Learning and Mitigation
Do not allow directory listing to be enabled on the server, and do not store 
sensitive data in plaintext files — keep it in a secure location instead.

### Level 3 Credentials
- **Username:** `natas3`
- **Password:** `K30JrSRHzjxq3paUQuwozY4MNvmNFyhI`



## Natas Level 3 → Level 4

### Description
This lab's password is exposed via `robots.txt`, which contains a `Disallow` entry for the path `/s3cr3t/`. Look for that path — we find `users.txt`, which has the next level's password.

### Steps to Solve
1. Open the **Natas Level 3** webpage and check `/robots.txt` in the URL.
2. You discover the new page listing `Disallow: /s3cr3t/`.
3. Check this path — you discover a new page listing `users.txt`.
4. The `users.txt` file reveals the password to log in as `natas4`.

### Key Learning and Mitigation
Do not disclose sensitive paths in `robots.txt`. This misconfiguration can lead to the discovery of admin pages or restricted pages containing sensitive information.

### Level 4 Credentials
- **Username:** `natas4`
- **Password:** `JDrPnuZAKyl6MkiqQGFIddrqpvgOASth`
