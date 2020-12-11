# HOWTO use fluentd to post errors into Slack channel

Prepare Slack app, go through [Sending messages using Incoming
Webhooks](https://api.slack.com/messaging/webhooks) and get webhook URL at step
4.

Create `Dockerfile`:
```
FROM fluent/fluentd:v1.9.1-debian-1.0

USER root
RUN fluent-gem install fluent-plugin-slack
USER fluent
```

Build docker:
```sh
docker build -t fluentd .
```

Add configuration, replace `<path-to-log>`, `<app-name>`, `<tag>` and
`<slack-webhook-url>` below by your values:
```
<source>
  @type tail
  path <path-to-log>/<app-name>.%Y-%m-%d.log
  pos_file /fluentd/etc/position/<app-name>.pos
  tag <tag>.error
  <parse>
    @type multiline
    format_firstline /\d{4}-\d{2}-\d{2}/
    format1 /(?<time>\S+ \S+) (?<thread>\[[^\]]+\]) (?<level>\S+) +(?<logger>\S+) - (?<message>.*)/
    time_key time
    time_format %Y-%m-%d %H:%M:%S,%L
    localtime false
  </parse>
</source>

<filter <tag>.error>
  @type grep
  <regexp>
    key level
    pattern ^ERROR$
  </regexp>
</filter>

<match <tag>.error>
  @type slack
  webhook_url <slack-webhook-url>
  channel <channel-name>

  message "<app-name> error at %s, message: %s"
  message_keys time,message
  flush_interval 0s

  <buffer>
    @type memory
    flush_mode immediate
  </buffer>
</match>
```

Create directory to keep position:
```sh
mkdir position
chown <user>:docker position
```

Start docker:
```sh
docker run --rm --name fluentd \
	-p 9880:9880 \
	-v <path-to-log>:<path-to-log> \
	-v `pwd`:/fluentd/etc \
	-e FLUENTD_CONF=fluentd.conf \
	fluentd 2>&1 >/dev/null &
```
