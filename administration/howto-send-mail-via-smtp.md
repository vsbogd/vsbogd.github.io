
Start telnet:
```sh
telnet aspmx.l.google.com 25
```

Send local domain:
```smtp
ehlo local.domain
```

Enter sender address:
```smtp
mail from:<user@local.domain>
```

Enter recipient address:
```smtp
rcpt to:<user@mail.com>
```

Enter mail body:
```smtp
data
Subject: test
From: user@local.domain
Test message
.
```

Quit telnet:
```smtp
quit
```
