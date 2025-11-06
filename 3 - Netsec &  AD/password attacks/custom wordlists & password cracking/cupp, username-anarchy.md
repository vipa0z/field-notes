## username wordlist:

```shell-session
$ sudo apt install ruby -y

$ git clone https://github.com/urbanadventurer/username-anarchy.git
$ cd username-anarchy

./username-anarchy Jane Smith > jane_smith_usernames.txt
```
2. with cewl
```
cewl
```


## password wordlist:
```shell-session
cupp -i
```


# Wordlist modifications
We now have a generated username.txt list and jane.txt password list, but there is one more thing we need to deal with. CUPP has generated many possible passwords for us, but Jane's company, AHI, has a rather odd password policy.

- Minimum Length: 6 characters
- Must Include:
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one number
    - At least two special characters (from the set `!@#$%^&*`)

As we did earlier, we can use grep to filter that password list to match that policy:

```shell-session
 grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```

This command efficiently filters `jane.txt` to match the provided policy, from ~46000 passwords to a possible ~7900. It first ensures a minimum length of 6 characters, then checks for at least one uppercase letter, one lowercase letter, one number, and finally, at least two special characters from the specified set. The filtered results are stored in `jane-filtered.txt`.

```shell-session
hydra -L usernames.txt -P jane-filtered.txt IP -s PORT -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```


---
## end of cheatsheet

#### Username Anarchy
Human creativity knows no bounds, and usernames often become a canvas for personal expression. Jane could seamlessly weave in her middle name, birth year, or a cherished hobby, leading to variations like `janemarie`, `smithj87`, or `jane_the_gardener`. The allure of `leetspeak`, where letters are replaced with numbers or symbols, could manifest in usernames like `j4n3`, `5m1th`, or `j@n3_5m1th`. Her passion for a particular book, movie, or band might inspire usernames like `winteriscoming`, `potterheadjane`, or `smith_beatles_fan`.

This is where `Username Anarchy` shines. It accounts for initials, common substitutions, and more, casting a wider net in your quest to uncover the target's username:






---
### concepts

# CUPP (crafting the pw list)
The efficacy of CUPP hinges on the quality and depth of the information you feed it. It's akin to a detective piecing together a suspect's profile - the more clues you have, the clearer the picture becomes. So, where can one gather this valuable intelligence for a target like Jane Smith?

- `Social Media`: A goldmine of personal details: birthdays, pet names, favorite quotes, travel destinations, significant others, and more. Platforms like Facebook, Twitter, Instagram, and LinkedIn can reveal much information.
- `Company Websites`: Jane's current or past employers' websites might list her name, position, and even her professional bio, offering insights into her work life.
- `Public Records`: Depending on jurisdiction and privacy laws, public records might divulge details about Jane's address, family members, property ownership, or even past legal entanglements.
- `News Articles and Blogs`: Has Jane been featured in any news articles or blog posts? These could shed light on her interests, achievements, or affiliations
# Example OSINT Findings

| Field               | Details                      |
| ------------------- | ---------------------------- |
| Name                | Jane Smith                   |
| Nickname            | Janey                        |
| Birthdate           | December 11, 1990            |
| Relationship Status | In a relationship with Jim   |
| Partner's Name      | Jim (Nickname: Jimbo)        |
| Partner's Birthdate | December 12, 1990            |
| Pet                 | Spot                         |
| Company             | AHI                          |
| Interests           | Hackers, Pizza, Golf, Horses |
| Favorite Colors     | Blue                         |
CUPP will then take your inputs and create a comprehensive list of potential passwords:

- Original and Capitalized: `jane`, `Jane`
- Reversed Strings: `enaj`, `enaJ`
- Birthdate Variations: `jane1994`, `smith2708`
- Concatenations: `janesmith`, `smithjane`
- Appending Special Characters: `jane!`, `smith@`
- Appending Numbers: `jane123`, `smith2024`
- Leetspeak Substitutions: `j4n3`, `5m1th`
- Combined Mutations: `Jane1994!`, `smith2708@`
Invoke CUPP in interactive mode, CUPP will guide you through a series of questions about your target, enter the following as prompted: