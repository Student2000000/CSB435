# Assignment 3
#### Questions: 
1) Roles: A list of roles with brief descriptions
2) Permissions: A list of permissions with brief descriptions
3) Role-Permission Matrix: A clear mapping showing which permissions belong to which roles
4) Rationale: Brief explanations of key design decisions
5) List of things you decided are not necessary (if appropriate)
6) List of things you would include/flesh out if you had more than 2 hours (if appropriate)
7) Any areas you think are week in your model, or you're less confident about

---
#### Answers

## 1
Hiring manager- A person in charge of reviewing interview notes and having a say in the rejection or acceptance of the applicant. A person with a higher level of control over the company, someone who would have the final say in wither or not application is rejected or not.  

 
Interviewer- A person in charge of conducting interviews and reviewing relevant applicant information  


Applicant- The person applying for a position 


## 2 
- Reviewing profile information- Ability to see key information about a candidate 


- Reviewing all candidate specific documentation- Ability to see potentially sensitive documentation provided by the candidate, and any other relevant information they would've provided about themself.  


- Job posting- Ability to post a job opening.  


- Job post deletion- Ability to delete a job posting.  


- Job post edit- Ability to modify a job posting. 


- Application submission- Ability to apply for a position.  


- View application status- Ability to view the status of an application  


- Change application status- Ability to change the status of an application.  


- Profile edit- Ability to change key information about your own profile. 


- Writing notes- Ability to write/view notes about an application. 


## 3

| Permission                                    | Role           |
| --------------------------------------------- | -------------- |
| Reviewing profile information                 | All            |
| Reviewing all candidate specific documentation | Hiring manager, Interveiwer|
| Job posting                                   | Hiring manager |
| Job post deletion                             | Hiring manager |
| Job post edit                                 | Hiring manager |
| Application submission                        | Applicant |
| View application status                       | Applicant |
| Change application status                     | Hiring manager, Interveiwer|
| Profile edit                                  | Applicant |
| Writing notes                                | Hiring manager, Interveiwer|


## 4

I decided to keep the permissions granular for this, there are arguably many more permissions I could've added, more in the realm of admin work for the system, but in the interest of time I wanted to hone in on how the user would interact with the system for the sole purpose of tracking applications. Most of the permissions are tied to what a Hiring manager/interviewer would need at the bare minimum to use this ATS, since this service is mostly for them if I understand the instructions correctly. I wanted there to be a clear hierarchy, where the interviewer sees only the relevant information about the client, while the hiring manager has the over all view of the system and candidates. I also included the applicant themself in place of an admin role. If this particular ATS allows for applicants to view the status of their applications then there certainly needs to be a very, very, clear separation of concerns regarding who can see what. My main (and most basic) thought was to have the hiring manager be at the highest tier, with the interviewer in the middle and the applicant at the bottom. 


## 5 

I decided not to include a company profile, or for interviewers or hiring managers to have any kind of profiles visible to the applicant. I think that this would've added on more layers of complexity than necessary for the purpose of this assignment, seeing as this is for the company to track the applications they receive first and foremost. The ability to post a job offering, edit or delete that offer as things changed, I thought was the most pertinant for a simple application tracing service. I omitted admin work for this reason too with the idea that the sole purpose of this ATS would be to track applications only. 

## 6 
I think the ability to run statistics for the job offers both from the applicant and business side would've been neat! Maybe specific permissions that allow a business to filter by certain categories or roles in the statistics. Or even the ability to see what time of day specifically candidates were more likely to apply, and if I did go that route I would've split the permission such that the hiring manager could see the really fine grain stats, while a interviewer would see maybe which roles were most applied to and so on. I think that would've been really cool! 

## 7 
I think my model is almost too basic. I work slow, and this was the result of my ~2 hrs. I really wanted to keep things simple, both so I didn't go over the ~2hr mark and also so I could focus on what made sense for what I have. I think with more time the addition of admin stuff and the ability for the company to have more fine grain controls over something like a profile. Additionally, I think I condensed a lot of stuff that with time I would've expanded upon. Like "Change application status" For instance, that could mean a variety of things and each could be broken up for different roles, say they got the job, the hiring manager could change the status to "Hired" or "Approved" where as the interviewer would not need that permission, but could put a "Phone Screen" tag instead. 





