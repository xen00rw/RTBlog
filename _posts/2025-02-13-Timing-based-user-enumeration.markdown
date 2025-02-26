---
title: "Timing based user enumeration"
date: 2025-02-13 00:00:00 -0300
categories: [reconnaissance]
tags: [reconnaissance, web]
description: This post will explain you how to find valid usernames in platforms besides the use of differences between responses or codes.
image:
  path: /assets/post-images/001-TimingBasedUserEnum/header.png
  #alt: image alternative text
pin: true
---

## Summary
As you know, during red team activities, user enumeration is a really interesting tactic. It can help us move through the target environment, gaining access to internal applications, SaaS applications, and other assets. One of the most common ways to find these entry points is through login forms, password reset forms, forgotten user or account reset mechanisms. It's common to identify them by analyzing different HTTP responses or even HTTP status codes.

But have you ever wondered about finding those users through a technique based on web timing?

## What is User Enumeration ?
Username enumeration occurs when an attacker can determine valid users in a system due to a misconfiguration or a design decision, leading to a username enumeration issue.

This vulnerability is commonly found in authentication forms, registration forms, and forgotten password functionalities.

The information exposed by the system allows attackers to leverage it in further attacks, such as password spraying or brute forcing, since the username is known to be correct.

## How to?
For this article, I will use an open-source application that I found and have already reported the bug for. I will create the users `john.doe` and `root`. Let's move on.

It's important to mention that user enumeration can sometimes be found in authentication forms and forgotten password functionalities. The first thing you need to confirm is whether the application is vulnerable through common methods, such as different HTTP responses or status codes.

First of all, I assume you already know how to use [BurpSuite Proxy](https://portswigger.net/burp). So, start it and navigate to the web application you want to test for this vulnerability.

Access the form where you suspect the vulnerability exists, such as the `login form`. Try making a login request or a password reset request. Then, go to BurpSuite’s HTTP History and search for the request you just made that contains the data you submitted in the form. Send this request to the Intruder.

![Login Attempt](/assets/post-images/001-TimingBasedUserEnum/001.png){: w="1200" h="630" }{: .dark}{: .shadow}
_Login Attempt_

Select the variable in the request that you want to test, e.g., `username`.

![Intruder Config](/assets/post-images/001-TimingBasedUserEnum/002.png){: w="1200" h="630" }{: .dark}{: .shadow}
_Intruder Configuration_

On the Payloads tab, set a context-based wordlist. For example, you can use company-related usernames or default system usernames. Do your best to tailor it to the target.

![Payloads Config](/assets/post-images/001-TimingBasedUserEnum/003.png){: w="420" h="240" }{: .dark}{: .shadow}
_Payloads Configuration_

Now, one of the most important steps is configuring the test to run without concurrent requests, sending only one request at a time. To do this, you should create a new resource pool, as shown in the image below.

![Resource Pool Config](/assets/post-images/001-TimingBasedUserEnum/004.png){: w="420" h="240" }{: .dark}{: .shadow}
_Resource Pool Configuration_

Now it's time to launch the attack and carefully check the response time for each request.

![Results](/assets/post-images/001-TimingBasedUserEnum/005.png){: w="1200" h="630" }{: .dark}{: .shadow}
_Results - Attack 1_

The difference in response times between the first two usernames is clearly noticeable. Now, let's run the attack two more times to confirm the results.

![Results](/assets/post-images/001-TimingBasedUserEnum/006.png){: w="1200" h="630" }{: .dark}{: .shadow}
_Results - Attack 2_

![Results](/assets/post-images/001-TimingBasedUserEnum/007.png){: w="1200" h="630" }{: .dark}{: .shadow}
_Results - Attack 3_

If you are looking for a time-based user enumeration vulnerability, you need to notice a stark difference in HTTP response times. Observe that the users root and `john.doe` consistently take longer to process the login attempt.

To analyze this further, let's compile all the results into an table.

![First results table](/assets/post-images/001-TimingBasedUserEnum/008.png){: w="420" h="240" }{: .dark}{: .shadow}
_Results Table_

Now, using a basic color scheme, we can categorize the values into high, medium, and low response times. It's already clear that there is a significant difference between these results.

![Second results table](/assets/post-images/001-TimingBasedUserEnum/009.png){: w="420" h="240" }{: .dark}{: .shadow}
_Parsed Results Table_

By analyzing this table, we can determine which usernames are valid and registered in the application.

## Doubts?

In some cases, you may need to deal with closely timed execution results, where the difference can be as small as half a millisecond. However, the variation will still be noticeable.

Network jitter can affect the accuracy of the attack, which is why we always recommend running the attack 3 to 5 times to better compare the values and confirm the findings.

## How to fix?

It's important to consider these solutions:

1. Implement a constant-time authentication process – Ensure that the application takes the same amount of time to process authentication attempts, regardless of the username.
2. Introduce a fixed delay – Equalize response times by adding a fixed delay for each login attempt.
3. Use asynchronous functions – Allow authentication processes to run in the background to prevent noticeable differences in execution time.