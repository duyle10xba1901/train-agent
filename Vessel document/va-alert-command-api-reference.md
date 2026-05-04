# va_alert_command - Alert Management API Reference

> **Source**: HiNAS_Supervisory_System_ICD_v1.2.1 (2026-03-23), Sections 3.2.6, 3.3.6, 4.7, 5.4
> **Direction**: Supervisory system -> HiNAS (ACN) / HiNAS -> Supervisory system (ALF, ALC, ARC)

---

## 1. Overview

Interface for the Supervisory system to send **alert state change commands** to HiNAS.

| Sentence | Key (ICD) | Proprietary Name | Direction | Role |
|----------|-----------|-----------------|-----------|------|
| **ACN** | `va_alert_command` | `PADRACN` | Supervisory -> HiNAS | Alert command (Acknowledge, Query, Transfer, Silence) |
| **ALF** | `PAVKALF` | `PAVKALF` | HiNAS -> Supervisory | Alert notification (2 sentences per alert: title + description) |
| **ALC** | `PAVKALC` | `PAVKALC` | HiNAS -> Supervisory | Active alert list (periodic every 30s + on state change) |
| **ARC** | `PAVKARC` | `PAVKARC` | HiNAS -> Supervisory | ACN command response |

> **Note**: The complete list of supported alert codes (ALF codes) and their descriptions is **TBD** (pending confirmation from Avikus).
> Avikus proprietary alerts: ALF code >= 10000, mnemonic = `AVK`. Standard IEC alerts: mnemonic = empty.

---

## 2. Protocol / Transport Information

| Item | Value |
|------|-------|
| **Standard** | IEC 61162-450 Ed.3 (IEC 61162-1 over UDP/IP) |
| **Related Standards** | IEC 62923-1 (Bridge Alert Management), IEC 62923-2 (Claim of Compliance) |
| **Network** | IPv4 UDP Multicast |
| **Multicast Address** | 239.129.0.1 ~ 239.129.0.16 *(TBD)* |
| **UDP Port** | 60001 ~ 60016 *(TBD)* |
| **TTL** | 1 (link-local only) |
| **Payload** | IEC 61162-1 sentences, ASCII |
| **Transmission** | ALF: Event-driven / ALC: Periodic(30s) + Event / ACN: Event-driven / ARC: Event-driven(response) |

---

## 3. Signal List

### 3.1. Supervisory -> HiNAS (Section 3.2.6)

| Title | Key | Description | Protocol | Format | Pattern / Period | Notes |
|-------|-----|-------------|----------|--------|-----------------|-------|
| **Alert Command** | `va_alert_command` | Alert commands from Supervisory to HiNAS: Acknowledge, Query, Responsibility Transfer, Silence | IEC 61162-450 Ed.3 UdPbC (UDP multicast, port *TBD*) | IEC 61162-1 ACN (`PADRACN`) | Event, On operator action | |

### 3.2. HiNAS -> Supervisory (Section 3.3.6)

| Title | Key | Description | Protocol | Format | Pattern / Period | Notes |
|-------|-----|-------------|----------|--------|-----------------|-------|
| **Alert List Frame (ALF)** | `PAVKALF` | Alert detail frame -- 2 sentences per active alert (title + description). Sent on new alert onset or state change. | IEC 61162-450 Ed.3 UdPbC (UDP multicast, port *TBD*) | IEC 61162-1 ALF | Event, On new alert | Supported alert code list **TBD** |
| **Alert List Cyclic (ALC)** | `PAVKALC` | Cyclic alert status summary. Provides ongoing list of active alerts. | IEC 61162-450 Ed.3 UdPbC (UDP multicast, port *TBD*) | IEC 61162-1 ALC | Periodic, 30s | |
| **Alert Command Response (ARC)** | `PAVKARC` | Response to ACN commands (Acknowledge, Query, Transfer, Silence) received from Supervisory system. | IEC 61162-450 Ed.3 UdPbC (UDP multicast, port *TBD*) | IEC 61162-1 ARC | Event, On alert command | |

### 3.3. Signal List (Section 4.7.3)

