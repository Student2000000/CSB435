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

Failure to clean and validate user input.

2. Write an example attack payload that would exploit this

From the textbook: 
"SELECT * FROM members WHERE
((u='admin' AND p=' ') OR 'x'='x') "

3. What damage could an attacker do with this vulnerability?

Using this attack payload allows the attacker to log in as an admin w/o ever having to know the admins password. From there the ataker could steal information, change passwords, ransom the information, or sell the information. 

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

Users can be knowlingly and unknowlingy malicious! It is not safe to trust random people not to do horrible things. 

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
    * Users can traverse through files backwards eventually reaching the root, there they can gain access to private or sensitive files. 

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

    * There's no sanitiation on the input, nothing checks that the patient id is even a number 

  * Provide an attack payload

    * DELETE FROM table
    WHERE id = patient_id

3. For the medication parameter:
  * Write a UNION injection that would dump all patient data

    * ' ; DROP TABLE patients; -

  * Explain how this attack works
    * Bc of the weird quotes thing, the attacker can just close the quote with another ' then put an executable SQL cmnd. 


### Code Snippet 5: Appointment Email Notifications
```
def send_appointment_reminder(email, patient_name, appointment_time):
    subject = f"Appointment Reminder for {patient_name}"
    body = f"Hello {patient_name}, your appointment is at {appointment_time}"

    cmd = f'/usr/bin/mail -s "{subject}" {email}'
    subprocess.call(cmd, shell=True, input=body.encode())
```
1. What injection vulnerability exists here?

Shell injection

2. Provide an attack payload that would execute arbitrary commands on the server

John Doe"; rm -rf /; #

3. Explain step-by-step how the attack works

By finsihing the quote then using a command seportor ";", the attacker can then write system commands that will be exectued on the shell because the code contains shell=True

4. When should shell=True be used in python subprocess?
 
When a command reles on shell features (and preferably when there's no user input involved.)

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

Answers for question 2:

1) 
  * Use of outdated hashing algorothm; MD5
  * MD5 was outdated bc of it's suceptibility to collison attacks, and bruteforcing. 
  * I would use SHA-256 or SHA-3 there instead, as you recomended from your slides. 

2) 
  * No salting in password_hash
  * Since the same hash is used for all the passwords and the hashing algorithm is susceptible already, cracking one password could compromise the others. 
  * According to Google, one of the main issues w/ a lack of salting is that it makes passwords susceptible to rainbow table attacks. Salts would generate a unique random string for each user then concatinate it with their acyal passwrod then hash that, thus making each encrpyted password compleatly unique, and less/completely susceptible to rainbow table attacks 

Answers to question 3:
Yes, I think it does. Firstly the function uses an ifelse loop, which would return back quicky if false, and secondly the SQL call would also fail quickly. I think the timing of these returns would be exploitable by a timing attack. 

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

  * The code uses the user's id for part of the token, and I think it stores the token with the user_id to be reused? 

2. How could an attacker predict these session tokens?

   * If the attacker knows the user_id, they already have a good chunk of the token, and bc the token also uses a randint in a 4 digit number by understaning the session ID format they might be able to use bruteforce. 

3. How long (length) should session tokens be? Why?

128 bits, bc it makes 2^128 possible combinations, which is really hard to guess/bruteforce. 

4. What's the difference between random and secrets modules in Python?

Random isn't made for security, it's just a silly little guy that spits out (somewhat) random numbers for us, secrets is actually cyptographic in nature. 

# Part 3: Risk Assessment

Provide your risk categorization for ALL the vulnerabilities you found (Snippets 1-7) from most to least dangerous in a markdown table.

The columns should be 'Rank' (0-6, 0 being 'worst'), 'Snippet #', 'Vulnerability name', and 'Severity' (Critical / High / Medium / Low).

Then answer:

1. Which single vulnerability would you fix first if you only had time for one? Why? (~paragraph)
2. Which vulnerability do you think would take the longest to fix? (~paragraph)
3. How much do you plan to charge for your security review? (~sentences)
4. Do you plan to sign up for FroggyHealthPortal? Explain why / why not. (~sentences)

| Rank | Snippet # | Vulnerability name      | Severity |
| ---- | --------- | ------------------      | -------- |
| 0    |  1        | Patient Search          | Critical |
| 1    |  4        | Prescription Lookup     | Critical |
| 2    |  3        | Pat. Notes Display      | Critical |
| 3    |  2        | Med. Rep. Generator     | Critical |
| 4    |  5        | Appt. Email Notifs.     | Critical |
| 5    |  7        | Session Token Generation| Critical |
| 6    |  6        | Password Storage        | Critical |

Tbh they were all pretty bad in my mind

* If I could fix one only I would choose #1: Patient Search. Just on the basis that if the attacker did get in though the session token, or commnd injection, or all the other problems this fake app so clearly has I would rather they not delete or take any patient data if possible. I think just about anything is better than what they have going on now tbh. But maybe this is a chicken or the egg thing, or how they say you should actually leave your car unlocked in shady areas just w/ nothing valuable in it such as to avoid a broken window. Like- would it be better to prevent them from getting in, or just protect well what's inside so they have nothing to take? 

* In my mind, the prescription and notes display one. I feel like that one runs deep. As in the issues are pretty deep rooted and would be better off just having been re-written rather than just fixed, a process that would likely take the longest in the long run (no pun intended). 

* Omg. I don't even know if I would charge them bc I think the whole thing just needs to be re-written all together, which is the job of a dev over a security person. You know what?- $50 just for the trouble of a. taking my time to show me this and b. for me having to tell them that they need a new app. 

* Hell no! (Can I say hell?) That's a one way ticket to getting your health data leaked, or ransomed, or just straight up deleted. Not a chance in the world. 



