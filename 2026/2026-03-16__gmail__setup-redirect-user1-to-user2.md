# Gmail Admin: Redirect Emails from Specific Senders (User1 → User2)

## Overview

This guide explains how to set up server-level email redirection in Google Workspace Admin Console so that emails from specific senders addressed to `user1@mydomain.com` are automatically redirected to `user2@mydomain.com` — without user1 receiving a copy.

---

## Steps

### 1. Access Compliance Settings

Go to Google workspace Admin:

```
Apps > Google Workspace > Gmail > Compliance
```

### 2. Add a Rule

Scroll to **Content Compliance** and click **Configure** or **Add Another Rule**.

### 3. Basic Setup

- **Description:** Give it a name, e.g.:
  ```
  Redirect Specific Senders User1 to User2
  ```
- **Email messages to affect:** Check the following:
  - ✅ Inbound
  - ✅ Internal – Receiving

### 4. Define Your Senders (Expressions)

1. In the **Expressions** section, select:
   > *"If ANY of the following match the message"*

2. Click **Add** and select **Advanced content match** from the dropdown.

3. Configure the match:

   | Field       | Value                        |
   |-------------|------------------------------|
   | Location    | Sender header                |
   | Match type  | Matches regex (or Equals)    |
   | Content     | *(see below)*                |

4. For multiple senders, use a **regex** pattern in the Content field:

   ```regex
   ^(sender1@domain\.com|sender2@other\.com|sender3@example\.com)$
   ```
   ```regex(in case you want to add domains instead of Individual emails)
   ^(?i).+@(domain1\.com|domain2\.com|domain3\.com)$ 
   ```
   > Add as many senders as needed using the `|` (pipe) separator.

### 5. Configure the Redirection

Under **Actions**, select **Modify message**, then:

- ✅ Check **Change envelope recipient**
- Select **Replace recipient**
- Enter the destination address:
  ```
  user2@mydomain.com
  ```

### 6. Ensure User1 Does NOT Get a Copy

1. Scroll to the bottom and click **Show options**.
2. Under **Envelope filter**, check:
   > ✅ *Only affect specific envelope recipients*
3. Select **Single email address** and enter:
   ```
   user1@mydomain.com
   ```

> ⚠️ **Important:** Do **NOT** check *"Also route to original destination"*. By replacing the recipient without enabling this option, the email is diverted entirely to user2 and user1 receives nothing.

### 7. Save

Click **Save** or **Add Setting** to activate the rule.

---

## Summary

| Setting | Value |
|---|---|
| Rule location | Gmail > Compliance > Content Compliance |
| Messages affected | Inbound + Internal – Receiving |
| Filter method | Regex match on Sender header |
| Redirect to | user2@mydomain.com |
| Original recipient suppressed | Yes (user1@mydomain.com) |

---

## Notes

- Changes may take **up to 24 hours** to propagate, but typically apply within a few minutes.
- To redirect **all** of user1's mail (not just specific senders), skip Step 4 entirely.
- To add more senders later, edit the regex expression in the rule.