| Title | Key | Direction | Description | Trigger | Source |
|-------|-----|-----------|-------------|---------|--------|
| **ALF** | `$--ALF` | HiNAS -> Supervisory | Alert notification. 2 sentences per alert (sentence 1 = title, sentence 2 = description). | [Event] Alert state change | HiNAS |
| **ALC** | `$--ALC` | HiNAS -> Supervisory | Active alert list (periodic + immediately on state change). | [Periodic] 30s + [Event] Alert state change | HiNAS |
| **ACN** | `$--ACN` | Supervisory -> HiNAS | Alert command (Acknowledge, Silence, Query, Transfer). | [Event] Operator action | Supervisory |
| **ARC** | `$--ARC` | HiNAS -> Supervisory | ACN processing result response (sent on both acceptance and rejection). | [Event] ACN received | HiNAS |

---

## 4. ACN (Alert Command) - Supervisory -> HiNAS

### 4.1. Sentence Format

```
$--ACN,hhmmss,aaa,c--c,x,a,a*hh<CR><LF>
$--ACN,{utc_time},{mnemonic},{alf},{instance_number},{command},{status}*hh<CR><LF>
```

### 4.2. Example

```
\s:AP0001,d:TC0001*3D\$TCACN,123456,AVK,777035,1,A,C*32<CR><LF>
```

### 4.3. Field Definitions

| Field | Symbol | Range / Values | Description |
|-------|--------|---------------|-------------|
| talker | `$--` | Talker ID | Talker identifier. Actual prefix depends on source device. |
| sentence | ACN | `ACN` | Sentence type identifier. |
| utc_time | hhmmss | `hhmmss` | UTC time of the command. |
| mnemonic | aaa | `AVK` or empty | Manufacturer mnemonic. Must match the target alert's mnemonic. |
| alf | c--c | integer `0-9999999` | Target alert identifier. |
| instance_number | x | integer `0-999999` | Target instance. `0` = all instances of the specified ALF code. |
| command | a | `A` \| `Q` \| `O` \| `S` | **A** = Acknowledge, **Q** = Query (re-send ALF), **O** = Responsibility Transfer, **S** = Silence. |
| status | a | `C` | `C` = Command. Only this value is accepted. |
| checksum | hh | 00-FF | XOR of all characters between `$` and `*`. |

### 4.4. Command Details

| Command | Name | Processing |
|---------|------|-----------|
| **A** | Acknowledge | Acknowledge the specified alert. `ALF=0` is **rejected** (Acknowledge All not permitted). **Caution (C) alerts are rejected** (auto-acknowledged). `instance_number=0`: Acknowledge all instances of the specified ALF code. |
| **Q** | Query | Request re-send of the ALF pair for the specified alert. `ALF=0`: Re-send ALF for all currently active alerts. `instance_number=0`: Re-send ALF for all instances of the specified ALF code. |
| **O** | Responsibility Transfer | Transfer responsibility for the alert to another system. Alert state changes to `O` (Transferred). **Emergency Alarm and Caution alerts are rejected.** |
| **S** | Silence | Silence the alert's audible/visual notification. **Only permitted for Alarm (A) and Warning (W) priority alerts.** Caution (C) cannot be silenced. `ALF=0`: Silence all currently active Alarm/Warning alerts. |

### 4.5. Behavior Rules

| Rule | Description |
|------|-------------|
| **Response on rejection** | If an ACN command is rejected (unknown alert, prohibited operation, etc.), HiNAS sends an ARC response echoing the command. The alert state is not changed. |
| **Acknowledge All rejected** | ACN with `alf=0` and command `A` (Acknowledge All) is always rejected. HiNAS responds with ARC. |

### 4.6. Multi-Sentence Splitting (82-Character Limit)

- Each `$--ACN` sentence addresses **exactly one alert command**
- Multiple commands (e.g., acknowledging several alerts) must be sent as **sequential separate sentences**
- Do not batch multiple commands into a single sentence
- No `total_sentences/sentence_number` grouping -- each sentence is self-contained -> ARC response

---

## 5. ARC (Alert Response Command) - HiNAS -> Supervisory

Sent by HiNAS in response to an ACN command. ARC is sent both on successful processing and on rejection.

### 5.1. Sentence Format

```
$--ARC,hhmmss,c--c,x,x,a*hh<CR><LF>
$--ARC,{utc_time},{mnemonic},{alf},{instance_number},{command}*hh<CR><LF>
```

### 5.2. Example

```
\s:TC0001,d:AP0001*3D\$TCARC,123456,AVK,777035,1,A*41<CR><LF>
```

### 5.3. Field Definitions

