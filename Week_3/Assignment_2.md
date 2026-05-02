# Part 1: Injection Vulnerability Analysis

### Code Snippet 1: Patient Search
```
@app.route('/search')
def search_patients():
    name = request.args.get('name', '')

    query = f"SELECT * FROM patients WHERE name LIKE '%{name}%'"
    results = db.execute(query).fetchall()

    return render_template('results.html', patients=results)
```
1. What type of injection vulnerability exists here?

Failure to clean and validate user input. "Little Bobby Tables, we call him."

2. Write an example attack payload that would exploit this

From the textbook: 
"SELECT * FROM members WHERE
((u='admin' AND p=' ') OR 'x'='x') "

3. What damage could an attacker do with this vulnerability?

Using this attack payload allows the attacker to log in as an admin w/o ever having to know the admins password. From there the ataker could steal information, change passwords, ransom the inforation, or sell the information. 

### Code Snippet 2: Medical Report Generator
```
@app.route('/report/<patient_id>')
def generate_report(patient_id):
    patient = get_patient(patient_id)

    # Generate PDF report
    filename = f"report_{patient_id}.pdf"
    cmd = f"pandoc -o /reports/{filename} /tmp/patient_{patient_id}.md"
    os.system(cmd)

    return send_file(f"/reports/{filename}")

```
1. What type of injection vulnerability exists here?

Command injection

2. Write an example attack payload

From slides:
magick foo.png ; rm -rf / foo.gif

3. What could an attacker do with this vulnerability?

Attackers could sift through file systems usng ls or dir, or if the program has high system privledges, then they could gain root/admin access. 

4. Explain why shell commands with user input are dangerous

Users can be kowlingly and unknowlingy malicious! It is not safe to trust random people not to do hrrible things. 

### Code Snippet 3: Patient Notes Display
```
@app.route('/notes/<note_id>')
def view_note(note_id):
    note = db.execute(f"SELECT content FROM notes WHERE id={note_id}").fetchone()

    # Display note content directly
    html = f"<div class='note'>{note['content']}</div>"

    return render(html)
```
* Identify TWO different injection vulnerabilities in this code
For each vulnerability:
  * What type of injection is it? 
  * Provide an attack payload example
  * Explain the impact

1. There is no trailing slash on the end of the file directory
    * Path traversal
    * ../
    * Users can traverse through files backwards eventualy reaching the root, there they can gain access to private or sensitive files. 

2. Html is rendered directly 
    * SQL injection
    * ;ls -la or ;cat /etc/passwd
    * If there were any malicious code in the note it would be rendered when opened. 

### Code Snippet 4: Prescription Lookup
```
@app.route('/prescriptions')
def get_prescriptions():
    patient_id = request.args.get('patient_id')
    medication = request.args.get('medication', '')

    query = f"""
        SELECT p.*, m.name, m.dosage
        FROM prescriptions p
        JOIN medications m ON p.med_id = m.id
        WHERE p.patient_id = {patient_id}
        AND m.name = '{medication}'
    """

    results = db.execute(query).fetchall()
    return jsonify(results)
```
1. This code has SQL injection in TWO different parameters. Identify both.
2. For the patient_id parameter:
  * Why is it vulnerable even without quotes?

    * Test

  * Provide an attack payload
3. For the medication parameter:
  * Write a UNION injection that would dump all patient data
  * Explain how this attack works

### Code Snippet 5: Appointment Email Notifications
```
def send_appointment_reminder(email, patient_name, appointment_time):
    subject = f"Appointment Reminder for {patient_name}"
    body = f"Hello {patient_name}, your appointment is at {appointment_time}"

    cmd = f'/usr/bin/mail -s "{subject}" {email}'
    subprocess.call(cmd, shell=True, input=body.encode())
```
1. What injection vulnerability exists here?
2. Provide an attack payload that would execute arbitrary commands on the server
3. Explain step-by-step how the attack works
4. When should shell=True be used in python subprocess?
 

# Part 2: Cryptography Mistakes

### Code Snippet 6: Password Storage
```
import hashlib

def create_user(username, password):
    password_hash = hashlib.md5(password.encode()).hexdigest()

    db.execute(
        "INSERT INTO users (username, password) VALUES (?, ?)",
        (username, password_hash)
    )

def check_password(username, password):
    stored_hash = db.execute(
        "SELECT password FROM users WHERE username = ?",
        (username,)
    ).fetchone()

    password_hash = hashlib.md5(password.encode()).hexdigest()

    if password_hash == stored_hash:
        return True
    return False
```
1. Identify TWO major cryptography mistakes in this code (there are more than two)
2. For each mistake:
  * What's wrong?
  * What attack does this enable?
  * How would you fix it?
3. Does the check_password function have an exploitable timing attack vulnerability? Explain why it does or doesn't in your opinion.

### Code Snippet 7: Session Token Generation 
```
import random
import time

def generate_session_token(user_id):
    random.seed(int(time.time()))
    token = f"{user_id}_{random.randint(1000, 9999)}"
    return token

def create_session(user_id):
    token = generate_session_token(user_id)
    db.execute("INSERT INTO sessions (token, user_id) VALUES (?, ?)", (token, user_id))
    return token
```
1. What cryptography mistake makes this token generation insecure?
2. How could an attacker predict these session tokens?
3. How long (length) should session tokens be? Why?
4. What's the difference between random and secrets modules in Python?

# Part 3: Risk Assessment

Provide your risk categorization for ALL the vulnerabilities you found (Snippets 1-7) from most to least dangerous in a markdown table.

The columns should be 'Rank' (0-6, 0 being 'worst'), 'Snippet #', 'Vulnerability name', and 'Severity' (Critical / High / Medium / Low).

Then answer:

1. Which single vulnerability would you fix first if you only had time for one? Why? (~paragraph)
2. Which vulnerability do you think would take the longest to fix? (~paragraph)
3. How much do you plan to charge for your security review? (~sentences)
4. Do you plan to sign up for FroggyHealthPortal? Explain why / why not. (~sentences)

| Rank | Snippet # | Vulnerability name | Severity |
| ---- | --------- | ------------------ | -------- |
| 1    | 7         | TEST               | Critical |


