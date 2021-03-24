# \[WIP] Git Based Auth Manager (GBAM)

A small and clear auth management method for developer team.

## prototype

```puml
@startuml
'https://plantuml.com/use-case-diagram

:Stuff:
:Leader:
:Audit Bot: as AB
:Operator:
:Auth Manage Server: as AMS

(Auth config repo) as ACR
(Auth Service <AWS>) as AS

Stuff --> ACR: edit
Leader --> ACR: audit
AB --> ACR: audit
AB --> ACR: validate
Operator --> ACR: accept merge

AMS <-- ACR: load config
AMS -up-> AS: update role config

Stuff <-up- AS: authenticate
@enduml
```

```puml
@startuml
'https://plantuml.com/sequence-diagram
autonumber

actor "Auth Service" as AS
actor Stuff
participant GIT
actor "Audit Bot" as AB
actor Leader
actor Operator

Stuff -> GIT: submit auth change request PR

AB --> GIT: validate request
AB -> GIT: check stuff, \nmake leader reviewer

AB -> Leader: auth request\n notification

Leader -> GIT: send '/approved'

AB -> Operator: auth request\n notification
Operator -> GIT: send '/continue'

AB <- GIT: read changes,\n generate request
AB -> AS: send change auth request
AS -> Stuff: grant auth
AS -> AB: operation success

AB -> GIT: continue merge PR

AB -> Stuff: authenticated notification
@enduml
```

### The role of git in GBAM

The git repo is used to save auth configs, accept PR (auth request). All requests are
transparent and auditable. All operations of a single auth request are within single
PR.


### Audit bot

Audit bot is the core character of GBAM. It:
- validates auth request config
- check stuff relationships and send review request to stuff's leader
- collect leader's `approved` message
- send final check request to operator
- collect operator's `continue` message
- submit changes to auth service (like AWS)
    - if failed, bot will send message to the PR and waiting for next commit 
- merge request PR