| Field | Symbol | Range / Values | Description |
|-------|--------|---------------|-------------|
| talker | `$--` | Talker ID | Talker identifier. |
| sentence | ARC | `ARC` | Sentence type identifier. |
| utc_time | hhmmss | `hhmmss` | UTC time of the response (echoed from ACN). |
| mnemonic | c--c | `AVK` or empty | Echoed from ACN. |
| alf | x | integer `0-9999999` | Echoed from ACN. |
| instance_number | x | integer `0-999999` | Echoed from ACN. |
| command | a | `A` \| `Q` \| `O` \| `S` | Echoed from ACN, confirming the command was processed. |
| checksum | hh | 00-FF | XOR of all characters between `$` and `*`. |

---

## 6. Alert Priority & State Model (IEC 62923-1)

### 6.1. Priority Levels

| Code | Priority | Description |
|------|----------|-------------|
| **A** | Alarm | Highest priority. Requires acknowledgement. Can be silenced. |
| **W** | Warning | Medium priority. Requires acknowledgement. Can be silenced. |
| **C** | Caution | Lowest priority. Does not require acknowledgement (auto-acknowledged). Cannot be silenced. |

### 6.2. Alert States

| Code | State | Description |
|------|-------|-------------|
| **V** | Active (unacknowledged) | Alert has been raised but not yet acknowledged. |
| **A** | Acknowledged | Operator has acknowledged the alert via ACN command `A`. |
| **S** | Silenced | Audible/visual notification silenced via ACN command `S`. Only for Alarm and Warning priorities. |
| **O** | Transferred | Responsibility for the alert has been transferred via ACN command `O`. |

---

## 7. ALF (Alert) - HiNAS -> Supervisory (Reference)

Each alert is sent as **two consecutive sentences** sharing the same Sequential Message ID. Sentence 1 contains the alert metadata and title; sentence 2 contains the description.

### 7.1. 1st Sentence (Title)

```
$--ALF,{total_sentences},{sentence_number},{message_id},{utc_time},{category},{priority},{alert_state},{mnemonic},{alf},{instance_number},{revision_counter},{escalation_counter},{title}*hh<CR><LF>
```

### 7.2. 2nd Sentence (Description)

```
$--ALF,{total_sentences},{sentence_number},{message_id},,,,{mnemonic},{alf},{instance_number},{revision_counter},{escalation_counter},{description}*hh<CR><LF>
```

> Fields `utc_time`, `category`, `priority`, `alert_state` are empty in the 2nd sentence.

### 7.3. Field Definitions

| Field | Symbol | Range / Values | Description |
|-------|--------|---------------|-------------|
| talker | `$--` | Talker ID | Talker identifier. Actual prefix depends on source device. |
| sentence | ALF | `ALF` | Sentence type identifier. |
| total_sentences | x | `2` (fixed) | Always 2 -- each alert requires a title sentence and a description sentence. |
| sentence_number | x | `1` \| `2` | `1` = this sentence (title), `2` = next sentence (description). |
| message_id | x | `0-9` | Cycles 0 -> 9 -> 0. Shared between the two sentences of the same alert event. |
| utc_time | hhmmss | `hhmmss` | UTC time of the alert state change. **Empty in 2nd sentence.** |
| category | a | `A` \| `B` \| `C` | Alert category per IEC 62923-1. **Empty in 2nd sentence.** |
| priority | a | `A` \| `W` \| `C` | `A` = Alarm, `W` = Warning, `C` = Caution. **Empty in 2nd sentence.** |
| alert_state | a | `V` \| `A` \| `S` \| `O` | `V` = Active, `A` = Acknowledged, `S` = Silenced, `O` = Transferred. **Empty in 2nd sentence.** |
| mnemonic | c--c | `AVK` or empty | Manufacturer mnemonic. `AVK` for proprietary alerts (ALF code >= 10000); empty for standard alerts. |
| alf | x | integer `0-9999999` | Alert identifier. |
| instance_number | x | integer `0-999999` or empty | Instance number for multi-instance alerts. Empty if not applicable. |
| revision_counter | x | `1-99` | Incremented on each ALF re-send for the same alert. |
| escalation_counter | x | `0-9` | Escalation level. `0` = no escalation. |
| title | c--c | string (max 16 chars) | Alert title text. **(1st sentence only)** |
| description | c--c | string | Alert description text (full). **(2nd sentence only)** |
| checksum | hh | 00-FF | XOR of all characters between `$` and `*`. |

### 7.4. Example

```
-- 1st sentence (Title):
\s:TC0001,d:AP0001*3D\$TCALF,2,1,0,123456,A,W,V,AVK,777035,1,0,0,Collision Danger*hh<CR><LF>

-- 2nd sentence (Description):
\s:TC0001,d:AP0001*3D\$TCALF,2,2,0,,,,AVK,777035,1,0,0,HiNAS detected collision danger*12<CR><LF>
```

