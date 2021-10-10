---
layout: post
title: Getting the phish with Mailchimp
---

I am in a friendship group which likes to do light-hearted pranks; removing labels from food tins, covering bedrooms in tinfoil, nothing to cause any negative effects to the other. Now we are all graduated, split across the UK and never had the chance to have a graduation ceremony. With lock-down in full swing, there was an idea. Why not a lighthearted prank / [rickroll](https://www.youtube.com/watch?v=dQw4w9WgXcQ) them with an email? 

Like most subscription lists and common emails ypu see, there's the iconic smiling monkey at the footer showing it was crafted by the great provider [Mailchimp](https://mailchimp.com). This time, the email would persuade them into clicking what would be their *"Virtual Graduation"*, but would actually be a video on YouTube by [big man tyrone](https://www.youtube.com/channel/UCi7BsyeNmdfZa37Ep8kl2LQ).

<!-- More -->

<iframe width="560" height="315" src="https://www.youtube.com/embed/1Bix44C1EzY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"></iframe>


## The Disclaimer
<div class="message">
	This was done as a simple joke to friends and has been kept within a small set of email addresses. It is *never* the intention of this post to misrepresent the university or cast any negative light. This post serves as some evidence for what capability a phisher could do with Mailchimp and what you can do in retaliation. If you have any issues with this, please contact me.
</div>

## The Setup

As the method of delivery was chosen to be Mailchimp, a simple account was made as per the usual process. To convey a somewhat sense of 'validity' the same colour schema was used from a previous email sent out by the University. Recipients were already known as they were friends, plus the use of custom name-tagging through Mailchimp helps spam-filtering. 

One surprising feature is the `From` section for a campaign, what email address is given to the user in their inbox as to who sent it. Whilst email domain verification is done when creating an account, this field could be filled in with an arbitrary email address. To keep it safe, I filled in `test@domain.com` with my personal email, but chose to give an alias so that it wouldn't *appear* to be from me. Of course, if the user clicked on the alias, then my email would be revealed.

<img width="90%" class="container" src="/assets/media/2021/03/06/from-section.png" alt="The from section in mailchimp giving choice of an arbitrary sender to the user">

Other things such as design and layout can be done with enough time, pulling from a template or from scratch. The email was crafted with the 'payload' as a web-link in a button labelled 'Virtual Graduation', it was ready to go.


## Delivery

The send-off was not as climactic as you would first think, the email was sent. No news for about an hour. It was only after a while did someone from the group **even bothered** to click on a button was there chatter about the silly prank. Yet, it was due to this first person did everyone else bother to fully read the email and check their version of the button, implying most emails are simply skimmed and left. That conclusion isn't such a revelation, the real revelation came shortly after the next day.

## Aftermath

Here's the catch, the **actual** graduation email from the University followed less than 24 hours later than this trick one. Coincidence? Surprisingly, yes. But the first thought was shock, the slim time nature set off alarm bells in my mind. This use of time frame, *i.e. getting to them before the official one* could done for other news in the more general, larger sense. 

This is the official email from the University:



<img class="container" src="/assets/media/2021/03/06/official-email.png" alt="The official email from the university regarding graduations">

And mine the day before:


<!-- <img src="/assets/media/2021/03/06/phish-email-a.png" alt="The phish email sent a day before"> -->
<!-- <img src="/assets/media/2021/03/06/phish-email-b.png" alt="The phish email sent a day before"> -->

<img class="container" src="/assets/media/2021/03/06/phish-email.png" alt="The phish email sent a day before">


You can see some clear similarities and differences; the social media icons, the slightly different logo for the University and a lack of an individual (there was no person at the end saying 'best regards' or 'yours'). For the more trained eye in security, my personal email is the biggest red flag (on purpose) to indicate false news. Plus, the long url link at the bottom isn't commonplace for professional emails. However, icons for social media on the phishing email also do take people to the legitimate sites and the section on the Alumni Network also was pulled from a web page made by the University. 

This shows as an example of how time and tailoring can make a pretty realistic phishing campaign. An additional point is that the link to a youtube video can easily be changed to a link to a more malicious website, plus that emails like this can be generalised to **everyone**.


## Actual defence

Mailchimp provides some good deterrence for people actually trying to do some actual phishing. The first is [pricing](https://mailchimp.com/pricing/), to get past 2,000 people in audience money has to be paid monthly. This means associating a bank account and more identity a scammer would want to reveal. Next, the use of unique campaign IDs help identify which account has been doing such malicious activity and shut down the account. Forms are also set up to help log [abuse](https://mailchimp.com/contact/abuse/) which deters bad actors from using this further. Mailchimp provides some great ways to help protect an individual from phishing, plus keep a business stay genuine to users.


------------

## A Personal Note on Phishing
Personally, phishing as an actual form of hacking is quite far down on my list in terms of actual technical genius. In the security community too, these methods are not held in high esteem either. Phishing has just come to be such a frequent and low-hanging fruit by most scammers nowadays it has lead to **85%** of all organisations [being hit by phishing](https://securityboulevard.com/2020/12/staggering-phishing-statistics-in-2020/). Even though it is disregarded as something trivial, that statistic speaks loudly. 

No one likes that feeling when they think they have just fallen for the bait. That simple click, the habitual response. I think that's what makes phishing deserve this view, you are trying to trick a victim for your gain. This is instead of trying to trick systems that don't think and feel in the same way a person would.

I can now expect your question then becomes *"Why did you do this if you don't like phishing tactics?"* The initial thought first and foremost was to simply play a small trick onto a couple of friends, and it will finish there too. It was the official email the next day which really sparked the point of this post, as it shown to me first-hand what power a simple social engineering trick can play if timed and tailored correctly.


