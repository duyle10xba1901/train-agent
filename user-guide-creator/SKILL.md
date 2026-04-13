---
name: user-guide-creator
description: Create, write, or draft short and concise user guidelines for software features following the HiCAMS company standard. Trigger whenever the user asks for a user guide, software manual, system documentation, or needs to document a specific screen/function. This skill formats explanations into the mandatory HiCAMS UI layouts, specifying screenshot placement and "orange dashed border" annotations.
---

# User Guide Creator

This skill helps you generate software user guides exactly according to the HiCAMS corporate standard.
HiCAMS user guides are designed to be extremely simple, concise, and focused on helping end-users understand the functionality rapidly.
They follow a strict visual and textual presentation logic organized by screens/functions.

## 1. Structure of a Guide (Slide/Page)

A standard HiCAMS guide page represents a single functional screen and MUST include:
1.  **Header**: The function or screen name, capitalized and bold (e.g., **VIEWER**, **EVENT LOG EXPLORER**, **SET UP**).
2.  **Short Description**: Exactly 1 to 2 bullet points describing the primary role or action for that screen. Keep it extremely brief and easy to understand.
3.  **Screenshot Placeholder**: A clear indication of where the system screenshot should go.
4.  **Annotations/Callouts Location**: A specific, descriptive list telling the user exactly where to draw "orange dashed rectangular borders" on the screenshot, and what text label (callout) to associate with them.

## 2. Tone and Style

-   **Language**: Vietnamese is the default. If the user explicitly asks for English, use English.
-   **Clarity**: Sentences must be extremely short, avoiding overly technical developer jargon. Explain the "what" and "how".
-   **Wording**: Use direct, imperative verbs (e.g., "Nhập", "Chọn", "Lọc", "Giám sát", "Quản lý" / "Input", "Select", "Filter", "Monitor", "Manage").

## 3. Output Formats

By default, output the generated user guide in **Markdown** format using the template below.

If the user explicitly requests to output a **PowerPoint (.pptx)** file, you MUST:
1. Draft the content using the logic below.
2. Call the `pptx` skill to generate the actual presentation file. The `pptx` skill is available in your standard toolkit.

### Markdown Output Template

Always use this exact template when outputting the markdown:

```markdown
# [TÊN CHỨC NĂNG HOẶC MÀN HÌNH IN HOA]

*   [1-2 câu ngắn ngọn mô tả chức năng của màn hình này. Dùng động từ mệnh lệnh. Ví dụ: "Nhập tài khoản/mật khẩu để truy cập vào hệ thống chính."]

**[Ảnh chụp màn hình (Screenshot) - Chèn file ảnh tĩnh của màn hình vào đây]**

### Hướng dẫn khoanh viền đứt nét màu cam (Orange Dashed Borders)
-   **Khu vực [Tên khu vực 1]**: Khoanh viền vùng [Mô tả chi tiết vị trí vùng trên ảnh, ví dụ: "Góc trên bên trái, phần filter ngày tháng"].
    -   **Nhãn ghi chú (Callout)**: "[Nội dung ghi chú ngắn gọn. Ví dụ: Lọc để hiển thị các bản ghi sự kiện mong muốn]"
-   **Khu vực [Tên khu vực 2]**: Khoanh viền vùng [Mô tả chi tiết vị trí vùng trên ảnh].
    -   **Nhãn ghi chú (Callout)**: "[Nội dung ghi chú ngắn gọn]"
```

## 4. Instructions for Assisting the User

When a user provides you with an image of a UI, a function description, or technical specs to write a guide for:
1. **Examine the inputs carefully**.
2. **Determine the core function name** for the header.
3. **Write the 1-2 bullet points** describing the screen's main purpose.
4. **Identify 2-5 key areas** on the UI that need explaining.
5. **Create the descriptive list** for the orange dashed borders based on those areas.
6. **Output the result** using the exact Markdown template above.
7. Only generate a `.pptx` file if explicitly requested by the user.
