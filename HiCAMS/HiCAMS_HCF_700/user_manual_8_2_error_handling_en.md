# 8.2 Error Handling: Primary Response to Camera Failure

## Description

This guide outlines the first steps you should take if a camera feed is repeatedly lost or displaying an error. Following these steps helps identify whether the issue is a temporary glitch, a network problem, or a hardware failure, enabling you to bring the surveillance system back online quickly or report it accurately.

---

## Primary Response Steps

If a camera issue is detected (e.g., the video feed turns black and shows "Disconnected"), please follow these steps:

### 1. Check Camera Status
Verify the specific error message on the camera feed to understand the root cause.
- **Identify the disconnected camera**: Locate the dark frame displaying the **"Disconnected"** text and the camera name.
- **Check the error tooltip**: Hover your mouse over the **information icon ("i")** in the corner of the disconnected frame. Read the tooltip to identify the exact error:
  - `IP error`: The camera is unreachable on the network.
  - `Account info error`: The ID or password for the camera is incorrect.
  - `Profile path error`: The camera stream profile setting is incorrect.
- **Check surrounding cameras**: Look at the other camera feeds in the grid. If only one camera is down, it is an isolated issue. If multiple or all cameras are down simultaneously, it is likely a network issue.

### 2. Verify Network Connection
Ensure the system and the camera are communicating properly.
- **Check system responsiveness**: Verify that you can still navigate between different screens (e.g., Viewer and Event Log Explorer). If the entire HiCAMS interface is unresponsive, check your workstation's network connection.
- **Test connection (For Admins/Managers)**: Go to **Set up > Channel management**. Select the problematic camera and click the **"Check"** button to force the system to test the connection source, IP, and protocol.

### 3. Refresh the Screen
Sometimes a temporary drop causes the feed to freeze or disconnect.
- **Reload the monitoring screen**: Click the **Viewer** tab again or refresh your web browser to force the system to attempt a reconnection.
- **Observe reconnection status**: After refreshing, watch the camera frame.
  - Look for the **"Loading"** spinner.
  - Look for the **"Synchronizing"** text.
- **Confirm recovery**: Wait to see if the live video resumes normally and the time synchronizes with real-time.

### 4. Restart the Camera (if applicable)
If the camera remains disconnected and it is an isolated `IP error`, the hardware might need a reboot.
- **Restart the device**: If you have physical access or network access to the camera's power supply / PoE switch, restart the camera device.
- **Wait for reconnection**: Wait 2-3 minutes for the camera to boot up.
- **Monitor the system**: Check the HiCAMS Viewer screen again to see if the system successfully passes the "Loading" and "Synchronizing" phases.

---

## Reporting Persistent Issues

If you have completed all the steps above and the camera remains **"Disconnected"**, escalate the issue to the IT department or System Administrator immediately.

Provide them with the following information you gathered from Step 1:
- The **Camera Name**
- The exact **Error Type** (IP error, Account info error, or Profile path error)
- The **Time** the camera went offline (shown in the tooltip)
- Whether other cameras are affected
