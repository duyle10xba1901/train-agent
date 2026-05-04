# 8.1 Daily Operational Checklist

## Description

The Daily Operational Checklist helps crew members ensure that the HiCAMS system is functioning properly before and during each shift. By performing these quick checks, you can identify and address issues early — keeping your CCTV surveillance and AI safety detection running smoothly at all times.

> **When to perform:** Complete this checklist at the **beginning of each shift** and once during the shift if possible.

---

## Checklist

### 1. Camera Status

Open the **Viewer** screen (default screen after login). Scan all camera feeds in the grid and verify the following:

| # | Check | What to look for | Status |
|---|-------|-------------------|--------|
| 1.1 | Verify all cameras are **online** | Each camera frame should display a live video feed. | ☐ Pass |
| 1.2 | Check for **"Disconnected"** cameras | Look for any camera frame showing a dark screen with the text **"Disconnected"**. | ☐ Pass |
| 1.3 | Identify the **disconnect reason** (if any) | Hover over the **info icon ("i")** on any disconnected camera frame. Read the tooltip — it will show the error type and timestamp. | ☐ Pass |
| 1.4 | Verify live video is **not frozen** | Confirm each camera feed is actively streaming (look for movement or changing timestamps). | ☐ Pass |
| 1.5 | Check for **"Loading"** or **"Synchronizing"** cameras | If any camera frame shows a spinning icon with "Loading" or "Synchronizing" text, wait a moment. If it persists for more than 1 minute, report the issue. | ☐ Pass |

**Possible disconnect errors and meaning:**

| Error Type | Meaning | Action |
|------------|---------|--------|
| IP error | Camera is not reachable on the network | Report to IT / Admin |
| Account info error | Camera credentials (ID/Password) are incorrect | Report to Admin |
| Profile path error | Camera stream profile is not configured correctly | Report to Admin |

> **Result:** All cameras should show live video. If any camera is disconnected, note the camera name, error type, and timestamp, then report to the Admin or Manager.

---

### 2. AI Surveillance Status

On the **Viewer** screen, check the left-hand control panel:

| # | Check | What to look for | Status |
|---|-------|-------------------|--------|
| 2.1 | Verify the **"AI Surveillance"** switch is **ON** | The toggle button on the left panel should be in the ON position. | ☐ Pass |
| 2.2 | Verify **General Severity (GS)** scores are **visible** | Each camera frame should display a severity level (Low / Medium / High). If the scores are hidden, AI Surveillance may be turned off. | ☐ Pass |
| 2.3 | Verify **detection frames** are working | Observe camera feeds for a few minutes. When a person or object appears, you should see colored bounding boxes (detection frames) appear around detected objects. | ☐ Pass |

> **Result:** The AI Surveillance switch must be ON. If it is off, click the switch to turn it on. If detection frames do not appear even when objects are present, report to the Admin.

---

### 3. Event Detection

Navigate to the **Event Log Explorer** screen by clicking **"Event Log Explorer"** in the top navigation bar.

| # | Check | What to look for | Status |
|---|-------|-------------------|--------|
| 3.1 | Verify **recent events are being logged** | Check the Event Log list. Confirm that recent events appear with today's date (or within the last few hours). | ☐ Pass |
| 3.2 | Review the **latest 5–10 events** | Click on each recent event in the list. In the detail panel, verify that the **Capture** tab shows a clear image with visible bounding boxes. | ☐ Pass |
| 3.3 | Check for **unusual patterns** | Look for any unexpected spike in events, or an unusual absence of events (no events for several hours when cameras are active). | ☐ Pass |
| 3.4 | Verify **event types are correct** | Confirm that the event type icons and descriptions match what is shown in the captured images. | ☐ Pass |

> **Result:** Events should be appearing regularly during operating hours. If no events have been logged for an extended period despite camera activity, report to the Admin.

---

### 4. Alert & Alarm System

Return to the **Viewer** screen and verify the alarm system is operational:

| # | Check | What to look for | Status |
|---|-------|-------------------|--------|
| 4.1 | Check for **active alarm bells** | Look for any red **Alarm bell** icon on camera frames. This indicates an open, unresolved event. | ☐ Pass |
| 4.2 | Verify **warning sound is audible** | Ensure your computer or workstation volume is turned on. When an alarm event occurs, a warning sound should play. | ☐ Pass |
| 4.3 | Acknowledge any **open alarms** | If there are active alarm bells from the previous shift, hover over the alarm bell until it shows "Turn off". Click it to acknowledge and resolve the event. | ☐ Pass |
| 4.4 | Verify **Event Popup** is working | If configured, a Channel Information popup should appear automatically when a high-risk event is detected. Confirm this feature is active. | ☐ Pass |

> **Result:** Alarms must remain active until manually acknowledged. Clear any leftover alarms from the previous shift. If no alarm sound is heard during a known event, check the workstation audio settings and report to the Admin.

---

### 5. Video Storage

Navigate to the **Video Storage** screen by clicking **"Video Storage"** in the top navigation bar.

| # | Check | What to look for | Status |
|---|-------|-------------------|--------|
| 5.1 | Verify **videos are being recorded** | Select "Auto" video type and click "Apply". Confirm that recent video recordings appear in the grid with today's date. | ☐ Pass |
| 5.2 | Verify **event videos have event icons** | Check that video thumbnails for event recordings display the correct event icon in the top-left corner. | ☐ Pass |
| 5.3 | Spot-check a **video playback** | Click on a recent video card. Confirm the video plays correctly with image and sound (if applicable). | ☐ Pass |
| 5.4 | Verify **download is working** | From the video detail screen, click the **Download** button. Confirm the MP4 file downloads successfully. | ☐ Pass |

> **Result:** Recent recordings should appear. If no videos are available or if playback fails, report to the Admin.

---

### 6. Network / Connection

Perform a general connectivity check:

| # | Check | What to look for | Status |
|---|-------|-------------------|--------|
| 6.1 | Confirm the **system is responsive** | Navigate between screens (Viewer → Event Log Explorer → Video Storage). Verify pages load without excessive delay. | ☐ Pass |
| 6.2 | Check for **lag or delay** in live feeds | On the Viewer screen, observe the camera timestamps. They should be close to the current real time (within a few seconds). | ☐ Pass |
| 6.3 | Verify **no mass disconnections** | If multiple cameras show "Disconnected" simultaneously, this may indicate a network-wide issue rather than individual camera failures. | ☐ Pass |

> **Result:** The system should be responsive with minimal delay. If multiple cameras are disconnected at once or if the system is unresponsive, report a potential network issue to IT immediately.

---

## Summary

| Check Area | Screen to Use | Key Action |
|------------|---------------|------------|
| Camera Status | Viewer | Scan grid for "Disconnected" cameras |
| AI Surveillance | Viewer (left panel) | Confirm the AI Surveillance switch is ON |
| Event Detection | Event Log Explorer | Review recent event logs |
| Alert & Alarm | Viewer | Acknowledge any open alarm bells |
| Video Storage | Video Storage | Verify recent recordings exist |
| Network / Connection | All screens | Navigate between screens, check responsiveness |

---

## Reporting Issues

If any check fails, record the following information and report to the **Admin** or **Manager**:

1. **Date and Time** of the issue
2. **Screen name** where the issue was found (e.g., Viewer, Event Log Explorer)
3. **Camera name** (if applicable)
4. **Error message or description** (e.g., "Disconnected — IP error — 2026-04-07 08:15")
5. **Your account name**

> **Tip:** Take a screenshot of the issue whenever possible to help the Admin diagnose the problem faster.
