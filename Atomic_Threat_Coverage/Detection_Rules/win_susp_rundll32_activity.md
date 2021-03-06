| Title        | Suspicious Rundll32 Activity |
|:-------------------|:------------------|
| Description        | Detects suspicious process related to rundll32 based on arguments |
| Tags               |   |
| ATT&amp;CK Tactic | ('Defense Evasion', 'TA0005'), ('Execution', 'TA0002')  |
| ATT&amp;CK Technique | T1085  |
| Dataneeded         | DN_0002_windows_process_creation_with_commandline_4688, DN_0003_windows_sysmon_process_creation_1 |
| Triggering         | T1085 |
| Severity Level     |        |
| False Positives    | False positives depend on scripts and administrative tools used in the monitored environment |
| Development Status | experimental      |
| References         | http://www.hexacorn.com/blog/2017/05/01/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline/, https://twitter.com/Hexacorn/status/885258886428725250, https://gist.github.com/ryhanson/227229866af52e2d963cf941af135a52 |
| Author             | juju4      |


## Detection Rules

### Sigma rule

```
action: global
title: Suspicious Rundll32 Activity
description: Detects suspicious process related to rundll32 based on arguments
status: experimental
references:
    - http://www.hexacorn.com/blog/2017/05/01/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline/
    - https://twitter.com/Hexacorn/status/885258886428725250
    - https://gist.github.com/ryhanson/227229866af52e2d963cf941af135a52
tags:
    - attack.defense_evasion
    - attack.execution
    - attack.t1085
author: juju4
detection:
    selection:
        CommandLine:
            # match with or without rundll32.exe to try to catch evasion
            - '*\rundll32.exe* url.dll,*OpenURL *'
            - '*\rundll32.exe* url.dll,*OpenURLA *'
            - '*\rundll32.exe* url.dll,*FileProtocolHandler *'
            - '*\rundll32.exe* zipfldr.dll,*RouteTheCall *'
            - '*\rundll32.exe* Shell32.dll,*Control_RunDLL *'
            - '*\rundll32.exe javascript:*'
            - '* url.dll,*OpenURL *'
            - '* url.dll,*OpenURLA *'
            - '* url.dll,*FileProtocolHandler *'
            - '* zipfldr.dll,*RouteTheCall *'
            - '* Shell32.dll,*Control_RunDLL *'
            - '* javascript:*'
            - '*.RegisterXLL*'
    condition: selection
falsepositives:
    - False positives depend on scripts and administrative tools used in the monitored environment
---
# Windows Audit Log
logsource:
    product: windows
    service: security
    description: 'Requirements: Audit Policy : Detailed Tracking > Audit Process creation, Group Policy : Administrative Templates\System\Audit Process Creation'
detection:
    selection:
        EventID: 4688
---
# Sysmon
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 1
level: medium

```





### Kibana query

```
(EventID:"4688" AND CommandLine:("*\\\\rundll32.exe* url.dll,*OpenURL *" "*\\\\rundll32.exe* url.dll,*OpenURLA *" "*\\\\rundll32.exe* url.dll,*FileProtocolHandler *" "*\\\\rundll32.exe* zipfldr.dll,*RouteTheCall *" "*\\\\rundll32.exe* Shell32.dll,*Control_RunDLL *" "*\\\\rundll32.exe javascript\\:*" "* url.dll,*OpenURL *" "* url.dll,*OpenURLA *" "* url.dll,*FileProtocolHandler *" "* zipfldr.dll,*RouteTheCall *" "* Shell32.dll,*Control_RunDLL *" "* javascript\\:*" "*.RegisterXLL*"))\n(EventID:"1" AND CommandLine:("*\\\\rundll32.exe* url.dll,*OpenURL *" "*\\\\rundll32.exe* url.dll,*OpenURLA *" "*\\\\rundll32.exe* url.dll,*FileProtocolHandler *" "*\\\\rundll32.exe* zipfldr.dll,*RouteTheCall *" "*\\\\rundll32.exe* Shell32.dll,*Control_RunDLL *" "*\\\\rundll32.exe javascript\\:*" "* url.dll,*OpenURL *" "* url.dll,*OpenURLA *" "* url.dll,*FileProtocolHandler *" "* zipfldr.dll,*RouteTheCall *" "* Shell32.dll,*Control_RunDLL *" "* javascript\\:*" "*.RegisterXLL*"))
```





