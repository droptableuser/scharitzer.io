---
title: "Password based authentication"
date: 2022-04-06
tags: ['AppSec']
---
The ugly truth about passwords, or memorized secrets, is, that we still have way too many. They won't go away any time soon either. They offer some benefits for the end user and the organisation providing password based authentication.
<!--more-->
1. Privacy: Nobody is informed about your user using your service/application in any way. It is just between them, you, all ISPs involved and most probably 2-3 financial institutions.
2. Dependency: Your service is not dependent on any other authentication service. An outage at an authentication provider can severely hinder your abilities to grant your users access to services they want to consume. 

There are several alternatives to memory based secrets (PINs, passwords, and passphrases) that I will not dive into in this blog post. You are most probably here because you need to secure a password based authentication as much as possible. Look no further, let's run down the list.

## Things you should start doing

### Multi-factor Authentication
Just enable it. Remind your users to enable it. This is the single most impactful improvement you can do. It is not bulletproof but it adds a layer of security where you or your users can catch that something is wrong.

### Passwort Auditing
This is the biggest change that came into existing in this [guideline](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5) , that only a few people were doing already. I started password auditing for improving security sometime around 2012. Usually this would have been an activity performed by a password cracker once a password database has been stolen, but we can harness the same power for good.

There are two ways of auditing the passwords:
#### On change
Every time a user change a password, the password should be validated against the following criteria:
1. Weak password
2. Leaked Password
3. Common Words
4. Service specific words (e.g.: my instagram password should not be `Marku$1nstagram2022!`

The easiest way for the leaked password list is using something like this  [API](https://haveibeenpwned.com/API/v2#SearchingPwnedPasswordsByRange) from haveibeenpwnd.
If you don't want to send any part of your password (in this case the first 5 characters of the hash) you can implement a simple password checker service in your network. You just have to fill the database with fresh password leaks once in a while.

#### Continually
Passwords are leaked every day, user accounts are compromised every day. So you should also monitor for stolen credentials on a continual basis. In a corporate environment this can be supplemented by monitoring for your e-mail domains as well. 

Auditing your passwords can be done as simple as running a leaked password list against your password storage with a tool like hashcat. Every password cracked that way needs to be changed by the user on next login. 

### Allow pasting of passwords
This is required for password managers, which you could encourage your users to use, by showing a simple pop up on your login screen. 

### Migrate your passwords to secure password storage formats
The list include:
* Argon2: the winner of the password hashing competition
* PBKDF2: an algorithm initially designed to derive cryptographic keys from passwords, also makes for a solid password storage function

Something that has been said for years in the password hashing area: Increase your workload to at least 10,000. What is workload? Workload means how many iterations of the function have to be run, before the submitted password and the stored hash can be compared. This has very little impact on the user experience and a lot of impact on your reaction time, if someone steals your password database. 

How big is the impact? There are benchmarks done by password cracking tools which are [publicly](https://gist.github.com/Chick3nman/e4fcee00cb6d82874dace72106d73fef) available.
An example:
```
sha256(sha256($pass).$salt)
Speed.#1.........:  2660.0 MH/s (64.43ms)

Python passlib pbkdf2-sha512 (Iterations: 24999)
Speed.#1.........:    57305 H/s (59.87ms)
```
This means that PBKDF2 (57 thousand hashes per second) is about 50.000x slower to crack than SHA256(2 billion hashes per second). This is negligible for your user, most of the time spent waiting for this to finish will most probably still be network latency, and this is the most critical part of verification. An attacker taking 50.000x longer to crack your passwords will not make them any happier.

## Things you can stop doing
### Password Length & Complexity
 
Stop enforcing a maximum password length. If I want to use a story of H.P. Lovecraft as my passphrase, your password field should not limit me. The [guideline](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5) mentions a minimum length of 8 characters. I personally would up this limit to 12 depending on your use case. It also mentions that your service should at least accept 64 characters. 

Password complexity is often thrown into the mix when we talk about password length. Without going into too much detail on this issue here. The guideline states to accept all ASCII printable characters. There should literally be no need for you to filter out characters like `'` and `;`.

Complexity rules just make for passwords that computers can crack easily, and humans have a hard time to remember.

### Password Changes

The biggest change in this guideline is related to regular password changes that many corporate users have suffered from for years. This part is a bit specific to corporate security, but bear with me, because the remedy to this issue also helps you not enforce arbitrary complexity rules as well. 

If you managed to do all the steps in the first section of this post, you are good to stop requiring your users to changed their password without reasonable suspicion that their account has been compromised.
