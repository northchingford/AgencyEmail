         _   _                 _____
 _____  | | | | _____      __ |_   _|__ _   _____
|_____| | |_| |/ _ \ \ /\ / /   | |/ _ (_) |_____|
|_____| |  _  | (_) \ V  V /    | | (_) |  |_____|
        |_| |_|\___/ \_/\_/     |_|\___(_)

**************************************************
* Author: Steven L. Croucher
* Date: 20261127
* Revision - v0.1 - Initial Draft
* Subject: Configuring Postfix to reject mail by
*          Domain name from a regex file
**************************************************

Requirements:

*	Postfix
*	Correct permissions to edit Postfix config
*	postfix-pcre package

Main text:

Rather than using sieve, or similar, rules to move unwanted email to a spam folder, it's possible to setup Postfix to reject mail at SMTP dialogue level and bounce unwanted mail with a 550 failure.  See this example from a Postfix log:

/var/log/mail.log.1:2026-11-20T14:35:07.412970+00:00 mailgw postfix/smtpd[36931]: NOQUEUE: reject: RCPT from v5119.v5c4dda55.use4.send.mailgun.net[159.112.241.119]: 550 5.7.1 <postmaster@cio130258.events.moneyshow.com>: Sender address rejected: Rejected as spam; from=<postmaster@cio130258.events.moneyshow.com> to=<stevenc@stevencroucher.com> proto=ESMTP helo=<v5119.v5c4dda55.use4.send.mailgun.net>

To configure your Postfix MTA to do this, you'll need to add the postfix-pcre package and create a suitable regex file with a list of domains in this format:

<domain in regex>			<Error code> <Error text>

Example:

/haverynash\.com$/			550 Rejected. Fake agency scam.

The first column is the domain name you're looking for, in this case a fake agency with a deliberate incorrect spelling of a legit firm.  Followed by a code and text to reply to the sending server.  A 5xx code is a hard failure so will stop any retries.

Once a list is completed, have postmap hash the file:

postmap /etc/postfix/sender_access

Now edit /etc/postfix/main.cf and look for the smtpd_relay_restrictions section and add this statement to the end:

check_sender_access pcre:/etc/postfix/sender_access

Example:

smtpd_relay_restrictions =      <....>
                                defer_unauth_destination,
                                reject_rbl_client zen.spamhaus.org,
                                check_sender_access pcre:/etc/postfix/sender_access


If you want to check earlier in the SMTP dialogue then add it to smtpd_recipient_restrictions instead.  I keep this check after the rbl checks, as untrustworthy mail is usually caught by spamhaus.

Hints:

*	Add your a test domain to sender_access file to test that the check really is being used.
*	Run postmap every time you edit the file and check your postfix logs for any reported errors.
*	You could use more complex regex, but I'm mostly interested in just blocking domains.	