### X-Pack Watcher

```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_xpack/watcher/watch/Suspicious-Rundll32-Activity <<EOF\n{\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "query_string": {\n              "query": "(EventID:\\"4688\\" AND CommandLine:(\\"*\\\\\\\\rundll32.exe* url.dll,*OpenURL *\\" \\"*\\\\\\\\rundll32.exe* url.dll,*OpenURLA *\\" \\"*\\\\\\\\rundll32.exe* url.dll,*FileProtocolHandler *\\" \\"*\\\\\\\\rundll32.exe* zipfldr.dll,*RouteTheCall *\\" \\"*\\\\\\\\rundll32.exe* Shell32.dll,*Control_RunDLL *\\" \\"*\\\\\\\\rundll32.exe javascript\\\\:*\\" \\"* url.dll,*OpenURL *\\" \\"* url.dll,*OpenURLA *\\" \\"* url.dll,*FileProtocolHandler *\\" \\"* zipfldr.dll,*RouteTheCall *\\" \\"* Shell32.dll,*Control_RunDLL *\\" \\"* javascript\\\\:*\\" \\"*.RegisterXLL*\\"))",\n              "analyze_wildcard": true\n            }\n          }\n        },\n        "indices": []\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "email": {\n        "to": null,\n        "subject": "Sigma Rule \'Suspicious Rundll32 Activity\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\ncurl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_xpack/watcher/watch/Suspicious-Rundll32-Activity-2 <<EOF\n{\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "query_string": {\n              "query": "(EventID:\\"1\\" AND CommandLine:(\\"*\\\\\\\\rundll32.exe* url.dll,*OpenURL *\\" \\"*\\\\\\\\rundll32.exe* url.dll,*OpenURLA *\\" \\"*\\\\\\\\rundll32.exe* url.dll,*FileProtocolHandler *\\" \\"*\\\\\\\\rundll32.exe* zipfldr.dll,*RouteTheCall *\\" \\"*\\\\\\\\rundll32.exe* Shell32.dll,*Control_RunDLL *\\" \\"*\\\\\\\\rundll32.exe javascript\\\\:*\\" \\"* url.dll,*OpenURL *\\" \\"* url.dll,*OpenURLA *\\" \\"* url.dll,*FileProtocolHandler *\\" \\"* zipfldr.dll,*RouteTheCall *\\" \\"* Shell32.dll,*Control_RunDLL *\\" \\"* javascript\\\\:*\\" \\"*.RegisterXLL*\\"))",\n              "analyze_wildcard": true\n            }\n          }\n        },\n        "indices": []\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "email": {\n        "to": null,\n        "subject": "Sigma Rule \'Suspicious Rundll32 Activity\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```





### Graylog

```
(EventID:"4688" AND CommandLine:("*\\\\rundll32.exe* url.dll,*OpenURL *" "*\\\\rundll32.exe* url.dll,*OpenURLA *" "*\\\\rundll32.exe* url.dll,*FileProtocolHandler *" "*\\\\rundll32.exe* zipfldr.dll,*RouteTheCall *" "*\\\\rundll32.exe* Shell32.dll,*Control_RunDLL *" "*\\\\rundll32.exe javascript\\:*" "* url.dll,*OpenURL *" "* url.dll,*OpenURLA *" "* url.dll,*FileProtocolHandler *" "* zipfldr.dll,*RouteTheCall *" "* Shell32.dll,*Control_RunDLL *" "* javascript\\:*" "*.RegisterXLL*"))\n(EventID:"1" AND CommandLine:("*\\\\rundll32.exe* url.dll,*OpenURL *" "*\\\\rundll32.exe* url.dll,*OpenURLA *" "*\\\\rundll32.exe* url.dll,*FileProtocolHandler *" "*\\\\rundll32.exe* zipfldr.dll,*RouteTheCall *" "*\\\\rundll32.exe* Shell32.dll,*Control_RunDLL *" "*\\\\rundll32.exe javascript\\:*" "* url.dll,*OpenURL *" "* url.dll,*OpenURLA *" "* url.dll,*FileProtocolHandler *" "* zipfldr.dll,*RouteTheCall *" "* Shell32.dll,*Control_RunDLL *" "* javascript\\:*" "*.RegisterXLL*"))
```

