---
layout: post
title: Toggl reports to Slack for free plan
---

I wanted to expose a bit of insight to my team. As we work in kind of upredictable manner we could use a report on who worked a day before. We use Trello as our time tracking tool. It supports weekly email reports but the price for this feature was a bit expensive for us. So I did a quick research and found [Trello report script](https://github.com/jsmartin/toggl_report) that does exactly what I wanted. It outputs [markdown](https://github.com/jsmartin/toggl_report/blob/master/sample.md) to STDOUT.

I've modified the script to include time spent for each entry and updated it to Python 3.

With the reporting part done I started to investigate how to post the report to Slack. I found three possible ways I can solve this. Lets discuss these options.

**Post the output as plaintext**

Slack's markdown support is pretty limited. It doesn't support neither headings nor lists. This approach has one notable benefit. It's one HTTP request.


**Use [message attachments](https://api.slack.com/docs/message-attachments)**

This approach might sound interesting, but it's not suitable for structured reports.

**[Upload report](https://api.slack.com/tutorials/working-with-files) as markdown file**

This approach is the best option, but I haven't had the time to implement it.

### Deployment

I've set up a cron job to run the script every day at 10:00.

```
00 09 * * * cd /home/ubuntu/toggl-report/ && /home/ubuntu/toggl-report/env/bin/dotenv-run /home/ubuntu/toggl-report/env/bin/python /home/ubuntu/toggl-report/run-report.py >> /home/ubuntu/toggl-report/log.log 2>&1
```

The project is available on https://github.com/stlk/slack-toggl-report
