# 6. Video Playback & Archiving

The HiCAMS system automatically archives video clips from CCTV cameras when safety events are detected by the AI. You can search, review, and download these video clips at any time through the **Video Storage** screen.

> **Access Rights Note:**
> - **Admin** and **Manager**: Can view video recordings from all cameras.
> - **User** (Standard User): Can only view video recordings from cameras assigned to them.

---

## 6.1 Video Search

The **Video Storage** screen allows you to search and filter stored video records. Below is a detailed step-by-step guide.

### Step 1: Select Video Type

At the top of the left filter panel, select one of the two types:

| Video Type | Description |
|------------|-------------|
| **Auto** (Autosave) | Videos automatically saved by the system when an event occurs. |
| **Manual** (Upload) | Videos manually uploaded to the system by users. |

### Step 2: Select Time Range

Use the **Date Picker** to define the search time range:
- **Start time**: Click on the calendar icon to select the start date and time.
- **End time**: Click on the calendar icon to select the end date and time.

> **Default:** The system displays data from **one month ago** up to the **current time**.

### Step 3: Select Camera (Channel)

Filter videos by specific camera:
- Enter the camera description into the search box, or
- Click the dropdown menu to select a camera from the list.

> **Default:** All channels (cameras) are selected.

### Step 4: Filter by Events (Optional)

You can narrow down search results using the following event filters:

**a) "Show only records include events" Checkbox**
- Check this box to display only video records that contain at least one event.
- Default: Unchecked (displays all videos).
- _Note: This function is disabled when the "Manual" video type is selected._

**b) Filter by Event Type**

Events are divided into 3 main classes:

| Event Class | Examples |
|-------------|----------|
| **Human** | No helmet detection, Fall down, etc. |
| **Disaster/Accident** | Smoke, Fire, etc. |
| **Traffic** | Traffic-related events |

- Click the **class checkbox** to select/unselect all event types within that class.
- Click the **expansion arrow** to view the detailed list of specific event types.
- Click a **specific event icon** to select/unselect individual types.
- Hover over an event icon to view its description (tooltip).

> _Note: The Event Type filter is disabled when the "Manual" video type is selected._

### Step 5: Click "Apply"

Once all filters are set, click the **"Apply"** button to execute the search. The video list will update accordingly.

### Understanding Search Results

Search results are displayed in a grid of **Video capture cards**. Each card includes:

- **Thumbnail:** For videos with events, the thumbnail shows the frame from the time the event first occurred.
- **Event Icon:** Displayed at the top left of the thumbnail, indicating the type of event that occurred.
- **Video Duration:** Displayed at the bottom right of the thumbnail.
- **Video Information:** Displayed below the thumbnail, including Channel name, channel description, and the recorded date/time.

**Sorting Results:**
- Click the **"Sort by"** dropdown menu at the top right of the list.
- Select **"Latest"** (Newest first) or **"Oldest"** (Oldest first).

**Total Videos:**
- The count of found videos is displayed as **"Total: [n] videos"** at the top right.

**Infinite Scroll:**
- The system initially loads 20 videos. Scroll to the bottom of the list to automatically load the next 20 videos (20 → 40 → 60, etc.).

---

## 6.2 Video Play and Download

### How to Play Back Video

**1. Click on a video card**

From the search results in the Video Storage screen, click on any video capture card. The **Video Details** modal window will open, and the video will **automatically start playing**.

**2. Use Playback Controls**

The Video Details window provides a comprehensive set of controls:

| Control | Function |
|---------|----------|
| **Play / Pause** | Toggle between playing and pausing the video. |
| **Rewind / Forward 10s** | Skip backward or forward by exactly 10 seconds. |
| **Time bar** (Timeline) | Shows playback progress. Click or drag the slider to seek to a specific time. |
| **Volume** | Toggle volume on/off. Default: **Off**. |
| **Play speed** | Select speed: 0.25x, 0.5x, 0.75x, Default (1x), 1.25x, 1.5x, 1.75x, 2x. |
| **Full screen** | Expand the video player to full-screen mode. |

**Time bar for event videos:**
- **Red marker:** Indicates the exact time position where the event occurred on the timeline.
- **White segment:** Indicates the current playback position.
- Hovering over the time bar shows a precise time label.

**3. View Event List**

For autosaved videos containing events, an **"Event list"** panel is displayed on the right:
- **Event total:** Total number of events in the video clip.
- **Event Details:** Event name, event image, and occurrence time.
- **False positive check:** Checkbox to mark an event as incorrectly detected (see Section 4.5 Feedback Loop).

**4. Navigate Between Videos**

Use the **Previous** and **Next** buttons to move directly to the previous or next video in the search results.
- The Previous button is disabled on the first video.
- The Next button is disabled on the last video.

---

### How to Download Video

**1. Open the video**

Click a video card in the Video Storage list to open the Video Details window.

**2. Click the "Download" button**

Click the **Download icon** located on the Video Details screen.

**3. Save the file**

The video will be downloaded to your computer in **MP4** format.

> **Note:** Only the MP4 format is supported for downloads.

---

### Manual Video Upload

If you need to upload a video manually to the system:

**1. Switch to "Manual" mode**

In the Video Storage screen, select the **"Manual"** video type.

**2. Click the "Upload video" button**

Located at the top of the video grid. An **Upload video** pop-up will appear.

**3. Select video file**

- Drag and drop a video file into the upload area, or
- Click the **"Select file"** button to browse your computer.

> **Accepted Formats:** Supports **MP4** and **MOV (H264)** files only. If the format is incorrect, the system will show an _"Invalid file format"_ error.

**4. Video Preview**

After selecting a file, a preview will appear with standard playback controls for review.

**5. Select Channel**

Use the **"Select channel"** dropdown to assign the video to a specific camera.

**6. Enter Description (Optional)**

Enter a description for the video in the **Description** field.

**7. Confirm Upload**

- Click **"Confirm"** to complete the upload.
- Click **"Cancel"** or **"Close"** to abort.

---

### Storage Rules & Auto-Deletion

The HiCAMS system applies the following archiving rules to manage storage capacity:

| Rule | Details |
|------|---------|
| **Autosaving** | The system automatically saves video clips containing detected events. |
| **Event-less Deletion** | Videos without events are **automatically deleted after 5 minutes** of storage. |
| **Storage Limits** | Admins and Managers can set a "Total recording limit" in the settings. |
| **FIFO Policy** | When storage is full, the system automatically deletes the **oldest video records** first to make room for new ones. |

> **Important Note:** If a video linked to an event log has been deleted due to storage policies, opening that event will display a **"Video has been removed"** message and playback controls will be disabled. However, the event information and captured image (Capture tab) remain accessible.
