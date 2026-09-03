# Natas – Server-Side Web Security

**Natas** is a web security challenge that teaches the fundamentals of server-side web security. 

Each level focuses on a specific vulnerability or security weakness. The objective is to identify and exploit these vulnerabilities to obtain the password required to access the next level.

---

## Default Credentials (Level 0)
* **Username:** `natas0`
* **Password:** `natas0`

---

## Level 0 → Level 1

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

---

## Level 1 → Level 2

### Credentials
* **Username:** `natas1`
* **Password:** `scfWG6qNEIdzqVyfRwEGXyNUfFZkZeQ7`

