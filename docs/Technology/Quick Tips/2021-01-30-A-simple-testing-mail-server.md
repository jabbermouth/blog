# A simple testing mail server

If you need a quick mail server to test your mail logic works without actually sending these test emails, I suggest using Mailhog as it will not only swallow your test emails but it also provides a simple UI to see the messages you’ve sent.

To get it up and running requires one simple Docker command:

```powershell
docker run -p 25:1025 -p 8025:8025 -d --restart always --name mailhog mailhog/mailhog
```

With this, you can send mail on port 25 via SMTP and view the dashboard at [http://localhost:8025/](http://localhost:8025/) and see any mail you’ve received.