---

## 8. ALC (Alert List Cyclic) - HiNAS -> Supervisory (Reference)

Lists all currently active alerts as entities. Sent periodically every 30 seconds and also immediately on any alert state change. If no alerts are active, a single sentence with `entity_count=0` is sent.

### 8.1. Sentence Format

```
$--ALC,{total_sentences},{sentence_number},{message_id},{entity_count},{entity1},{entity2},...*hh<CR><LF>
```

Each entity format: `{mnemonic},{alf},{instance_number},{revision_counter}`

### 8.2. Field Definitions

| Field | Symbol | Range / Values | Description |
|-------|--------|---------------|-------------|
| talker | `$--` | Talker ID | Talker identifier. |
| sentence | ALC | `ALC` | Sentence type identifier. |
| total_sentences | x | `01-99` | Total sentences for this message. 2-digit, zero-padded. |
| sentence_number | x | `01-99` | Current sentence number. 2-digit, zero-padded. |
| message_id | x | `00-99` | Sequential message identifier, `00` to `99`. |
| entity_count | x | `0-9` | Number of alert entities in **this** sentence. `0` = no active alerts. |
| entity | x | `mnemonic,alf,instance_number,revision_counter` | Comma-separated group. Repeated `entity_count` times. |
| checksum | hh | 00-FF | XOR of all characters between `$` and `*`. |

### 8.3. Examples

```
-- One active alert:
\s:TC0001,d:AP0001*3D\$TCALC,01,01,00,1,AVK,777035,1,0*34<CR><LF>

-- Two active alerts:
\s:TC0001,d:AP0001*3D\$TCALC,01,01,00,2,AVK,777035,1,0,AVK,777036,1,0*68<CR><LF>

-- No active alerts:
\s:TC0001,d:AP0001*3D\$TCALC,01,01,00,0*69<CR><LF>
```

### 8.4. 82-Character Splitting

- Each ALC sentence is limited to **82 characters** (IEC 61162-1)
- When the entity list exceeds this limit, entities are split across multiple ALC sentences
- `total_sentences`: Total number of ALC sentences in this cycle
- `entity_count`: Reflects only the entities in **that** sentence
- All sentences share the same `message_id`
- Supervisory system must collect all sentences with the same `message_id` to reconstruct the full alert list

---

## 9. Scenario Summary

| # | Scenario | Flow |
|---|----------|------|
| 1 | **Normal Alert Lifecycle** | HiNAS: ALC(empty) -> [Alert activated] -> ALF(title+desc) -> ALC(updated) |
| 2 | **Acknowledge** | Supervisory: ACN(A) -> HiNAS: ARC(A) -> ALF(state=A) -> ALC(updated) |
| 3 | **Silence** | Supervisory: ACN(S) -> HiNAS: ARC(S) -> ALF(state=S) -> ALC(updated). Alarm/Warning only. |
| 4 | **Query** | Supervisory: ACN(Q) -> HiNAS: ARC(Q) -> ALF(re-sent, revision++). No ALC re-send. |
| 5 | **Responsibility Transfer** | Supervisory: ACN(O) -> HiNAS: ARC(O) -> ALF(state=O) -> ALC(updated) |
| 6 | **Rejected: Ack All** | Supervisory: ACN(alf=0,A) -> HiNAS: ARC(rejection). Individual Ack only. |
| 7 | **Rejected: Silence on Caution** | Supervisory: ACN(S, Caution alert) -> HiNAS: ARC(rejection). Alarm/Warning only. |

---

## 10. TAG Block Reference

```
\s:<Source SFI>,d:<Dest SFI>,i:<Command ID>,c:<Timestamp>*<Checksum>\
```

| Field | Description |
|-------|-------------|
| `s` | Source SFI (System Function Identifier) |
| `d` | Destination SFI |
| `i` | Command ID - UUID (RFC 4122), assigned when sending ACN, echoed in ARC response |
| `c` | POSIX timestamp (seconds since 1970-01-01) |
| Checksum | XOR of all characters between `\` and `*`, encoded as 2-digit hex |

- HiNAS -> Supervisory: talker `TC` (HiNAS SFI)
- Supervisory -> HiNAS: talker prefix is TBD
- `$PAVK---`: HiNAS (Avikus) proprietary sentence prefix
- `$PADR---`: Supervisory proprietary sentence prefix
