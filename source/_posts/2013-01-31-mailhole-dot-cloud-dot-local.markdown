---
layout: post
title: "mailhole.cloud.local"
date: 2013-01-31 12:01
comments: true
categories: 
---
Thanks to jimr, we now have a locally running system mirroring the functionality of mailinator at mailhole.cloud.local; [DEVOPS-96](https://jira.cloud.local/browse/DEVOPS-96) is now being closed, and integration tests still need to be updated, but to test it out, try the following.

First, `brew install msmtp`; 

Now, drop a local file out on your machine; the parser for Mailhole expects that the body of the email contains all the details of the message – the envelope isn't used for parsing.

    cat > /tmp/email.txt <<EOF
    From: bug@knightlabs.com
    To: testuser@somesite.com
    Subject: This is a test!
    
    Testing!
    EOF

Now simply run:

    msmtp -v \
      --host=mailhole.cloud.local \
      --port=2525 \
      --from=bug@knightlabs.com testuser@somesite.com \
      < /tmp/email.txt

msmtp requires passing in the from parameter, even though its not used by mailhole.

Finally, check out http://mailhole.cloud.local:8025/mailboxes/testuser@somesite.com to see the JSON results for parsing.

Just for the record, the mailhole installation running uses 2525 for the SMTP port and 8025 for the web endpoint. Details on the code itself are at https://github.com/blackboard/cloud-test-tools/tree/develop/smtp_sink